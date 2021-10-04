---
title: Powershell Slack Bot using the Real Time Messaging API
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2017/01/chat-bot-1.jpg"
date: '2017-01-03 16:13:34'
tags:
- powershell
- slack
---
This post details how PowerShell can be used to run a custom Slack Bot that utilises the Slack RTM (Real Time Messaging) API. 

Following on from my previous post where I set up a [Slack Slash Command using Azure Functions](http://wragg.io/a-slack-slash-command-using-powershell-azure-functions/), I wanted to provide more functionality to my team via a single multi function Slack Bot. Slack has three APIs: The Web API, The Events API and the RTM API. There are lots of [community frameworks](https://api.slack.com/community) for working with the Slack APIs in a variety of languages, but for PowerShell existing options seemed to be limited to the Web API.

> The full code for this project can be found [here in GitHub](https://github.com/markwragg/Powershell-SlackBot) where I warmly welcome contributions. To learn more about the how and the why, read on below.

### Why use the RTM API?

I actually got a workable solution implemented with the Web API, but it felt a little like bastardisation when using it to author a bot. I used the [PSSlack module by Warren F](http://ramblingcookiemonster.github.io/PSSlack/) to repeatedly read the message history of a slack Channel, monitoring for messages directed to the name of my bot and responding when appropriate. This was fine to a point, but apart from feeling like a misuse of the API it also had a few limitations:

- **I had to specify which Channel/s to monitor for messages.** This is fine if you want the Bot to only be present in specific channels. Also you could potentially monitor all channels by using the Web API to get a list of them, but as the message history of channels can be large, I was skeptical about how well this would scale for a busy Slack account.
- **The Web API can't read or respond to direct messages.** One of my users asked for this as a feature, so that they could interact with the Bot without always spamming other users.

The RTM API allows a bot to establish a websocket connection to the Slack API through which it receives an ongoing stream of messages that it is privy to (just like any other user does via whatever Slack client they use). The Bot connects and sees chat messages for channels it is present in, messages sent to it directly, user presence changes etc.

![](/content/images/2017/01/Slack-Bot-Slack-1.png)

### Caveats

- There's no native way to interact with Websockets via PowerShell. Instead, you need to utilise .NET and the [ClientWebSocket Class](https://msdn.microsoft.com/en-us/library/system.net.websockets.clientwebsocket(v=vs.110).aspx) of the [System.Net.WebSockets Namespace](https://msdn.microsoft.com/en-us/library/system.net.websockets(v=vs.110).aspx).
- System.Net.WebSockets is only available with Windows 8 / Server 2012 and newer Operating Systems.
- The RTM API can only send "simple" messages. However there's nothing stopping you from utilising the Web API if/when you want to send more advanced messages (e.g with [attachments](https://api.slack.com/docs/attachments)) and using [PSSlack](http://ramblingcookiemonster.github.io/PSSlack/) can make this easier. 

> I would not have figured out how to do this had I not come across [this code](https://github.com/brianddk/ripple-ps-websocket) by [brianddk](https://github.com/brianddk) which demonstrated the exact .NET concepts I needed within a PowerShell script, albeit for a different purpose.

### Getting Started

If you haven't already, you first need to log in to your Slack account and create a Bot under Custom Integrations > Bots. You then need the API token of your Bot, which I've elected to store in an XML file via `Export-Clixml` so that it's not hard coded in my script and instead read from a file:

```
Param(
    $Token = (Import-Clixml Token.xml)
)
```

### The code

As detailed in the [RTM API Documentation](https://api.slack.com/rtm), the script first makes a Web API call to the `rtm.start` method with the Bot's token:

```
$RTMSession = Invoke-RestMethod -Uri https://slack.com/api/rtm.start -Body @{token="$Token"}
Write-Verbose "I am $($RTMSession.self.name)"
```

> If this call is successful, `$RTMSession` will contain various properties that you might also find useful, including details of the Bot users name, ID, preferences, a list of channels, groups etc.

$RTMSession then has a `.URL` property which is the WebSocket URL needed to connect to the RTM API using the ClientWebSocket .NET class.

I have encapsulated the rest of the code in a Try block, because I wanted to make sure that the WebSocket was properly closed whenever I quit the script. This does happen automatically when the PowerShell process ends, but if you're repeatedly stopping and starting the the script within a single PowerShell window (or the ISE) it doesn't, so by using `Try` I could put clean up code in a `Finally` block which is executed when you quit the script with CTRL+C.

```
Try{
    Do{
        $WS = New-Object System.Net.WebSockets.ClientWebSocket                                                
        $CT = New-Object System.Threading.CancellationToken                                                   

        $Conn = $WS.ConnectAsync($RTMSession.URL, $CT)                                                  
        While (!$Conn.IsCompleted) { Start-Sleep -Milliseconds 100 }

        Write-Verbose "Connected to $($RTMSession.URL)"

        $Size = 1024
        $Array = [byte[]] @(,0) * $Size
        $Recv = New-Object System.ArraySegment[byte] -ArgumentList @(,$Array)
```
The above establishes the connection to the websocket. This is encapsulated in a Do loop, because one of the messages sent through the RTM API is a "reconnect_url" event, which gives us a new websocket URL to connect to every 30 seconds. This isn't necessarily used (the websocket connection should stay alive as long as we need it) but it gives a fallback way to reconnect should the connection drop, without having to recall the `rtm.start` method.

```
        While ($WS.State -eq 'Open') {

            $RTM = ""
        
            Do {
                $Conn = $WS.ReceiveAsync($Recv, $CT)
                While (!$Conn.IsCompleted) { Start-Sleep -Milliseconds 100 }

                $Recv.Array[0..($Conn.Result.Count - 1)] | ForEach { $RTM += [char]$_ }
       
            } Until ($Conn.Result.Count -lt $Size)

            Write-Verbose "`n$RTM"
```
If we have a successful connection to the websocket, we next read the current message through it using the `ReceiveAsync` method of the .NET class in to `$RTM` as JSON.

If we have received a message, we convert it from JSON and handle it:

```
            If ($RTM){
                $RTM = ($RTM | convertfrom-json)
                    
                Switch ($RTM){
                    {($_.type -eq 'message') -and (!$_.reply_to)} { 
                        
                        If ( ($_.text -Match "<@$($RTMSession.self.id)>") -or $_.channel.StartsWith("D") ){
                            #A message was sent to the bot
                            
                            # *** Responses go here, for example..***
                            $words = ($_.text.ToLower() -split " ")
                            
                            Switch ($words){
                                {@("hey","hello","hi") -contains $_} { Send-SlackMsg -Text 'Hello!' -Channel $RTM.Channel }
                                {@("bye","cya") -contains $_} { Send-SlackMsg -Text 'Goodbye!' -Channel $RTM.Channel }

                                default { Write-Verbose "I have no response for $_" }
                            }

                        }Else{
                            Write-Verbose "Message ignored as it wasn't sent to @$($RTMSession.self.name) or in a DM channel"
                        }
                    }
                    {$_.type -eq 'reconnect_url'} { $RTMSession.URL = $RTM.url }
                        
                    default { Write-Verbose "No action specified for $($RTM.type) event" }            
                }
```
In the above we look for messages that were either in a Direct Message channel (e.g from any user, privately to the Bot user) or were in any public Channel that the Bot has been invited to and where the message includes @*botname* (where the name of the Bot was determined when we invoked the `rtm.start` method).

If either of these are true, then a Switch statement looks for opportunities to respond (see below for the `Send-SlackMsg` function). This is obviously where you extend the Bot with the responses and functionality that you want it to perform.

Here we could also add Switch logic to handle other types of messages, e.g if we wanted the Bot to take some action for any of the other [RTM Events](https://api.slack.com/rtm).

![](/content/images/2017/01/Slack-Bot.png)

The code loops continuously, reading messages from the websocket instantly whenever they are available, until the scripts execution is forcibly stopped:
```
            }
        }   
    } Until (!$Conn)

}Finally{

    If ($WS) { 
        Write-Verbose "Closing websocket"
        $WS.Dispose()
    }

}
```
In lieu of any other activity, you'll see the Bot receive a "reconnect_url" event every 30 seconds, which I suspect also serves to keep the connection alive.

### Functions

This function uses the established websocket to send simple slack messages back to any Channel specified (including Direct Message channels) so that we can respond to messages:
```
Function Send-SlackMsg
{
    [cmdletbinding()]
    Param(
        $Text,
        $Channel,
        $ID = (get-date).ticks
    )
    
    $Prop = @{'id'      = $ID;
              'type'    = 'message';
              'text'    = $Text;
              'channel' = $Channel}
            
    $Reply = (New-Object –TypeName PSObject –Prop $Prop) | ConvertTo-Json
            
    $Array = @()
    $Reply.ToCharArray() | ForEach { $Array += [byte]$_ }          
    $Reply = New-Object System.ArraySegment[byte]  -ArgumentList @(,$Array)

    $Conn = $WS.SendAsync($Reply, [System.Net.WebSockets.WebSocketMessageType]::Text, [System.Boolean]::TrueString, $CT)
    While (!$Conn.IsCompleted) { Start-Sleep -Milliseconds 100 }

    Return $ID
}
```
As noted earlier, we can only reply with the simple message format in this way. To use any other message formatting type (e.g attachments) you'll need to fall back to the Web API.

My basic Bot doesn't make use of the below function (which I did not write), but I wanted to share it as it is useful if you want to convert the timestamp (TS) properties you get with some of the Slack events in to a PowerShell DateTime object. 

```
Function ConvertFrom-UnixTime {
    param(
        [Parameter(Mandatory=$true,ValueFromPipeline=$true)]
        [Int32]$UnixTime
    )
    begin {
        $startdate = Get-Date –Date '01/01/1970' 
    }
    process {
        $timespan = New-Timespan -Seconds $UnixTime
        $startdate + $timespan
    }
}
```
This function is also in the PSSlack module where it's credited as being from [Powershell.com](http://community.idera.com/powershell/powertips/b/tips/posts/converting-unix-time).

### Contributions

This project is in Github as [Powershell-SlackBot](https://github.com/markwragg/Powershell-SlackBot) and I welcome pull requests or suggestions to help improve it. It is now also listed on the [Slack Community page](https://api.slack.com/community#powershell) under Powershell.