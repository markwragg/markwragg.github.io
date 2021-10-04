---
title: 'Send notifications to Hipchat with Powershell #ChatOps'
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2016/06/hc.jpg"
date: '2016-06-04 13:32:51'
---
I recently implemented a Powershell module to send notifications in to our Hipchat rooms. This post explains how that script works and why this was an important shift for how we handle notifications.

A number of our administrative scripts were historically configured to send emails when they performed key activities to ensure these automations were visible. As with most teams, we receive a lot of email so I suspect (like me) most of the team eventually configured rules to have these emails auto-filed, hiding them and negating the intended effect. 

While most teams are probably already utilising some form of electronic chat mechanism  (~~Lync~~ Skype for Business etc.) these tools are typically designed to just permit communication. Hipchat (and other tools like it such as Slack) extend this by allowing integrations to be built that make the chat tool more powerful, useful and unavoidably central to how operations occur. Sending notifications in to chat rooms is the most basic of these integrations.

![Hipchat](/content/images/2016/06/HipchatShot-wide-1.png)

*-- These tools are an enabler for a cultural movement known as "[ChatOps](http://blogs.atlassian.com/2016/01/what-is-chatops-adoption-guide/)". [Popularised by Github](https://www.youtube.com/watch?v=NST3u-GjjFw), the idea is that the chat room becomes a place where work is not only discussed but occurs and where the result of that work can then be known by the entire team.*

There are several existing third-party Powershell modules available for sending notifications to Hipchat, but I was interested in the challenge of developing my own. Hipchat themselves provided guidance for using Powershell for sending notifications, but the guidance was based around version 1 of their API, which is now deprecated. My module is named Powershell-HipChat and is available on Github here: 

> https://github.com/markwragg/Powershell-Hipchat

The code operates via a function named `send-hipchat` which is included in the Powershell module file: hipchat.psm1 (which also comes with a Module Manifest psd1 file). 

Before you can make use of the function you'll need two things from Hipchat: the name or ID of the room you want to send notifications in to and an API token for that room.

**To get the ID or name of your room:**

1. Log in to the Hipchat web interface and browse to "Rooms". If you're not self-hosting, that will be https://yourname.hipchat.com/rooms
2. Click on the room you want. You can use the friendly name of that room if you wish (ensure you encapsulate it in speech marks if it includes spaces) but I recommend you instead use the ID of the room, which you see on the Summary page listed as "API ID" as well as in address bar for that page. By using the ID you won't be affected if the room is renamed.

**To get an API token:**

*Beware that the script uses version 2 of the API, and you cannot use version 1 API tokens. The version 1 API tokens were global, where as version 2 tokens are specifically created for each individual room.* 

1. Within the room's page (per the above steps) navigate to "Tokens".
2. Complete the "Create a new token" form. The "Label" you choose will appear with each notification, so you may want to use it to describe the source activity of the notification.
3. The scope should be "Send notification".

If you want to send messages to multiple rooms, you must create a token for each room. You can also create multiple tokens per room, which you can then label for different purposes.

**To use the function:**

1. [Download/clone the files](https://github.com/markwragg/Powershell-Hipchat) to where you wish to use them.
2. Load the module via the manifest with `Import-module hipchat.psd1`
3. Execute the command (or include it in your script), e.g: `send-hipchat -message "It was the best of times. It was the worst of times." -color "random" -apitoken <your apitoken> -room <your room id> -retry 5 -retrysecs 30`

Here's the result in Hipchat:

![Hipchat notification examples](/content/images/2016/06/Hipchat-Notification-Examples-1.png)

## The code

After an initial implementation that relied on cmdlets available from Powershell 3 and newer, I discovered that some of our environment was still running on Powershell 2. While I intend to upgrade those servers in the near future, in the interim I updated the script to remove those dependencies. That does add quite a lot of bloat, so I've made it somewhat explicit where that has occurred so that anyone who is sure that they are running Powershell 3 or above can easily strip out the Powershell 2 sections.

The first of those is that Powershell 2 does not have a convertto-json function, but after some googling I was able to retro-fit one via the below function that is only declared if the version is less than Powershell 3:

```language-powershell
if ($PSVersionTable.PSVersion.Major -lt 3 ){
          
    function ConvertTo-Json([object] $item){
        add-type -assembly system.web.extensions
        $ps_js=new-object system.web.script.serialization.javascriptSerializer
        return $ps_js.Serialize($item)
    }

}
```
*--- The above function (or the PS3+ cmdlet) is needed because Hipchat requires the message it receives to be formatted as JSON.*

After this, the rest of the module is the `send-hipchat` function:

```language-powershell
function Send-Hipchat {
    [CmdletBinding()]
```
The below parameters have validation and sensible defaults where it seemed appropriate. For example there is only a specific list of colours that are valid.

I added retry functionality to the script because my initial testing was a bit inconsistent in delivering the message first time, but this may well have been as a result of my infrastructure rather than a failing on the part of Hipchat. I left this functionality in in case it was useful in the future, but by default retry = 0 which means it will only try to deliver it once. If you do set a retry, you can configure the delay between tries with retrysecs (30 by default):
```language-powershell
    Param(
        [Parameter(Mandatory = $True)][string]$message,
        [ValidateSet('yellow', 'green', 'red', 'purple', 'gray','random')][string]$color = 'gray',
        [switch]$notify,
        [Parameter(Mandatory = $True)][string]$apitoken,   
        [Parameter(Mandatory = $True)][string]$room,
        [int]$retry = 0,
        [int]$retrysecs = 30
    )
```
This next part declares a hash table of your notification parameters. The Notify parameter defines whether active users in the room get a notification (i.e an unread message indicator, or other notification per their preferences) that the message has been added to the room. If notify is set to false, the message is effectively silent.
```language-powershell
    $messageObj = @{
        "message" = $message;
        "color" = $color;
        "notify" = [string]$notify
    }
```
The API URL for Hipchat is standard unless you are self-hosting. If you are, change the URL below:
```language-powershell        
    $uri = "https://api.hipchat.com/v2/room/$room/notification?auth_token=$apitoken"
    $Body = ConvertTo-Json $messageObj
    $Post = [System.Text.Encoding]::UTF8.GetBytes($Body)
        
    $Retrycount = 0
    
    While($RetryCount -le $retry){
	    try {
```
If Powershell 3 or above is available, delivering the message is a one line use of `invoke-webrequest`. If not, it's a slightly more complicated case of using a .NET equivalent:
```language-powershell
if ($PSVersionTable.PSVersion.Major -gt 2 ){
   $Response = Invoke-WebRequest -Method Post -Uri $uri -Body $Body -ContentType "application/json"
}else{
   $Request = [System.Net.WebRequest]::Create($uri)
   $Request.ContentType = "application/json"
   $Request.ContentLength = $Post.Length
   $Request.Method = "POST"

   $requestStream = $Request.GetRequestStream()
   $requestStream.Write($Post, 0,$Post.length)
   $requestStream.Close()

   $Response = $Request.GetResponse()
   $stream = New-Object IO.StreamReader($Response.GetResponseStream(), $Response.ContentEncoding)
   $content = $stream.ReadToEnd()
   $stream.Close()
   $Response.Close()
}
Write-Verbose "'$message' sent!"
Return $true
```
If any error has occurred, we report it to the console and if retry has been set, wait retrysecs before looping. The If logic below ensures we don't wait after the last attempt:
```language-powershell
        } catch {
            Write-Error "Could not send message: `r`n $_.Exception.ToString()"

             If ($retrycount -lt $retry){
                Write-Verbose "retrying in $retrysecs seconds..."
                Start-Sleep -Seconds $retrysecs
            }
        }
        $Retrycount++
    }
```
If we haven't successfully delivered after the maximum retries we inform the user via the console and `return $false`. This is is important as it allows logic to be built around the function in the calling script to determine if it was successful (and to perhaps fall back to email notification if it was not).

Note the below lines don't get executed when we're successful as the earlier `return $true` ends the script immediately.
```    
    Write-Verbose "Could not send after $Retrycount tries. I quit."
    Return $false
}
```

## Usage suggestions

Your use cases will vary depending on what suits your environment. It's a good idea to push out notifications from any automation that is affecting change in your environment to ensure these changes are visible to appropriate audiences. Another way in which notifications are commonly used as part of ChatOps is to inform the team on the progress of the automated build/test process of releases.

Here's some automations that we've embedded notifications in to:

- **Windows updates:** We use a script in our environment that triggers Windows updates (rather than directly by Windows) because it allows us to also temporarily pause monitoring. A HipChat notification lets us see exactly when each server has rebooted, so if there is a negative effect we immediately know why.
- **Software customisations:** Some of our software is customised via a standard installer and our Centralised Operations team can schedule these installations around when customer's want them to occur. We've added notifications that tell us when an installation has been scheduled and then that sends or a red or green notification after the installation reporting failure or success.
- **Storage vMotions:** It's occasionally necessary to relocate multiple Virtual Machines between datastores. To ensure this doesn't cause disruption we have a script (thanks to [Sam Martin](http://sammart.in) that moves a list of VMs one at a time (saving us having to trigger each move manually). To monitor the progress of this each move drops a notification to Hipchat.