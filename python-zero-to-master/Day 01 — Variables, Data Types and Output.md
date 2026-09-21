# Python Day 1 — Variables, Data Types and Output

## Day 1 goal

By the end of Day 1, Veena should be able to:

- Display information using `print()`
- Store information in variables
- Understand the four basic data types
- Check a value’s type using `type()`
- Follow Python’s top-to-bottom execution
- Perform calculations using variables
- Update variable values
- Create clear variable names
- Read and correct common beginner errors

---

# 1. The vital 20%

These ideas provide most of today’s value:

```text
Value → stored in a variable
Variable → used in a calculation
Calculation → stored as a result
print() → displays the result
type() → identifies the data type
```

Example:

```python
price = 50
quantity = 3

total = price * quantity

print("Total =", total)
```

Output:

```text
Total = 150
```

---

# 2. The `print()` function

`print()` displays information on the screen.

```python
print("Welcome to Python!")
print(100)
print(10 + 20)
```

Output:

```text
Welcome to Python!
100
30
```

## Text requires quotation marks

Correct:

```python
print("Veena D")
```

Incorrect:

```python
print(Veena D)
```

Without quotation marks, Python thinks `Veena` and `D` are names in the program.

## Printing multiple values

```python
name = "Veena D"
age = 18

print("Name =", name)
print("Age =", age)
```

Output:

```text
Name = Veena D
Age = 18
```

A comma allows Python to print text and variables together.

---

# 3. What is a variable?

A variable is a meaningful name used to store a value.

Think of it as a labelled container:

```text
┌───────────────┐
│ name          │ → "Veena D"
├───────────────┤
│ age           │ → 18
├───────────────┤
│ course        │ → "CSE AI"
└───────────────┘
```

Python code:

```python
name = "Veena D"
age = 18
course = "CSE AI"
```

The `=` symbol means:

> Calculate or read the value on the right, and store it using the name on the left.

```python
age = 18
```

This means:

```text
Store 18 inside age.
```

It does not display anything.

To display the value:

```python
print(age)
```

---

# 4. How assignment works

Study this statement:

```python
total = 10 + 20
```

Python performs it in this order:

```text
1. Calculate 10 + 20
2. Produce 30
3. Store 30 in total
```

Memory becomes:

```text
total → 30
```

Then:

```python
print(total)
```

Output:

```text
30
```

---

# 5. Basic data types

A data type tells Python what kind of value it is handling.

## String — `str`

A string stores text.

```python
name = "Veena D"
city = "Dharmavaram"
course = "CSE AI"
```

Strings must be placed inside quotes.

---

## Integer — `int`

An integer stores a whole number.

```python
age = 18
marks = 90
quantity = 5
```

Integers do not use quotation marks.

---

## Float — `float`

A float stores a decimal number.

```python
average = 85.5
price = 49.99
temperature = 36.5
```

---

## Boolean — `bool`

A Boolean stores one of two values:

```python
is_student = True
has_passed = False
```

Python uses:

```python
True
False
```

Capitalisation matters.

These are incorrect:

```python
true
false
```

---

# 6. Number versus text

These values look similar but have different types:

```python
age_number = 18
age_text = "18"
```

Their types are:

```text
18   → int
"18" → str
```

This works:

```python
print(18 + 2)
```

Output:

```text
20
```

This joins text:

```python
print("18" + "2")
```

Output:

```text
182
```

The quotation marks change the meaning.

---

# 7. Checking a type with `type()`

Use `type()` to identify a value’s data type.

```python
name = "Veena D"
age = 18
average = 85.5
is_student = True

print(type(name))
print(type(age))
print(type(average))
print(type(is_student))
```

Output:

```text
<class 'str'>
<class 'int'>
<class 'float'>
<class 'bool'>
```

The important parts are:

```text
str
int
float
bool
```

---

# 8. Arithmetic operators

| Operator | Purpose | Example | Result |
|---|---|---|---:|
| `+` | Addition | `10 + 5` | `15` |
| `-` | Subtraction | `10 - 5` | `5` |
| `*` | Multiplication | `10 * 5` | `50` |
| `/` | Division | `10 / 5` | `2.0` |
| `%` | Remainder | `10 % 3` | `1` |
| `**` | Power | `2 ** 3` | `8` |
| `//` | Floor division | `10 // 3` | `3` |

For Day 1, focus mainly on:

```text
+  -  *  /
```

---

# 9. Calculations with variables

## Rectangle area

```python
length = 10
width = 4

area = length * width

print("Rectangle area =", area)
```

Output:

```text
Rectangle area = 40
```

## Student marks

```python
maths = 80
science = 90
social = 70

total = maths + science + social
average = total / 3

print("Total =", total)
print("Average =", average)
```

Output:

```text
Total = 240
Average = 80.0
```

## Shop bill

```python
price = 100
quantity = 3

final_bill = price * quantity

print("Final bill =", final_bill)
```

Output:

```text
Final bill = 300
```

---

# 10. Updating a variable

A variable can receive a new value.

```python
score = 50
print(score)

score = 90
print(score)
```

Output:

```text
50
90
```

After this statement:

