# CS50 2020 Week6: Python Problem Answers

<!--more-->
Click title to detail page

## [Hello](https://cs50.harvard.edu/x/2020/psets/6/hello)

```python
from cs50 import get_string

name = get_string("What's your name? ")

print(f"hello, {name}")
```
## [Mario](https://cs50.harvard.edu/x/2020/psets/6/mario/more/)

```python
from cs50 import get_int

height = get_int("Height: ")

while height <= 0 or height >= 9:
    height = get_int("Height: ")

for i in range(height):
    for k in range(2):
        if k == 0:
            if height != 1:
                print(" " * (height - i - 1), end="")
            print("#" * (i + 1), end="")
            print("  ", end="")
        else:
            print("#" * (i + 1))
```

## [Credit](https://cs50.harvard.edu/x/2020/psets/6/credit)

```python
from cs50 import get_string

number = get_string("Number: ")

digit = len(number)
chk1 = chk2 = 0
first_two = int(number[:2])

for i in range(len(number)):
    if digit % 2 == 0:
        if i % 2 == 0:
            tmp = int(number[i]) * 2
            chk2 += tmp // 10
            chk2 += tmp % 10
        else:
            chk1 += int(number[i])
    else:
        if i % 2 == 0:
            chk1 += int(number[i])
        else:
            tmp = int(number[i]) * 2
            chk2 += tmp // 10
            chk2 += tmp % 10

if (chk1 + chk2) % 10 != 0:
    print("INVALID")
else:
    if first_two == 34 or first_two == 37:
        if digit == 15:
            print("AMEX")
        else:
            print("INVALID")
    elif first_two >= 51 and first_two <= 55:
        if digit == 16:
            print("MASTERCARD")
        else:
            print("INVALID")
    elif first_two // 10 == 4:
        if digit == 16 or digit == 13:
            print("VISA")
        else:
            print("INVALID")
    else:
        print("INVALID")

```
## [Readability](https://cs50.harvard.edu/x/2020/psets/6/readability)

```python
from cs50 import get_string

input = get_string("Text: ")

letters = words = sentences = 0

for i in range(len(input)):
    if input[i].isalpha():
        letters += 1
    elif input[i].isspace():
        words += 1
    elif input[i] == "." or input[i] == "!" or input[i] == "?":
        sentences += 1

words += 1

index = int(round((0.0588 * (letters / words) * 100 - 0.296 * (sentences / words) * 100 - 15.8), 0))

if index <= 1:
    print("Before Grade 1")
elif index >= 2 and index <= 15:
    print(f"Grade {index}")
elif index >= 16:
    print("Grade 16+")
```

## [DNA](https://cs50.harvard.edu/x/2020/psets/6/dna)

```python
import sys
import csv

if len(sys.argv) != 3:
    print("Usage: python dna.py data.csv sequence.txt")

strs = {}

with open(sys.argv[1]) as database:
    rows = csv.DictReader(database)
    for row in rows:
        for key in row:
            if key != "name":
                if key in strs:
                    strs[key] = {**strs[key], row["name"]: int(row[key])}
                else:
                    strs[key] = {row["name"]: int(row[key])}

match = {}
match_find = False

with open(sys.argv[2]) as seq_file:
    seq = seq_file.readline()

    for key in strs:
        str_max = 0
        str_tmp = 0
        str_idx = 0
        str_start = False

        for i in range(len(seq)):
            if seq.startswith(key, i) and not str_start:
                str_tmp = 1
                str_idx = i
                str_start = True
            elif str_start:
                if i - str_idx < len(key):
                    continue
                elif seq.startswith(key, i):
                    str_tmp += 1
                    str_idx = i
                else:
                    str_start = False
                    if str_tmp > str_max:
                        str_max = str_tmp

        for people, value in strs[key].items():
            if str_max == value:
                if people in match:
                    match[people] += 1
                else:
                    match[people] = 1

for key in match:
    if match[key] == len(strs.keys()):
        print(key)
        match_find = True

if not match_find:
    print("No match")
```
