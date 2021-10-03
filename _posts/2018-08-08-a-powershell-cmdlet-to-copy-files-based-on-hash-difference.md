---
layout: post
title: Copy files with hash difference via PowerShell
image: "/content/images/2018/08/apple-1868383_1920.jpg"
date: '2018-08-08 13:23:43'
tags:
- powershell
---

This blog post details a PowerShell Core compatible cmdlet that I have authored named `Copy-FileHash` that you can use to copy modified files from one path tree to another. The cmdlet determines which files have different contents by calculating their hash values through the `Get-FileHash` cmdlet. This might be useful if you need to copy just files that have been modified between two paths and aren't able to rely on the modified date of those files to determine which have changed. 

> **What is a hash value?**
>
> *"A hash value is a unique value that corresponds to the content of the file. Rather than identifying the contents of a file by its file name, extension, or other designation, a hash assigns a unique value to the contents of a file. File names and extensions can be changed without altering the content of the file, and without changing the hash value. Similarly, the file's content can be changed without changing the name or extension. However, changing even a single character in the contents of a file changes the hash value of the file."*
>
> -- Source: [Get-FileHash Official Documentation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/get-filehash?view=powershell-6)

If you want to go directly to the code, you can find it [in GitHub here](https://github.com/markwragg/PowerShell-HashCopy), or you can [install it from the PowerShell Gallery](https://www.powershellgallery.com/packages/HashCopy/) by executing:
```
Install-Module HashCopy -Scope CurrentUser
```
You can then use:
```
Get-Help Copy-FileHash -Full
```
For full details on how the cmdlet works.

> `Copy-FileHash` requires Windows PowerShell version 4 or above, or PowerShell Core version 6 or above (through which the cmdlet should work on MacOS, Ubuntu or Windows). 

Here is an example of `Copy-FileHash` in action:

![Copy-FileHash](/content/images/2018/08/Copy-FileHash-1.png)

In the above example `1.txt` was in the source path but not in the destination path so was created and then copied, `2.txt` was found to be different in the source path and so was overwritten. The file objects for the resultant modified files were returned because `-PassThru` was used.

#### Why might this be useful?

I recently had a situation where I needed to be able to deploy only changed files from a set of files. Those files had been put in to a .zip package, created from a git-based Source Control and ultimately deployed to a server via Octopus Deploy. It's a [built-in and necessary behaviour of Git to change the modified date of files](https://git.wiki.kernel.org/index.php/GitFaq#Why_isn.27t_Git_preserving_modification_time_on_files.3F) as it manages them. As a result, I couldn't rely on comparing the modified date of the files to know which had changed (as in general, it would behave as if all the files were newer) but I knew that the source control versions were canonical, so if any of the files in source were different to the files on the server, the source control version should be used.

Since version 4, PowerShell has had a standard cmdlet for generating a hash value for a file, called `Get-FileHash`. The cmdlet was introduced primarily to help with Desired State Configuration (DSC). A Pull Server implementation of DSC will hash the configurations to determine if a change has occurred and then apply it. Per the above, as long as you're using a secure algorithm you can safely assume that if two files calculate different hash values they are different versions of the same file.

My cmdlet is relatively simple, but it has a couple of helpful features that my scenario required:

- You can provide a source path and use the `-Recurse` parameter to synchronise all modified files and folders in that path with a specified destination path. 
- If there are any files in the source path that are not in the destination, the cmdlet will copy those files across, as well as create any missing sub-folders beneath those files, as needed.

#### How does it work?

Here's a breakdown of the code:

```language-powershell
$SourcePath = If ($PSBoundParameters.ContainsKey('LiteralPath')) {
    (Resolve-Path -LiteralPath $LiteralPath).Path
}
Else {
    (Resolve-Path -Path $Path).Path
}

If (-Not (Test-Path $Destination)){
    New-Item -Path $Destination -ItemType Container | Out-Null
    Write-Warning "$Destination did not exist and has been created as a folder path."
}

$Destination = Join-Path ((Resolve-Path -Path $Destination).Path) -ChildPath '/'

$SourceFiles = (Get-ChildItem -Path $Source -Recurse:$Recurse -File).FullName
```
This initial stage handles whether the user has elected to use the `LiteralPath` or `Path` parameters.

> `-LiteralPath` is a standard parameter on a number of the built-in file handling cmdlets. By default `-Path` will interpret certain special characters as wildcard or other operators (square brackets for example). If you want to ensure your paths are handled explicitly, you should use the `-LiteralPath` parameter.

It then uses `Resolve-Path` to ensure that we have the full real path for both the source and destination paths (this proved to be necessary when Pester testing and using the `TestDrive:\` Pester path, which was handled poorly without converting it to it's actual path in the cmdlet).

If the `-Destination` path doesn't exist, it creates it as a folder and warns the user that we've done so. It then retrieves all files from the source path, including those in sub-folders if `-Recurse` has been used.

```language-powershell
ForEach ($Source in $SourcePath) {
    $SourceFiles = (Get-ChildItem -Path $Source -Recurse:$Recurse -File).FullName

    ForEach ($SourceFile in $SourceFiles) {
        $DestFile = Join-Path (Split-Path -Parent $SourceFile) -ChildPath '/'
        $DestFile = $DestFile -Replace "^$([Regex]::Escape($Source))", $Destination
        $DestFile = Join-Path -Path $DestFile -ChildPath (Split-Path -Leaf $SourceFile)

        $SourceHash = (Get-FileHash $SourceFile -Algorithm $Algorithm).hash

        If (Test-Path $DestFile) {
            $DestHash = (Get-FileHash $DestFile -Algorithm $Algorithm).hash
        }
        Else {
            If ($PSCmdlet.ShouldProcess($DestFile, 'New-Item')) {
                New-Item -Path $DestFile -Value (Get-Date).Ticks -Force | Out-Null
            }
            $DestHash = $null
        }

        If (($SourceHash -ne $DestHash) -and $PSCmdlet.ShouldProcess($SourceFile, 'Copy-Item')) {
            Copy-Item -Path $SourceFile -Destination $DestFile -Force:$Force -PassThru:$PassThru
        }
    }
}
```

The second part of the script iterates through each file found in the source path via a `ForEach` loop. It determines what the equivalent destination path would be for a file by doing a regular expression replace on the source path with the destination path (note that the regex pattern includes the `^` special character that ensures it only does this replace where it occurs from the beginning of the string). We have to use the .NET regex escape method to ensure that characters in the path are not mis-interpreted as regex special characters. This part of the script is also making use of `Split-Path` and `Join-Path` to deconstruct and reconstruct the path. By doing so, we allow PowerShell to handle how paths should be constructed and therefore make this cmdlet cross-platform compatible (e.g it will work via PowerShell Core on Ubuntu).

Having got the destination file path, we then check if it exists. If it doesn't we use `New-Item -Force` to create it, which has the helpful side-effect of also creating any sub-folders missing in that files path.

Now we use `Get-FileHash` to get the hash of each file (this is now possible even where the destination file didn't originally exist as `New-Item` has created it with a random value -- the reason for the random value is so that the file still gets copied in the later step, because if someone has used the scripts `-PassThru` parameter this then ensures that newly created destination files will also get returned, even where they are empty).

Finally we compare the two hash results and perform a copy where the results differ.

Note that the cmdlet is using `SupportsShouldProcess` so if you want to see a dry run result before you perform a copy, you can do so via the `-WhatIf` and all the destructive actions will be listed via verbose statements.

Again, if you would like to try `Copy-FileHash` out yourself, you can install it from the PowerShell Gallery via:
```
Install-Module HashCopy -Scope CurrentUser
```
If you'd like to review the code or you have any ideas for improvements, check out the GitHub repo [here](https://github.com/markwragg/PowerShell-HashCopy).