---
title: PowerShell vs Python
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2025/code.jpg"
  teaser: "/content/images/2025/code.jpg"
date: '2025-05-15 09:00:00'
tags:
- powershell
- python
---

PowerShell and Python are similar languages.

<style type="text/css">
  td { vertical-align: top; }
</style>

<table>

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

<tr><td colspan="3">
<div markdown="1">
### Variables

- In both languages, a variable is created the moment you first assign a value to it.
</div>
</td></tr>

<tr>
<td>Creating variables</td>
<td>
<div markdown="1">

```powershell
$myVariable = 5
$text = 'sometext' #strings can be declared with ' or "
```
PowerShell variables are case-insensitive, `$text` is the same as `$Text`.

</div>
</td>
<td>
<div markdown="1">

```python
myVariable = 5
text = 'sometext' #strings can be declared with ' or "
```
Python variables are case-sensitive `text` is **not** the same as `Text`.

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
