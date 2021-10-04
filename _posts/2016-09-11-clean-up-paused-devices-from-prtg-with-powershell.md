---
title: Cleaning up paused devices from PRTG with Powershell
header:
  overlay_image: "/content/images/2016/09/isometricBG_5.jpg"
date: '2016-09-11 18:50:00'
tags:
- powershell
- prtg
---
One of our AWS based products uses auto-scaling and when new instances are deployed a script automatically creates sensors for them in PRTG (our monitoring tool). When the instances are scaled down/terminated there is not a script that automatically removes them from PRTG (in part so that we can temporarily retain the monitoring history). As a result, our monitoring over time can become cluttered with redundant paused devices, so a script was needed to automate the process of clearing those down.

As covered previously, PRTG has a HTTP based API that provides a way to query the configuration and return results as JSON (my preference) or XML or to manipulate objects by passing parameters through the querystring.

After I had the basic needed functionality written in Powershell, I underwent several revisions as I was keen to try out (and understand the implementation of) some Powershell functionality that I hadn't used much in the past. 

# Whatif and Confirm

The first of those was to add `-whatif` and `-confirm` functionality to the script. In case you're not aware: 

- `-whatif` makes a cmdlet print out the changes it would make without making them.
- `-confirm` makes a cmdlet prompt for confirmation before making each change (with the option to say yes/no to all).

Adding the following to the top of a script or function:
```
[CmdletBinding()]
```
Enables you to use common parameters such as `-verbose` and `-debug` to turn on and off any `write-verbose` and `write-debug` messages you've included.

Extending this to be:
```
[CmdletBinding(SupportsShouldProcess=$True,ConfirmImpact=Medium)]
```
Is the first step to enabling `-confirm` and `-whatif`. You don't have to specify `ConfirmImpact` (Medium is the default I believe) but this parameter controls whether confirm is on or off by default and can be set to [High, Medium, Low or None](https://msdn.microsoft.com/en-us/library/system.management.automation.confirmimpact(v=vs.85).aspx).

As PRTG doesn't provide an easy way to reverse the deletion of an object (you'd need to restore the whole configuration database from a backup) I've set `ConfirmImpact=High`. As a result, my script will always prompt for confirmation even when `-confirm` hasn't been explicitly set. Doing this allowed me to strip out of my script a manual "Are you sure?" check via `read-host`.

> It's worth noting that if a user changes their `$confirmpreference` variable to "none" they'll override my default.

The second step needed to fully implement `-whatif` and `-confirm` is to identify what parts of your script are destructive and surround them with the following `if` block:

```
If ($PSCmdlet.ShouldProcess("$($_.device)","PRTG: Delete Object"))
{
   Destructive bit of code..
}
```
Two parameters you pass to `ShouldProcess` (as shown above) populate the whatif message with what is being affected and what the effect is. And that's it, `-whatif` and `-confirm` parameters implemented!

![](/content/images/2016/09/RemovePRTGPausedDevices-WhatIf-Confirm.png)

# Script to get paused PRTG devices and remove them
At this point I had a single script for getting a list of paused devices based on several ways that I wanted to filter for those (which the script accepts as parameters):

- `-name` filters the list of paused devices to those that partially match the text provided in the device name field
- `-message` filters the list of paused devices to those that partially match the text provided in the message (comment added by a user) field. This was useful for me because we had a process where certain sensors were paused referenced a particular runbook ID.
- `-dayspaused` filters the list of paused devices to just those that have been paused at least this many days.

These filters can be combined. For example to match any devices partially named "LON" with "Paused by me" in the message field that have been paused for 7 days or longer you'd run the script with:

