---
title: Random
image: "/content/images/2017/06/dice_game_black_background_close_up__u_2560x1600.jpg"
---
Getting a single random item is easy

    Get-Service | Get-Random

But what if you want to return more than one? You could do a loop: 

    1..3 | ForEach-Object { Get-Service | Get-Random }

But you might get the same one come back twice. What if you want them to be unique?

You can do this:

    Get-Service | Sort-Object {Get-Random} | Select -First 3

Source: http://ilovepowershell.com/2015/01/24/easiest-way-shuffle-array-powershell/

1..10 | ForEach-Object { Get-Random -Min 1 -Max 10 }

$Heroes | Sort-Object {Get-Random}

Get-Random -Min 1 -Max 100


$Heroes = @('Tony','Bruce','Clark','Peter','Diana')

Get-Random $Heroes -Count $Heroes.count

$Heroes | Sort-Object {Get-Random}



1..10 | Get-Random -Count ([int]::MaxValue)