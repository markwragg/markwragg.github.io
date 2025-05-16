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
# This is a comment
Write-Host "Hello, world!" #This is also a comment
```
</div>
</td>
<td>
<div markdown="1">

```python
# This is a comment
print("Hello, World!") #This is also a comment
```

</div>
</td>
</tr>

<tr>
<td>Defining a string</td>
<td>
<div markdown="1">

```powershell
$text = 'sometext'
```

</div>
</td>
<td>
<div markdown="1">

```python
text = 'sometext'
```

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
