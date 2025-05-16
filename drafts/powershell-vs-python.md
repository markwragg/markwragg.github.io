---
title: PowerShell vs Python
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2025/code.jpg"
  teaser: "/content/images/2025/code.jpg"
date: "2025-05-15 09:00:00"
toc: true
toc_label: "Code concepts"
toc_icon: "code"
tags:
  - powershell
  - python
---

PowerShell and Python are popular programming languages, with a lot of similarities. PowerShell is commonly referred to as a shell scripting language (more akin to Bash) but functionally has a lot in common with Python, and can be used to generate scripts of equal complexity.

As someone who has a strong familiarity with PowerShell, I'm finding it useful to reference the concepts of Python against their PowerShell equivalents. [Adam Driscoll did this previously in 2020 and his page was incredibly helpful](https://blog.ironmansoftware.com/powershell-vs-python/). Below I've created my own (following a similar approach, referencing the concepts covered by [W3Schools](https://www.w3schools.com/python/default.asp)) and comparing them side by side with the PowerShell equivalent, to help cement my knowledge as I learn Python.

> The examples given below have been tested as working in PowerShell 7.4 and Python 3.13.

<style type="text/css">
  td { vertical-align: top; }
</style>

<table>

<tr><td colspan="3"><div markdown="1">
### Getting started

- The latest version of PowerShell is cross-platform and can be [installed on Windows, MacOS and Linux](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell?view=powershell-7.5).
- Python can also be [installed on a variety of platforms including Windows, MacOS and Linux](https://www.python.org/downloads/).
- Indentation is important in Python and is used to associate certain blocks of code, similar to how curly braces are used in PowerShell. The number of spaces you use is up to you, but must be consistent for each indented block.

</div></td></tr>
<tr width="100%"><th width="20%">Concept</th><th width="40%">PowerShell</th><th width="40%">Python</th></tr>

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
Write-Host "Hello, world!" #This too

<# This is a
multiline comment #>
```

</div>
</td>
<td>
<div markdown="1">

```python
#This is a comment
print("Hello, World!") #This too

# This is a
# multiline comment
```

Unofficially, you can also use `'''` for multiline comments, which are ignored unless used as [docstrings](https://www.geeksforgeeks.org/python-docstrings/).

</div>
</td>
</tr>

<tr><td colspan="3"><div markdown="1">
### Variables

- In both languages, a variable is created the moment you first assign a value to it.
- In both languages, variables are scoped based on where they are declared: [PowerShell](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_scopes?view=powershell-7.5) \| [Python](https://www.w3schools.com/python/python_scope.asp).
- Variables do not need to be declared with a specific type, and can change type after being set. You can however use casting to specify the data type.
</div></td></tr>
<tr width="100%"><th width="20%">Concept</th><th width="40%">PowerShell</th><th width="40%">Python</th></tr>

<tr>
<td>Creating variables</td>
<td>
<div markdown="1">

```powershell
$myVariable = 5
$text = 'sometext'
```

PowerShell variable names are case-insensitive, `$text` is the same as `$Text`.

</div>
</td>
<td>
<div markdown="1">

```python
myVariable = 5
text = 'sometext'
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
$x = [string]3   # '3'
$y = [int]3
$z = [double]3.14
```
`float` in PowerShell maps to `System.Single` which is a 32 bit integer. `double` is used in the example above to be equivalent to the Python `float` which uses a 64 bit integer.

</div>
</td>
<td>
<div markdown="1">

```python
x = str(3)   # '3'
y = int(3)
z = float(3.14)
```

`float` in Python is a 64 bit integer.

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

<tr><td colspan="3"><div markdown="1">
### Data Types

- Lists and tuples are standard Python data types that store values in a sequence. Sets are another standard Python data type that also store values. The major difference is that sets, unlike lists or tuples, cannot have multiple occurrences of the same element and store unordered values.

</div></td></tr>
<tr width="100%"><th width="20%">Concept</th><th width="40%">PowerShell</th><th width="40%">Python</th></tr>

<tr>
<td>Built-in Data Types</td>
<td>
<div markdown="1">

```powershell
$x = "some string"             #string
$x = 20                        #int
$x = 20.5                      #double
$x = "ben","max","kim"         #array

$x = 0..5                      #range
$x = @{name = "ben"; age = 36} #hashtable

$x = $true                     #bool
```

</div>
</td>
<td>
<div markdown="1">

```python
x = "some string"                #str
x = 20                           #int
x = 20.5                         #float
x = ["ben","max","kim"]          #list
x = ("ben","max","kim")          #tuple
x = range(6)                     #range
x = {"name" : "ben", "age" : 36} #dict
x = {"ben", "max", "kim"}        #set
x = True                         #bool
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

<tr><td colspan="3"><div markdown="1">
### Strings

- In both PowerShell and Python, strings can be specified via single or double quotes.
- Both PowerShell and Python have a number of built-in methods you can use on strings, but they differ. For a full list of methods see these pages: [PowerShell](http://xahlee.info/powershell/powershell_string_methods.html) \| [Python](https://www.w3schools.com/python/python_ref_string.asp)

</div></td></tr>
<tr width="100%"><th width="20%">Concept</th><th width="40%">PowerShell</th><th width="40%">Python</th></tr>

<tr>
<td>Quotation marks</td>
<td>
<div markdown="1">

```powershell
$str1 = 'this is a string'
$str2 = "this is also a string"
```
In PowerShell you can interpolate variables inside a double quoted string:

```powershell
$name = 'Mark'
Write-Host "my name is $name"
Write-Host 'my name is ' + $name
```
```plaintext
my name is Mark
my name is Mark
```

</div>
</td>
<td>
<div markdown="1">

```python
str1 = 'this is a string'
str2 = "this is also a string"
```

In Python you can interpolate variables in any string. The below uses the [f-string method](https://www.geeksforgeeks.org/formatted-string-literals-f-strings-python/):

```python
name = 'Mark'
print(f"my name is {name}")
print(f'my name is {name}')
```
```plaintext
my name is Mark
my name is Mark
```

</div>
</td>
</tr>

<tr>
<td>Multiline strings</td>
<td>
<div markdown="1">

```powershell
$str1 = @"
This is
a multiline
string
"@

$str2 = @'
This is also
a multiline
string
'@
```

</div>
</td>
<td>
<div markdown="1">

```python
str1 = """This is
a multiline
string"""

str2 = '''This is also
a multiline
string'''
```

</div>
</td>
</tr>

<tr>
<td>String length</td>
<td>
<div markdown="1">

```powershell
"This is a string".length
```
```plaintext
16
```

</div>
</td>
<td>
<div markdown="1">

```python
print(len("This is a string"))
```
```plaintext
16
```

</div>
</td>
</tr>

<tr>
<td>String methods</td>
<td>
<div markdown="1">

```powershell
$str = 'heLLo'

$str.padleft(8)           # --> '   heLLo'
$str.padright(8)          # --> 'heLLo   '
$str.tolower()            # --> 'hello'
$str.toupper()            # --> 'HELLO'
$str.replace('LL','YY')   # --> 'heYYo'
```

</div>
</td>
<td>
<div markdown="1">

```python
str = 'heLLo'

str.rjust(8)             # --> '   heLLo'
str.ljust(8)             # --> 'heLLo   '
str.lower()              # --> 'hello'
str.upper()              # --> 'hello'
str.replace('LL','YY')   # --> 'heYYo'

str.capitalize()         # --> 'HeLLo'
str.center(11)           # --> '   heLLo   '
```

</div>
</td>
</tr>

<tr><td colspan="3"><div markdown="1">
### Operators

- For a full list of operators see these pages: [PowerShell](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_operators?view=powershell-7.5) \| [Python](https://www.w3schools.com/python/python_operators.asp).

</div></td></tr>
<tr width="100%"><th width="20%">Concept</th><th width="40%">PowerShell</th><th width="40%">Python</th></tr>

<tr>
<td>Arithmetic</td>
<td>
<div markdown="1">

```powershell
$a + $b                  # Addition
$a - $b                  # Subtraction
$a * $b                  # Multiplication
$a / $b                  # Division
$a % $b                  # Modulus
[Math]::Pow($a, $b)      # Exponentiation
[math]::floor($a / $b)   # Floor division
```

</div>
</td>
<td>
<div markdown="1">

```python
a + b    # Addition
a - b    # Subtraction
a * b    # Multiplication
a / b    # Division
a % b    # Modulus
a ** b   # Exponentiation
a // b   # Floor division
```

</div>
</td>
</tr>

<tr>
<td>Assignment</td>
<td>
<div markdown="1">

```powershell
$x += 3   # --> x = x + 3 (add)
$x -= 3   # --> x = x - 3 (subtract)
$x *= 3   # --> x = x * 3 (multiply)
$x /= 3   # --> x = x / 3 (divide)
$x %= 3   # --> x = x % 3 (modulus)
```

</div>
</td>
<td>
<div markdown="1">

```python
$x += 3    # --> x = x + 3 (add)
$x -= 3    # --> x = x - 3 (subtract)
$x *= 3    # --> x = x * 3 (multiply)
$x /= 3    # --> x = x / 3 (divide)
$x %= 3    # --> x = x % 3 (modulus)
$x **= 3   # --> x = x ** 3 (exponentiation)
$x //= 3   # --> x = x // 3 (floor divide)
```

</div>
</td>
</tr>

<tr>
<td>Comparison</td>
<td>
<div markdown="1">

```powershell
$a -eq $b   # Equals
$a -ne $b   # Not equals
$a -lt $b   # Less than
$a -le $b   # Less than or equal to
$a -gt $b   # Greater than
$a -ge $b   # Greater than or equal to
```

</div>
</td>
<td>
<div markdown="1">

```python
a == b   # Equals
a != b   # Not equals
a < b    # Less than
a <= b   # Less than or equal to
a > b    # Greater than
a >= b   # Greater than or equal to
```

</div>
</td>
</tr>

<tr>
<td>Escape characters</td>
<td>
<div markdown="1">

```powershell
"This is `"how`" to escape" # Escape character
"This is `n on two lines"   # Insert new line
"`tThis has a tab space"    # Insert tab space
```

</div>
</td>
<td>
<div markdown="1">

```python
"This is \"how\" to escape" # Escape character
"This is \n on two lines"   # Insert new line
"\tThis has a tab space"    # Insert tab space
```

</div>
</td>
</tr>

<tr>
<td>Logical</td>
<td>
<div markdown="1">

```powershell
($a -gt 5) -and ($b -lt 10)
($a -gt 5) -or ($b -lt 10)
-not ($a -gt 5 -and $b -lt 10)
```

</div>
</td>
<td>
<div markdown="1">

```python
(a > 5) -and (b < 10)
(a > 5) -or (b < 10)
not(a > 5 and b < 10)
```

</div>
</td>
</tr>

<tr>
<td>Membership</td>
<td>
<div markdown="1">

```powershell
$a = 1,2,3
3 -in $a  # True
4 -in $a  # False
```

</div>
</td>
<td>
<div markdown="1">

```python
a = (1,2,3)
3 in a  # True
4 in a  # False
```

</div>
</td>
</tr>

<tr><td colspan="3"><div markdown="1">
### Conditional statements

- Python uses indentation to define scope in it's code. PowerShell uses curly-brackets.

</div></td></tr>
<tr width="100%"><th width="20%">Concept</th><th width="40%">PowerShell</th><th width="40%">Python</th></tr>

<tr>
<td>if / else-if / else</td>
<td>
<div markdown="1">

```powershell
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

<tr>
<td>switch / match</td>
<td>
<div markdown="1">

```powershell
switch ($day) {
    1..5 {
        Write-Host "weekday"
    }
    6..7 {
        Write-Host "weekend"
    }
    default {
        throw "day out of range"
    }
}
```

</div>
</td>
<td>
<div markdown="1">

```python
match day:
  case 1 | 2 | 3 | 4 | 5:
    print("weekday")

  case 6 | 7:
    print("weekend")

  case _:
    raise Exception("day out of range")

```

</div>
</td>
</tr>


<tr><td colspan="3"><div markdown="1">
### Iteration

- Python doesn't have a `do`..`until` loop, but you can achieve the same result by negating the condition of a `while` loop.

</div></td></tr>
<tr width="100%"><th width="20%">Concept</th><th width="40%">PowerShell</th><th width="40%">Python</th></tr>

<tr>
<td>While Loops</td>
<td>
<div markdown="1">

```powershell
$i = 1

while ($i -lt 5) {
  $i += 1
}
```

</div>
</td>
<td>
<div markdown="1">

```python
i = 1

while i < 5:
  i += 1

```

</div>
</td>
</tr>

<tr>
<td>For Loops</td>
<td>
<div markdown="1">

Iterate over a collection with a `foreach` loop:

```powershell
$fruits = "apple", "banana", "cherry"

foreach ($fruit in $fruits) {
  Write-Host $fruit
}
```

Iterate a set number of times at a custom increment:

```powershell
for ($i = 0; $i -le 20; $i += 2){
    Write-Host $i
}
```

</div>
</td>
<td>
<div markdown="1">

Iterate over a list with a `for` loop:

```python
fruits = ["apple", "banana", "cherry"]

for fruit in fruits:
  print(fruit)

```

Iterate a set number of times at a custom increment:

```python
for i in range(0, 21, 2):
    print(i)

```

</div>
</td>
</tr>

</table>
