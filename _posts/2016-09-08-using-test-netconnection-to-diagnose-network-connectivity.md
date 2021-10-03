---
layout: post
title: Using Test-NetConnection to diagnose network connectivity
image: "/content/images/2016/06/connectivity.png"
date: '2016-09-08 20:53:00'
tags:
- powershell
---

Powershell version 4 and Windows 8.1 / 2012 R2 introduced the `Test-NetConnection` command as a tool for performing network connectivity tests with Powershell. `Test-NetConnection` allows you to perform ping, traceroute and TCP port tests and from Windows 10 and Server 2016 onward introduces the ability to do "Diagnose Routing" tests with the same cmdlet.

> The Test-NetConnection cmdlet displays diagnostic information for a connection. It supports ping test, TCP test, route tracing, and route selection diagnostics. Depending on the input parameters, the output can include the DNS lookup results, a list of IP interfaces, IPsec rules, route/source address selection results, and/or confirmation of connection establishment.
>
> *Source: https://technet.microsoft.com/en-us/library/dn372891.aspx*

The official documentation indicates that there is no "default value" for the `-ComputerName` parameter (the destination of your tests), but it does seem to actually default to  internetbeacon.msedge.net if not specified. Other parameters for this command vary depending on which type of testing you want to do.

# Test-NetConnection

- Uses `Write-Progress` for status messages and `Write-Warning` to report failure states (Ping or TCP connect failed) to the console.
- There is no way to define a timeout value for any of the tests.
- The cmdlet always does a ping test and name resolution, even if you are using it to test a TCP port or to perform a traceroute (you get both results, with ping and name resolution seemingly being performed first which can slow the performance of the cmdlet overall).
- You can use `-InformationLevel Quiet` to simplify the result to true/false:

> `-InformationLevel`: If you set this parameter to Quiet, the cmdlet returns basic information. For example, for a ping test, this cmdlet returns a Boolean value that indicates whether the attempt to ping a host or port is successful.

#### Ping / ICMP testing

Perform a simple ping test (although as noted below in my section on tools for other Windows versions, test-connection might be a better choice for this as it's more flexible):
```
Test-NetConnection google.com
```
#### TCP port testing

Perform a test to a Common TCP port by name using the `-CommonTCPPort` parameter (options are only the following: HTTP, RDP, SMB, WINRM) or to any numbered TCP port by using the `-Port` parameter, examples:
```
Test-NetConnection wragg.io -CommonTCPPort HTTP
Test-NetConnection twitter.com -Port 443
Test-NetConnection localhost -Port 445
```

![](/content/images/2016/09/test-netconnection-port443.png)

#### Traceroute testing 
Perform a traceroute (from Windows 10 / Server 2016 only) you can use the `-Hops` parameter to define a maximum number of hops:
```
Test-NetConnection sammart.in -TraceRoute
```
![](/content/images/2016/09/test-netconnection-traceroute.png)

Here's how you might use this to get the second to last hop of a traceroute:
```
(Test-NetConnection google.com -TraceRoute).TraceRoute | Select -SkipLast 1 | Select -Last 1
```
# Examples

Here's a Gist with various other examples expanding on those described above:

<script src="https://gist.github.com/markwragg/78e923e9e7a1d5aa8a6acdcc2dc6bdce.js"></script>

# Tools for other Win / Server versions

There are some alternative options for doing similar levels of testing as `Test-NetConnection` allows with versions of Windows or Powershell that don't support the cmdlet.

#### Test-Connection
The `Test-Connection` cmdlet was introduced in Powershell 3 and allows you to do a simple ping test. For this purpose it's much more versatile than Test-NetConnection as you can define the TTL (Time To Live), Packet Size, Delay and other settings as well as use it in `-quiet` mode to suppress messages.

#### System.Net.Sockets
For performing TCP port tests, you can use the .NET class System.Net.Sockets.TcpClient, which can be particularly advantageous if you don't want to install the Telnet Client.

`(New-Object System.Net.Sockets.TcpClient).Connect('{remote server}', {port})`

*Source: http://www.travisgan.com/2014/03/use-powershell-to-test-port.html*

#### Poor man's Powershell Traceroute
Below is a function found on Stack Overflow written by Mathias R Jessen which provides a way to do a traceroute via Powershell without the Test-NetConnection function:

*Source: http://stackoverflow.com/questions/32434882/powershell-tracert*

```language-powershell
function Invoke-Tracert {
    param([string]$RemoteHost)

    tracert $RemoteHost |ForEach-Object{
        if($_.Trim() -match "Tracing route to .*") {
            Write-Host $_ -ForegroundColor Green
        } elseif ($_.Trim() -match "^\d{1,2}\s+") {
            $n,$a1,$a2,$a3,$target,$null = $_.Trim()-split"\s{2,}"
            $Properties = @{
                Hop    = $n;
                First  = $a1;
                Second = $a2;
                Third  = $a3;
                Node   = $target
            }
            New-Object psobject -Property $Properties
        }
    }
}
```
*Script author: [Mathias R Jessen](http://stackoverflow.com/users/712649/mathias-r-jessen)*