```
-name "LON" -message "Paused by me" -dayspaused 7 -prtgurl prtg.mycompany.com -username myuser -passhash 12345
```
Noting that (as above) you also need to include the URL to your server (excluding https://) and a username and passhash that has permissions to query the API. Here's the script:

<script src="https://gist.github.com/markwragg/18ef05965e2f6b2cf0ea150e7c52d487.js"></script>

# Getters and Setters

From here, it occurred to me that as I was doing two distinct actions a "Get" type action to retrieve and filter the data I wanted and a "Set" type action to make changes via that collection (in this case a Remove action) the script warranted breaking in to those components, with the ability to use the pipeline to communicate objects between them.

Making these seperate functions makes them more flexible, but ultimately what I wanted was the original goal to continue to be as simple as 

```
Get-PRTGPausedDevices -name "somedevices" | Remove-PRTGPausedDevices
```

I actually ended up with 3 functions and here's why:

1. I obviously needed a `Get-PRTGPausedDevices` function to do the grunt work of querying the PRTG API, filtering the results and returning that as a Powershell object
2. I needed a very simple `Remove-PRTGDevices` function that I could pass the results to and trigger the removal from PRTG (I even descoped the "Paused" part at this point and figured this function could  be simplified to just remove anything you want from PRTG). But it occurred to me it still made sense to have..
3. A `Remove-PRTGPausedDevices` function that removed Paused devices, but that could also be used in isolation to also do the get/filtering part per the first function. The reason this made sense to me is because (for example) you can use the `Remove-Item` cmdlet on it's own to delete files. You don't have to use `Get-ChildItem` and pipe the results to it. But you can do that. So what I wanted was a similar function, that could be used on it's own via parameters (per my original script) but that could also accept pipeline input from my Get-PRTGPausedDevices function or from any other source that might be suitable in the future.

# PRTG paused devices module
Here's the 3 functions described above packaged as a module. To use them, simply download the file, load it with:
```
Import-Module prtgPausedDevices.psml
```
and then execute the above listed cmdlet names.
<script src="https://gist.github.com/markwragg/cd9fd02ad539fb5dd4adfa6add0909bc.js"></script>

# Things I learned

I had not developed functions in this style before and I learnt a few things along the way. I don't think the above module represents "best practice", particularly on the issue of credentials that I will cover later. Finding some indication of best practice for this sort of thing turned out to be very difficult, so the above is a working solution with some choices made that I will describe below.

Implementing pipeline input for my `Remove-` functions was a simple case of declaring one or more parameters as
```
[int][Parameter(ValueFromPipelineByPropertyName=$True)]$objid
```
Which I did as a minimum with $objid so that I had a unique identifier with which to perform the delete actions against. `ValueFromPipelineByPropertyName` makes the script automatically match any incoming properties to the same named parameters, but what's cool is that if you're also passing in objects that have other data that data isn't lost and is available for use by the script. This meant that when I was piping in the Device name, I could use it without it being explicitly defined as a parameter.

You can optionally choose to structure your function with the following blocks (Begin and End are optional):

```
Begin{
    Things that happen once before anything else 
    (including before there being any pipeline input).
}
Process{
    Thing that happens for each pipeline object.
}
End{
    Thing that happens once at the end.
}
```

There's some efficiency to working this way, but there's some caveats too:

- You can't count the total of how many objects have been piped in to the function or (as far as I could tell) keep a track of how far through the list of objects you were (e.g to use with `write-progress`). 
- You also can't work with the pipeline input at all in the Begin phase as at this point it doesn't exist. That being said I liked having these 3 structural blocks so used this structure for my more indepth Remove-PRTGPausedDevices function. 
- It meant I could use the Begin section to perform the "Get-" behaviour if there was not pipeline input, based on any defined parameters I also had to include a ForEach-Object loop in to the Process section (despite that section looping by design) to cover where there was no Pipeline input and the function was generating its own input.

For the smaller/simpler `Remove-PRTGDevices` function, I went with the other option which was to not have Begin..Process..End blocks at all. Instead you use a ForEach-Object block to loop through the pipeline and you can use the default $Input object to query the collection, so can get a count of how many objects there are and report progress via `Write-Progress`.
![](/content/images/2016/09/Remove-PRTGDevices.png)
*-- I still use `Write-Progress` to give feedback in the other function, I just can't show progress through the total. However as the web calls can be slow it still seemed worthwhile to use it to give the user feedback.*

The final functionality I wanted to include is likely the most controversial. I wanted it so that I could accept the PRTGURL, Username and Passhash parameters via the Pipeline from the other cmdlets. So that I could have:
```
Get-PRTGPausedDevices -PRTGURL prtg.myserver.com -Username Mark -Passhash 12345 | Remove-PRTGPausedDevices -WhatIf
```
Rather than:
```
Get-PRTGPausedDevices -PRTGURL prtg.myserver.com -Username Mark -Passhash 12345 | Remove-PRTGPausedDevices -PRTGURL prtg.myserver.com -Username Mark -Passhash 12345 -WhatIf
```

Putting aside the issue of passing credentials in plaintext (I played around with ConvertTo/From-SecureString for a bit but quickly came unstuck) deciding the right way to communicate the credentials between the Functions was difficult.

One option was to just define them as Globally scoped variables in the `Get-` function with e.g 
```
Set-Variable -Name PRTGURL -Value $PRTGURL
```
So that they were then accessible to the other functions. I then cleaned them up with `Remove-Variable` in the End section, but if someone hits CTRL+C during processing they are never cleaned up.

The solution I went with in the end was to simply include them as additional properties of each object during the `Get-` function as so:
```
$_ | Add-Member –MemberType NoteProperty –Name "PRTGURL" –Value $PRTGURL -Force
$_ | Add-Member –MemberType NoteProperty –Name "username" –Value $Username -Force
$_ | Add-Member –MemberType NoteProperty –Name "passhash" –Value $Passhash -Force
            
```
The downside is that the credentials are duplicated horribly, once for each object in the pipeline.

The upside is that I can accept them automatically in the `Remove-` functions simply by having them as Parameters that accept Pipeline input. Further I can make these parameters Mandatory and then if they are coming in via the Pipeline the user isn't prompted for them, but if they aren't provided via the pipeline the user has to provide them. This wasn't possible (or as easy anyway) with them as Global variables.

I did briefly consider having a nested object, so that I could have the sort of parent properties (PRTGURL and credentials) and then a child object that had all of the devices. This broke some functionality i'd added earlier, which was the below (from the End block), which sets a nice default view output for the `Get-PRTGPausedDevices` function where it only displays the device name, message and paused date and using `Format-Table` by default:
```
$Devices.PSObject.TypeNames.Insert(0,'PRTG.PausedDevice')
$defaultDisplaySet = 'device','message','paused'
$defaultDisplayPropertySet = New-Object System.Management.Automation.PSPropertySet(‘DefaultDisplayPropertySet’,[string[]]$defaultDisplaySet)
$PSStandardMembers = [System.Management.Automation.PSMemberInfo[]]@($defaultDisplayPropertySet)
$Devices | Add-Member MemberSet PSStandardMembers $PSStandardMembers
```
![](/content/images/2016/09/Get-PRTGPausedDevices-Format-Table.png)
I'm sure it is possible to get this functionality working even where there's a nested Object, but trying to do so started to make the whole thing horribly complex.

I'm also sure there's a better answer to handling credentials (or "parent" type properties) between functions via the pipeline, so if you know it please comment! Overall i'm pretty happy with the result and (until I know better) I would use this framework in the future for developing similar functionality.