```python
score = 90
```

the value currently stored in `score` is `90`.

## Updating using the old value

```python
score = 50
score = score + 10

print(score)
```

Execution:

```text
1. Read the old score: 50
2. Calculate 50 + 10
3. Store 60 back in score
```

Output:

```text
60
```

---

# 11. Variable-naming rules

## Valid names

```python
name = "Veena D"
age = 18
mark1 = 90
student_name = "Veena D"
total_marks = 270
```

## Invalid names

```python
# 1name = "Veena D"       Starts with a number
# student name = "Veena"  Contains a space
# student-name = "Veena"  Contains a hyphen
# class = "CSE AI"        Uses a Python keyword
```

## Recommended convention

Use lowercase words separated by underscores:

```python
student_name = "Veena D"
total_marks = 270
final_bill = 500
```

This style is called `snake_case`.

## Use meaningful names

Unclear:

```python
x = 100
y = 3
z = x * y
```

Clear:

```python
price = 100
quantity = 3
final_bill = price * quantity
```

The computer can run both versions, but the second is easier for people to understand.

---

# 12. Python is case-sensitive

These are different names:

```python
name
Name
NAME
```

Example:

```python
name = "Veena D"
print(Name)
```

This fails because only `name` was created.

Correct:

```python
name = "Veena D"
print(name)
```

Similarly:

```python
print("Hello")
```

is correct, but:

```python
Print("Hello")
```

is incorrect.

---

# 13. Comments

A comment explains the purpose of code.

Comments begin with `#`.

```python
# Store the price and quantity
price = 100
quantity = 3

# Calculate the final bill
final_bill = price * quantity

print("Final bill =", final_bill)
```

Python ignores comments while executing the program.

Use comments to explain useful intent, not every obvious detail.

---

# 14. Complete execution trace

Study this program:

```python
print("Program started")

price = 100
quantity = 3

total = price * quantity
discount = total * 10 / 100
final_bill = total - discount

print("Total =", total)
print("Discount =", discount)
print("Final bill =", final_bill)

print("Program completed")
```

## Step-by-step trace

| Step | Statement | Memory or output |
|---:|---|---|
| 1 | `print("Program started")` | Displays `Program started` |
| 2 | `price = 100` | `price → 100` |
| 3 | `quantity = 3` | `quantity → 3` |
| 4 | `total = price * quantity` | `total → 300` |
| 5 | Calculate discount | `discount → 30.0` |
| 6 | Calculate final bill | `final_bill → 270.0` |
| 7 | Print total | Displays `Total = 300` |
| 8 | Print discount | Displays `Discount = 30.0` |
| 9 | Print final bill | Displays `Final bill = 270.0` |
| 10 | Final `print()` | Displays `Program completed` |

Complete output:

```text
Program started
Total = 300
Discount = 30.0
Final bill = 270.0
Program completed
```

---

# 15. Common mistakes

## Missing quotation marks

Incorrect:

```python
name = Veena D
```

Correct:

```python
name = "Veena D"
```

---

## Using a variable before creating it

Incorrect:

```python
print(city)
city = "Dharmavaram"
```

Correct:

```python
city = "Dharmavaram"
print(city)
```

---

## Incorrect capitalisation

Incorrect:

```python
Print("Hello")
```

Correct:

```python
print("Hello")
```

---

## Joining text and numbers incorrectly

Incorrect:

```python
age = 18
print("Age = " + age)
```

Correct:

```python
age = 18
print("Age =", age)
```

---

## Calculating without printing

```python
number = 10
double = number * 2
```

The calculation is correct, but nothing is displayed.

Add:

```python
print("Double =", double)
```

---

# 16. Feynman explanation

Veena should be able to explain Day 1 like this:

> A variable is a name that stores a value. Every value has a data type, such as text, a whole number, a decimal or a Boolean. Python normally executes statements from top to bottom. We can use stored values in calculations and display results using `print()`.

If this cannot be explained without notes, review the unclear section.

---

# Feynman teach-back questions

Explain these aloud:

1. What does `print()` do?
2. What is a variable?
3. What does `=` mean in Python?
4. Why does creating a variable not display it?
5. What is the difference between `18` and `"18"`?
6. Explain `str`, `int`, `float` and `bool`.
7. What does `type()` do?
8. How does Python evaluate `total = price * quantity`?
9. What happens when a variable is assigned again?
10. Why are meaningful variable names important?
11. What does case-sensitive mean?
12. Why do we use comments?

---

# Day 1 practice

Write these independently:

1. Print `Welcome to Python learning!`
2. Store and print a student’s name, age and course.
3. Store first and last names separately and print the full name.
4. Calculate the area of a rectangle.
5. Calculate the area of a square.
6. Calculate the total and average of three marks.
7. Convert kilometres into metres.
8. Calculate a 10% discount on an amount.
9. Calculate an item bill using price and quantity.
10. Print a number’s double, triple and square.

For each program, explain:

```text
Input or stored values:
Process:
Output:
Data types used:
Execution order:
```

## Completion standard

Day 1 is complete when Veena can create, calculate, trace and explain these programs without copying.
