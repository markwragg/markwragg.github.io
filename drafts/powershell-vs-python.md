---
title: A turn to the dark side — Learning Python through PowerShell
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2025/darth-vader.jpg"
  teaser: "/content/images/2025/darth-vader.jpg"
date: "2025-05-15 09:00:00"
tags:
  - powershell
  - python
---

<style type="text/css">
  td { vertical-align: top; }

  #top_left_col {
    float:left;
    width:72%;
    padding-right: 30px;
  }
  #top_right_col {
    float:right;
    width:28%;
  }
</style>

<div id="top_left_col" markdown="1">

PowerShell and Python are powerful programming languages with many similarities. While PowerShell is technically a shell scripting language (like Bash), functionally it has a lot more in common with Python, and can be used to generate scripts of equal complexity.

As someone who (the force) is strong with PowerShell, I'm finding it useful (as I am increasingly tempted by the dark side) to reference the concepts of Python against their PowerShell equivalents.

[Adam Driscoll did this previously in 2020 and his page is incredibly helpful](https://blog.ironmansoftware.com/powershell-vs-python/). Below I've followed a similar approach, by using many of the concepts covered by the [W3Schools Python tutorials](https://www.w3schools.com/python/default.asp)) and comparing them side by side with their PowerShell equivalents, to help cement my knowledge as I learn Python.

</div>

<div id="top_right_col" markdown="1">
{% include toc icon="code" title="Contents" %}
</div>

<table>
<tr><td colspan="3"><div markdown="1">

> The examples below have been tested in PowerShell version 7.4.7 and Python version 3.13.3.

I've tried to keep the example code as narrow as possible, but if you're visiting this site via a mobile you may need to use landscape mode.

If you find any errors in the code examples (or if you have any suggestions for new ones) please let me know.

### Get started

- The latest version of PowerShell is cross-platform and can be [installed on Windows, MacOS and Linux](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell?view=powershell-7.5).
- Python can also be [installed on a variety of platforms including Windows, MacOS and Linux](https://www.python.org/downloads/).
- Indentation is important in Python and is used to associate certain blocks of code, similar to how curly braces are used in PowerShell. The number of spaces you use is up to you, but must be consistent for each indented block.

</div></td></tr>
<tr><th width="20%">Concept</th><th width="40%">PowerShell</th><th width="40%">Python</th></tr>

<tr>
<td>Check installed version</td>
<td>
<div markdown="1">

```powershell
($PSVersionTable).PSVersion
```

```plaintext
7.4.7
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

`float` in PowerShell maps to `System.Single` which is a 32 bit floating-point number. `double` is used in the example above to be equivalent to the Python `float` which uses a 64 bit integer.

</div>
</td>
<td>
<div markdown="1">

```python
x = str(3)   # '3'
y = int(3)
z = float(3.14)
```

`float` in Python is a 64 bit floating-point number.

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

- In both languages, strings can be specified via single or double quotes.
- Both languages have a number of built-in methods you can use on strings, but they differ. For a full list of methods see these pages: [PowerShell](http://xahlee.info/powershell/powershell_string_methods.html) \| [Python](https://www.w3schools.com/python/python_ref_string.asp)

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
str1 = """
This is
a multiline
string
"""

str2 = '''
This is also
a multiline
string
'''
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
```

</div>
</td>
</tr>

<tr><td colspan="3"><div markdown="1">
### Arrays

- Python does not have built-in support for arrays, but Python Lists can be used instead, as demonstrated in the below examples. You can get [arrays via the NumPy module](https://www.w3schools.com/python/numpy/numpy_creating_arrays.asp).
- PowerShell has arrays, but once defined they are a fixed length. To append a new item PowerShell recreates the entire array with the new element. You can alternatively use `[System.Collections.ArrayList]` which is a dynamic array that behaves more like Python Lists.

</div></td></tr>
<tr width="100%"><th width="20%">Concept</th><th width="40%">PowerShell</th><th width="40%">Python</th></tr>

<tr>
<td>Create an array</td>
<td>
<div markdown="1">

```powershell
$basket = @("apple","pear","plum")
$basket

```
```plaintext
apple
pear
plum
```

</div>
</td>
<td>
<div markdown="1">

```python
basket = ["apple","pear","plum"]
for fruit in basket:
  print(fruit)
```
```plaintext
apple
pear
plum
```

</div>
</td>
</tr>

<tr>
<td>Array length (number of elements)</td>
<td>
<div markdown="1">

```powershell
$basket = @("apple","pear","plum")
$basket.count
```
```plaintext
3
```

</div>
</td>
<td>
<div markdown="1">

```python
basket = ["apple","pear","plum"]
print(len(basket))
```
```plaintext
3
```

<tr>
<td>Add an item to an array</td>
<td>
<div markdown="1">

```powershell
$basket = @("apple","pear","plum")
$basket += "banana"
```

</div>
</td>
<td>
<div markdown="1">

```python
basket = ["apple","pear","plum"]
basket.append("banana")
```

</div>
</td>
</tr>

<tr>
<td>Remove an item from an array</td>
<td>
<div markdown="1">

```powershell
$basket = @("apple","pear","plum")
$basket = $basket | where { $_ -ne "banana" }
```
Alternatively you could define a `[System.Collections.ArrayList]` object which has `add` and `remove` methods.

</div>
</td>
<td>
<div markdown="1">

```python
basket = ["apple","pear","plum"]
basket.remove("banana")
```
This will only remove the first occurrence of the specified value.

</div>
</td>
</tr>

<tr>
<td>Sort an array</td>
<td>
<div markdown="1">

```powershell
$basket = @("apple","pear","plum","banana")
$basket | Sort-Object
$basket

```
```plaintext
apple
banana
pear
plum
```
Alternatively you could define a `[System.Collections.ArrayList]` object which has a `sort` method.

</div>
</td>
<td>
<div markdown="1">

```python
basket = ["apple","pear","plum","banana"]
basket.sort()
for fruit in basket:
  print(fruit)
```
```plaintext
apple
banana
pear
plum
```
Sorts the list ascending by default. Use `sort(reverse=True)` to specify descending.

</div>
</td>
</tr>

<tr><td colspan="3"><div markdown="1">
### Dictionaries

- Dictionaries are a collection of key value pairs. In PowerShell a dictionary is referred to as a [hashtable](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_hash_tables?view=powershell-7.5).
- In both languages, dictionaries (hashtables) are changeable: you can modify/add/remove items after creation. Dictionary keys must be unique.

</div></td></tr>
<tr width="100%"><th width="20%">Concept</th><th width="40%">PowerShell</th><th width="40%">Python</th></tr>

<tr>
<td>Creation</td>
<td>
<div markdown="1">

```powershell
$car = @{
  brand = "Audi"
  model = "Q7"
  year = 2019
}
```

You can specify an ordered hashtable by adding the `[ordered]` type accelerator.

</div>
</td>
<td>
<div markdown="1">

```python
car = {
  "brand": "Audi",
  "model": "Q7",
  "year": 2019
}
```

As of Python 3.7, dictionaries are ordered by default. Prior to 3.7 they are unordered.

</div>
</td>
</tr>

<tr>
<td>Return an item</td>
<td>
<div markdown="1">

```powershell
$car["brand"]
```

```plaintext
Audi
```

</div>
</td>
<td>
<div markdown="1">

```python
print(car["brand"])
```

```plaintext
Audi
```

</div>
</td>
</tr>

<tr>
<td>List keys</td>
<td>
<div markdown="1">

```powershell
$car.keys
```

```plaintext
year
brand
model
```

</div>
</td>
<td>
<div markdown="1">

```python
print(car.keys())
```

```plaintext
dict_keys(['brand', 'model', 'year'])
```

The list of the values is a view of the dictionary. Any changes to the dictionary will be reflected in the list.

</div>
</td>
</tr>

<tr>
<td>List items</td>
<td>
<div markdown="1">

```powershell
$car.values
```

```plaintext
2019
Audi
Q7
```

</div>
</td>
<td>
<div markdown="1">

```python
print(car.items())
```

```plaintext
dict_items([('brand', 'Audi'),
  ('model', 'Q7'),
  ('year', 2019)])
```

The list of the values is a view of the dictionary. Any changes to the dictionary will be reflected in the list.

</div>
</td>
</tr>

<tr>
<td>Check if a key exists</td>
<td>
<div markdown="1">

```powershell
if ('model' -in $car.Keys) {
  $true
}
```

</div>
</td>
<td>
<div markdown="1">

```python
if "model" in car:
  print(True)

```

</div>
</td>
</tr>

<tr>
<td>Add or update an item</td>
<td>
<div markdown="1">

```powershell
$car["color"] = "Blue"
```

</div>
</td>
<td>
<div markdown="1">

```python
car["color"] = "Blue"
```

</div>
</td>
</tr>

<tr>
<td>Remove an item</td>
<td>
<div markdown="1">

```powershell
$car.remove("color")
```

</div>
</td>
<td>
<div markdown="1">

```python
car.pop("color")
```

</div>
</td>
</tr>

<tr>
<td>Remove all items</td>
<td>
<div markdown="1">

```powershell
$car.clear()
```

</div>
</td>
<td>
<div markdown="1">

```python
car.clear()
```

</div>
</td>
</tr>

<tr>
<td>Dictionary length</td>
<td>
<div markdown="1">

```powershell
$car.count
```

```plaintext
3
```

</div>
</td>
<td>
<div markdown="1">

```python
print(len(car))
```

```plaintext
3
```

</div>
</td>
</tr>

<tr><td colspan="3"><div markdown="1">
### Dates

- To work with dates in Python you need to import the built-in `datetime` module.

</div></td></tr>
<tr width="100%"><th width="20%">Concept</th><th width="40%">PowerShell</th><th width="40%">Python</th></tr>

<tr>
<td>Return the current date/time</td>
<td>
<div markdown="1">

```powershell
$now = Get-Date


$now
$now.day
$now.month
$now.year
```

```plaintext
17 May 2025 13:27:16
17
5
2025
```

</div>
</td>
<td>
<div markdown="1">

```python
import datetime
now = datetime.datetime.now()

print(now)
print(now.day)
print(now.month)
print(now.year)
```

```plaintext
2025-05-17 12:28:25.903551
17
5
2025
```

</div>
</td>
</tr>

<tr>
<td>Create a date object</td>
<td>
<div markdown="1">

```powershell
$x = Get-Date -Day 17 -Month 5 -Year 2025

```

Time parameters can also be specified, their default is the current time.

</div>
</td>
<td>
<div markdown="1">

```python
import datetime
x = datetime.datetime(2025, 5, 17)
```

Time parameters can also be specified, their default is 0.

</div>
</td>
</tr>

<tr>
<td>Formatting dates</td>
<td>
<div markdown="1">

```powershell
$x = Get-Date -Day 17 -Month 5 -Year 2025


Get-Date $x -UFormat '%a' # --> Sat
Get-Date $x -UFormat '%A' # --> Saturday
Get-Date $x -UFormat '%w' # --> 6
Get-Date $x -UFormat '%d' # --> 17
Get-Date $x -UFormat '%b' # --> May
Get-Date $x -UFormat '%m' # --> 05
Get-Date $x -UFormat '%y' # --> 25
Get-Date $x -UFormat '%Y' # --> 2025
```

See here for the full list of supported [Python datetime formatting codes](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/get-date?view=powershell-7.5#notes).

</div>
</td>
<td>
<div markdown="1">

```python
import datetime
x = datetime.datetime(2025, 5, 17)

x.strftime("%a") # --> Sat
x.strftime("%A") # --> Saturday
x.strftime("%w") # --> 6
x.strftime("%d") # --> 17
x.strftime("%b") # --> May
x.strftime("%m") # --> 05
x.strftime("%y") # --> 25
x.strftime("%Y") # --> 2025
```

See here for the full list of supported [PowerShell datetime formatting codes](https://docs.python.org/3/library/datetime.html#format-codes).

</div>
</td>
</tr>

<tr><td colspan="3"><div markdown="1">
### JSON

- To work with JSON in Python you need to import the built-in `json` module.

</div></td></tr>
<tr width="100%"><th width="20%">Concept</th><th width="40%">PowerShell</th><th width="40%">Python</th></tr>

<tr>
<td>Convert from JSON</td>
<td>
<div markdown="1">

```powershell

$x = '{"name":"Bob","age":30,"city":"Bath"}'
$y = $x | ConvertFrom-Json
$y.name
$y.age
$y.city
```

PowerShell converts JSON to a PSCustomObject.

```plaintext
Bob
30
Bath
```

</div>
</td>
<td>
<div markdown="1">

```python
import json
x = '{ "name":"Bob","age":30,"city":"Bath"}'
y = json.loads(x)
print(y["name"])
print(y["age"])
print(y["city"])
```

Python converts JSON to a dictionary object.

```plaintext
Bob
30
Bath
```

</div>
</td>
</tr>

<tr>
<td>Convert to JSON</td>
<td>
<div markdown="1">

```powershell

$x = @{
  name = "Bob"
  age  = 30
  city = "Bath"
}

$y = $x | ConvertTo-Json
$y
```

The ConvertTo-Json cmdlet converts any .NET object to JSON. The properties become the name / values, and the methods are removed.

```plaintext
{
  "age": 30,
  "name": "Bob",
  "city": "Bath"
}
```

</div>
</td>
<td>
<div markdown="1">

```python
import json
x = {
  "name": "Bob",
  "age": 30,
  "city": "Bath"
}

y = json.dumps(x)
print(y)
```

Python objects are converted into the JSON equivalent: `dict` --> object, `list`, `tuple` --> array, `str` --> string, `int`, `float` --> number, `True`/`False` --> true/false, `None` --> null.

```plaintext
{"name":"John","age": 30,"city":"New York"}
```

You can further customise the indentation, separators and sorting as follows (these can also be combined):

```python
json.dumps(x, indent=4)
json.dumps(x, separators=(". ", " = "))
json.dumps(x, sort_keys=True)
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
x += 3    # --> x = x + 3 (add)
x -= 3    # --> x = x - 3 (subtract)
x *= 3    # --> x = x * 3 (multiply)
x /= 3    # --> x = x / 3 (divide)
x %= 3    # --> x = x % 3 (modulus)
x **= 3   # --> x = x ** 3 (exponentiation)
x //= 3   # --> x = x // 3 (floor divide)
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
"This is `"how`" to escape" # Escape next
"This is `n on two lines"   # new line
"`tThis has a tab space"    # tab space
```

</div>
</td>
<td>
<div markdown="1">

```python
"This is \"how\" to escape" # Escape next
"This is \n on two lines"   # new line
"\tThis has a tab space"    # tab space
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
(a > 5) and (b < 10)
(a > 5) or (b < 10)
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
3 -in $a       # True
4 -in $a       # False
3 -notin $a    # False
```

PowerShell also has the `-contains` operator.

</div>
</td>
<td>
<div markdown="1">

```python
a = (1,2,3)
3 in a     # True
4 in a     # False
3 not in a # False
```

</div>
</td>
</tr>

<tr><td colspan="3"><div markdown="1">
### Conditionals

- Python uses indentation to define the code that belongs to a conditional statement. PowerShell uses curly braces.

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

<tr><td colspan="3"><div markdown="1">
### Functions

- Python uses indentation to define the code that belongs to a function. PowerShell uses curly braces.
- In PowerShell a `param()` block can also be used to define function input parameters (arguments), with additional validation.

</div></td></tr>
<tr width="100%"><th width="20%">Concept</th><th width="40%">PowerShell</th><th width="40%">Python</th></tr>

<tr>
<td>Defining and calling a function</td>
<td>
<div markdown="1">

```powershell
function sayHello {
  Write-Host "Hello"
}

sayHello
```

```plaintext
Hello
```

</div>
</td>
<td>
<div markdown="1">

```python
def sayHello():
  print("Hello")


sayHello()
```

```plaintext
Hello
```

</div>
</td>
</tr>

<tr>
<td>Function arguments / parameters</td>
<td>
<div markdown="1">

```powershell
function sayHello($name) {
  Write-Host "Hello $name"
}

sayHello "Sarah"
```

```plaintext
Hello Sarah
```

```powershell
function sayHello($fname, $lname) {
  Write-Host "Hello $fname $lname"
}

sayHello "Bob" "Bilby"
```

```plaintext
Hello Bob Bilby
```

Parameters can be invoked by name:

```python
sayHello -fname "Bob" -lname "Bilby"
```

</div>
</td>
<td>
<div markdown="1">

```python
def sayHello(name):
  print(f"Hello {name}")


sayHello("Sarah")
```

```plaintext
Hello Sarah
```

```python
def sayHello(fname,lname):
  print(f"Hello {fname} {lname}")


sayHello("Bob", "Bilby")
```

```plaintext
Hello Bob Bilby
```

Parameters can be invoked by name:

```python
sayHello(fname = "Bob", lname = "Bilby")
```

</div>
</td>
</tr>

<tr>
<td>Argument / parameter default values</td>
<td>
<div markdown="1">

```powershell
function sayHello($name, $co = "BobCo") {
  Write-Host "Hello $name from $co"
}

sayHello "Jeff"
```

```plaintext
Hello Jeff from BobCo
```

</div>
</td>
<td>
<div markdown="1">

```python
def sayHello(name,co = "BobCo"):
  print(f"Hello {name} from {co}")


sayHello("Jeff")
```

```plaintext
Hello Jeff from BobCo
```

</div>
</td>
</tr>

<tr>
<td>Arbitrary arguments *args</td>
<td>
<div markdown="1">

```powershell
function sayHello() {
  foreach ($name in $args) {
    Write-Host "Hello $name"
  }
}

sayHello "Jeff" "Bob" "Sarah"
```

```plaintext
Hello Jeff
Hello Bob
Hello Sarah
```

</div>
</td>
<td>
<div markdown="1">

```python
def sayHello(*names):
  for name in names:
    print(f"Hello {name}")



sayHello("Jeff", "Bob", "Sarah")
```

```plaintext
Hello Jeff
Hello Bob
Hello Sarah
```

</div>
</td>
</tr>

<tr>
<td>Arbitrary keyword arguments **kwargs</td>
<td>
<div markdown="1">

```powershell
function personDetails($person) {
  foreach ($p in $person.getenumerator()) {
    Write-Host "$($p.key)`: $($p.value)"
  }
}

personDetails @{fn = "Bob"; ln = "Bilby"}
```

</div>
</td>
<td>
<div markdown="1">

```python
def personDetails(**person):
  for key in person:
    print(f"{key}: {person[key]}")


personDetails(fn = "Bob", ln = "Bilby")
```

</div>
</td>
</tr>

<tr>
<td>Return values</td>
<td>
<div markdown="1">

```powershell
function multiplyBy5 ($x) {
  return ($x * 5)
}

multiplyBy5 5
```

```plaintext
25
```

</div>
</td>
<td>
<div markdown="1">

```python
def multiplyBy5 (x):
    return (x * 5)


print(multiplyBy5(5))
```

```plaintext
25
```

</div>
</td>
</tr>

<tr>
<td>Empty function</td>
<td>
<div markdown="1">

```powershell
function emptyFunction {}

```

</div>
</td>
<td>
<div markdown="1">

```python
def emptyFunction:
  pass
```

</div>
</td>
</tr>

<tr><td colspan="3"><div markdown="1">
### Classes

- Both languages are object oriented. Almost everything is an object with properties and methods.
- A Class is a way to define a custom object's structure.

</div></td></tr>
<tr width="100%"><th width="20%">Concept</th><th width="40%">PowerShell</th><th width="40%">Python</th></tr>


<tr>
<td>Class with a fixed property</td>
<td>
<div markdown="1">

```powershell
Class Customer {
  $Bank = "GlobalBank"
}

# Example usage:
$c = [Customer]::new()
$c.Bank
```

```plaintext
GlobalBank
```

</div>
</td>
<td>
<div markdown="1">

```python
class Customer:
  Bank = "GlobalBank"

# Example usage:
c = Customer()
print(c.Bank)
```

```plaintext
GlobalBank
```

</div>
</td>
</tr>

<tr>
<td>Class with assignable properties</td>
<td>
<div markdown="1">

```powershell
Class Customer {
  [string]$name
  [decimal]$balance

  $Bank = "GlobalBank"

  Customer($name, $balance = 0) {
    $this.name = $name
    $this.balance = $balance
  }
}

# Example usage:
$c = [Customer]::new("Alice",100)
$c.name
$c.balance
```

```plaintext
Alice
100
```

</div>
</td>
<td>
<div markdown="1">

```python
class Customer:
  bank = "GlobalBank"

  def __init__(self, name, balance=0):
    self.name = name
    self.balance = balance

# Example usage:
c = Customer("Alice",100)
print(c.name)
print(c.balance)
```

```plaintext
Alice
100
```

</div>
</td>
</tr>

<tr>
<td>Class with properties and methods</td>
<td>
<div markdown="1">

```powershell
Class Customer {
  [string]$name
  [decimal]$balance

  $Bank = "GlobalBank"

  Customer($name, $balance = 0) {
    $this.name = $name
    $this.balance = $balance
  }

  [void]Deposit($amount) {
    if ($amount -le 0) {
      throw "Negative deposit."
    }
    $this.balance += $amount
  }

  [void]Withdraw($amount) {
    if ($amount -gt $this.balance) {
      throw "Insufficient funds."
    }
    $this.balance -= $amount
  }

  [string]ToString() {
    $fbalance = "£{0:N2}" -f $this.balance
    return "$($this.name): $fbalance"
  }
}

# Example usage:
$c = [Customer]::new("Alice", 100)
$c.Deposit(50)
$c.Withdraw(30)
$c.ToString()
```

```plaintext
Alice: £120.00
```

</div>
</td>
<td>
<div markdown="1">

```python
class Customer:

  bank = "GlobalBank"

  def __init__(self, name, balance=0):
    self.name = name
    self.balance = balance

  def deposit(self, amount):
    if amount <= 0:
        raise ValueError("Negative deposit.")

    self.balance += amount

  def withdraw(self, amount):
    if amount > self.balance:
        raise ValueError("Insufficient funds.")

    self.balance -= amount

  def __str__(self):
    fbalance = f"£{self.balance:.2f}"
    return f"{self.name}: {fbalance}"


# Example usage:
c = Customer("Alice", 100)
c.deposit(50)
c.withdraw(30)
print(c)
```

```plaintext
Alice: £120.00
```

</div>
</td>
</tr>

<tr><td colspan="3"><div markdown="1">
### Modules

- In both languages, a module is simply a collection of functions that you want to include in another script or application.
- In Python modules are discovered via the [module search path](https://docs.python.org/3/tutorial/modules.html#the-module-search-path):

  > When a module named `spam` is imported, the interpreter first searches for a built-in module with that name. These module names are listed in `sys.builtin_module_names`. If not found, it then searches for a file named `spam.py` in a list of directories given by the variable `sys.path`.

</div></td></tr>
<tr width="100%"><th width="20%">Concept</th><th width="40%">PowerShell</th><th width="40%">Python</th></tr>

<tr>
<td>Use a module</td>
<td>
<div markdown="1">

```powershell
# Save this in a file called MyModule.ps1
function greeting($name):
  Write-Host "Hello, $name"

# In your main file
Import-Module MyModule.ps1

greeting "Mark"
```

</div>
</td>
<td>
<div markdown="1">

```python
# Save this in a file called MyModule.py
def greeting(name):
  print("Hello, {name}")

# In your main file
import MyModule

MyModule.greeting("Mark")
```

</div>
</td>
</tr>

<tr>
<td>List the functions in a module</td>
<td>
<div markdown="1">

```powershell
Import-Module MyModule.ps1

(Get-Command -Module MyModule).Name

```

</div>
</td>
<td>
<div markdown="1">

```python
import MyModule

for name in dir(MyModule):
  print(name)
```

</div>
</td>
</tr>

<tr>
<td>Import a specific function</td>
<td>
<div markdown="1">

```powershell
Import-Module MyModule -Function "greeting"
```

</div>
</td>
<td>
<div markdown="1">

```python
from MyModule import greeting
```

</div>
</td>
</tr>

<tr>
<td>Install a module</td>
<td>
<div markdown="1">

```powershell
Install-Module SomeModule
```

</div>
</td>
<td>
<div markdown="1">

```python
pip install SomeModule
```

</div>
</td>
</tr>

<tr>
<td>Uninstall a module</td>
<td>
<div markdown="1">

```powershell
Uninstall-Module SomeModule
```

</div>
</td>
<td>
<div markdown="1">

```python
pip uninstall SomeModule
```

</div>
</td>
</tr>

<tr>
<td>List installed modules</td>
<td>
<div markdown="1">

```powershell
# List modules installed via PowerShellGet
Get-InstalledModule
```

</div>
</td>
<td>
<div markdown="1">

```python
# List modules installed via pip
pip list
```

</div>
</td>
</tr>

<tr><td colspan="3"><div markdown="1">
### Exceptions

- PowerShell and Python have similar concepts for handling excpetions, with `try..catch` and `try..except` being functionally similar. Both support catching multiple specific exception types and a `finally` block for performing post-exception clean up tasks, such as closing connections or removing temporary files.

</div></td></tr>
<tr width="100%"><th width="20%">Concept</th><th width="40%">PowerShell</th><th width="40%">Python</th></tr>

<tr>
<td>Raise an exception</td>
<td>
<div markdown="1">

```powershell
throw 'An exception occurred'
```

</div>
</td>
<td>
<div markdown="1">

```python
raise Exception("An exception occurred")
```

</div>
</td>
</tr>

<tr>
<td>Catch exceptions</td>
<td>
<div markdown="1">

```powershell
try {
  Write-Host $x -ErrorAction Stop
}
catch {
  Write-Error ("An exception occurred")
}
```

</div>
</td>
<td>
<div markdown="1">

```python
try:
  print(x)

except:
  print("An exception occurred")

```

</div>
</td>
</tr>

<tr>
<td>Catch multiple exceptions</td>
<td>
<div markdown="1">

```powershell
try {
  Get-Content .\file.txt -ErrorAction Stop
}
catch [System.IO.IOException] {
  Write-Error "A file error occurred"
}
catch {
  Write-Error "A different error occurred"
}
```

</div>
</td>
<td>
<div markdown="1">

```python
try:
  f = open('file.txt', 'w')

except OSError:
  print("A file error occurred")

except:
  print("A different error occurred")

```

</div>
</td>
</tr>

<tr>
<td>Execute code after an exception</td>
<td>
<div markdown="1">

```powershell
try {
  $f = [System.IO.File]::Open("file.txt")
}
catch {
  Write-Error "Failed to open"
}
finally {
  $f.close()
}
```

</div>
</td>
<td>
<div markdown="1">

```python
try:
  f = open('file.txt', 'w')

except:
  print("Failed to open")

finally:
  f.close()

```

</div>
</td>
</tr>