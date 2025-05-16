---
title: PowerShell vs Python
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2025/code.jpg"
  teaser: "/content/images/2025/code.jpg"
date: '2025-05-15 09:00:00'
toc: true
toc_label: "Code concepts"
toc_icon: "code"
tags:
- powershell
- python
---

PowerShell and Python are popular programming languages, with a lot of similarities. PowerShell is commonly referred to a shell scripting language (more akin to Bash) but functionally has a lot in common with Python, and can be used to generate scripts of equal complexity.

As someone who has a strong familiarity with PowerShell, I'm finding it useful to reference the concepts of Python against their PowerShell equivalents. [Adam Driscoll did this previously in 2020 and his page was incredibly helpful](https://blog.ironmansoftware.com/powershell-vs-python/). Below I've created my own (following a similar approach, referencing the concepts covered by [W3Schools](https://www.w3schools.com/python/default.asp)) and comparing them side by side with the PowerShell equivalent, to help cement my knowledge as I learn Python.

> The PowerShell examples given below have been tested as working in PowerShell 7.5.

<style type="text/css">
  td { vertical-align: top; }
</style>

<table>

<tr>
<td colspan="3">
<div markdown="1">
### Getting started

- The latest version of PowerShell is cross-platform and can be [installed on Windows, MacOS and Linux](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell?view=powershell-7.5).
- Python can also be [installed on a variety of platforms including Windows, MacOS and Linux](https://www.python.org/downloads/).
</div>
</td>
</tr>

<tr width="100%">
<th width="20%">Concept</th>
<th width="40%">PowerShell</th>
<th width="40%">Python</th>
</tr>


<tr>
<td>Check installed version</td>
<td>
<div markdown="1">

```powershell
$PSVersionTable
```
```plaintext
Name        Value
----        -----
PSVersion   7.4.7
..
```

</div>
</td>
<td>
<div markdown="1">

```python
python --version
```
```plaintext
Python 3.13.3
```

</div>
</td>
</tr>

<tr>
<td>Hello World</td>
<td>
<div markdown="1">

```powershell
Write-Host "Hello, world!"
```
```plaintext
Hello, World!
```
Alternatively use `Write-Output` to return a PowerShell object.

</div>
</td>
<td>
<div markdown="1">

```python
print("Hello, World!")
```
```plaintext
Hello, World!
```

</div>
</td>
</tr>

<tr>
<td>Comments</td>
<td>
<div markdown="1">

```powershell
#This is a comment
Write-Host "Hello, world!" #This is also a comment

<# This is a
multiline comment #>
```
</div>
</td>
<td>
<div markdown="1">

```python
#This is a comment
print("Hello, World!") #This is also a comment

# This is a
# multiline comment
```
Unofficially, you can also use `'''` for multiline comments, which are ignored unless used as [docstrings](https://www.geeksforgeeks.org/python-docstrings/).

</div>
</td>
</tr>

<tr>
<td colspan="3">
<div markdown="1">
### Variables

- In both languages, a variable is created the moment you first assign a value to it.
- Variables do not need to be declared with a specific type, and can change type after being set. You can however use casting to specify the data type.
</div>
</td>
</tr>

<tr width="100%">
<th width="20%">Concept</th>
<th width="40%">PowerShell</th>
<th width="40%">Python</th>
</tr>

<tr>
<td>Creating variables</td>
<td>
<div markdown="1">

```powershell
$myVariable = 5
$text = 'sometext' #strings can be declared with ' or "
```
PowerShell variable names are case-insensitive, `$text` is the same as `$Text`.

</div>
</td>
<td>
<div markdown="1">

```python
myVariable = 5
text = 'sometext' #strings can be declared with ' or "
```
Python variable names are case-sensitive `text` is **not** the same as `Text`.

</div>
</td>
</tr>

<tr>
<td>Casting</td>
<td>
<div markdown="1">

```powershell
$x = [string]3
$y = [int]3
$z = [float]3
```
</div>
</td>
<td>
<div markdown="1">

```python
x = str(3)
y = int(3)
z = float(3)
```

</div>
</td>
</tr>

<tr>
<td>Assign multiple values</td>
<td>
<div markdown="1">

```powershell
$x,$y,$z = 1,2,3
```
</div>
</td>
<td>
<div markdown="1">

```python
x,y,z = 1,2,3
```

</div>
</td>
</tr>

<tr>
<td colspan="3">
<div markdown="1">
### Conditions and If statements
</div>
</td>
</tr>

<tr>
<td>if / elseif / else</td>
<td>
<div markdown="1">

```powershell
$a = 33
$b = 200

if ($b -gt $a) {
  Write-Host "b is greater than a"
}
elseif ($a -eq $b) {
  Write-Host "a and b are equal"
}
else {
  Write-Host "a is greater than b"
}
```

</div>
</td>
<td>
<div markdown="1">

```python
a = 33
b = 200

if b > a:
  print("b is greater than a")

elif a == b:
  print("a and b are equal")

else:
  print("a is greater than b")

```

</div>
</td>
</tr>

</table>
