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

<table width="100%">

<tr>
<th>Concept</th>
<th>PowerShell</th>
<th>Python</th>
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
