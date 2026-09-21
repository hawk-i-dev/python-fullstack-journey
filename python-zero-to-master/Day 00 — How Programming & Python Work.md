# Python Day 0 — How Programming & Python Work

## Day 0 goal

Before learning Python syntax, understand:

- What programming means
- How a computer follows instructions
- How Python executes code
- Input → Process → Output
- What an algorithm is
- Why errors occur
- How to think before coding
- How to learn using the 80/20 rule and Feynman technique

---

## 1. What is programming?

Programming means giving a computer a sequence of clear instructions to complete a task.

A computer:

- Does not guess what we mean
- Does not automatically correct our logic
- Follows instructions exactly
- Follows instructions in the given order

### Real-life example

Instructions for making tea:

```text
1. Take a vessel.
2. Add water.
3. Heat the water.
4. Add tea powder.
5. Add milk.
6. Boil the tea.
7. Serve it.
```

These instructions form an **algorithm**.

A program is an algorithm written using a programming language.

---

## 2. What is Python?

Python is a programming language used to communicate instructions to a computer.

Python is popular because its syntax is comparatively simple and readable.

```python
print("Welcome to Python")
```

This instruction tells Python to display:

```text
Welcome to Python
```

Python can be used for:

- Problem-solving
- Automation
- Web development
- Data analysis
- Artificial intelligence
- Machine learning
- Testing
- Desktop applications

For now, our main goals are:

> Learn Python fundamentals and develop logical problem-solving skills.

---

## 3. The essential programming tools

### Editor or IDE

An editor is where we write code.

Examples:

- IDLE
- PyCharm
- Visual Studio Code
- Replit
- Pydroid 3 on Android

An IDE usually provides:

- A code editor
- A Run button
- Error highlighting
- A terminal
- Debugging features

### Python file

Python programs are generally stored in files ending with `.py`.

```text
practice.py
calculator.py
student_result.py
```

### Python interpreter

The interpreter reads and executes Python instructions.

Think of it as a translator:

```text
Python code
     ↓
Python interpreter
     ↓
Computer performs the instructions
```

### Terminal or console

The terminal displays:

- Program output
- Input requests
- Error messages

---

## 4. How Python executes a program

Python normally processes code from top to bottom.

```python
print("Program started")

name = "Veena D"

print("Hello,", name)

print("Program completed")
```

Execution order:

```text
1. Print Program started
2. Store "Veena D" in name
3. Read the value stored in name
4. Print Hello, Veena D
5. Print Program completed
```

Output:

```text
Program started
Hello, Veena D
Program completed
```

### Important idea

Creating a value does not display it automatically.

```python
name = "Veena D"
```

This only stores the value.

To display it:

```python
print(name)
```

---

## 5. Input → Process → Output

Most programs follow three basic stages.

```text
Input → Process → Output
```

### Input

Information given to the program.

Examples:

- Name
- Age
- Price
- Quantity
- Marks

### Process

Work performed using the input.

Examples:

- Addition
- Multiplication
- Comparison
- Searching
- Sorting

### Output

The result produced by the program.

Examples:

- Total bill
- Average marks
- Pass or Fail
- Search result

### Example: shop bill

```python
price = 50
quantity = 3

total = price * quantity

print("Total =", total)
```

Here:

```text
Input:
price = 50
quantity = 3

Process:
total = price × quantity

Output:
Total = 150
```

---

## 6. What is an algorithm?

An algorithm is a clear sequence of steps used to solve a problem.

It is written before converting the solution into programming code.

### Problem

Calculate the area of a rectangle.

### Algorithm

```text
1. Start.
2. Get the length.
3. Get the width.
4. Multiply length by width.
5. Display the area.
6. Stop.
```

### Python program

```python
length = 10
width = 4

area = length * width

print("Area =", area)
```

Output:

```text
Area = 40
```

### Why write an algorithm first?

It helps us:

- Understand the problem
- Divide it into smaller steps
- Avoid random coding
- Find missing requirements
- Convert the solution into code more easily

The most important habit is:

> First determine the steps. Then write the Python syntax.

---

## 7. Code versus output

Code is the instruction written by the programmer.

```python
print("Hello, Veena D!")
```

Output is what the computer displays:

```text
Hello, Veena D!
```

They are not the same thing.

| Code | Output |
|---|---|
| `print("Python")` | `Python` |
| `print(10 + 20)` | `30` |
| `print(5 * 4)` | `20` |

---

## 8. Python syntax

Syntax means the grammar or writing rules of a programming language.

Correct:

```python
print("Hello")
```

Incorrect:

```python
print("Hello"
```

The incorrect code is missing the closing parenthesis.

Just as English sentences follow grammar rules, Python programs follow syntax rules.

Python is also case-sensitive:

```python
print("Hello")
```

is correct, but:

```python
Print("Hello")
```

is incorrect because `print` and `Print` are different names.

---

## 9. Three important types of errors

Errors are a normal part of programming. They help us identify what needs correction.

### 1. Syntax error

Python’s writing rules are violated.

Incorrect:

```python
print("Hello"
```

Correct:

```python
print("Hello")
```

Python cannot execute the incorrect program because the closing parenthesis is missing.

---

### 2. Runtime error

The syntax is valid, but the program fails while executing.

```python
number = 10
result = number / 0
```

The program starts but fails because division by zero is impossible.

---

### 3. Logic error

The program runs, but the result is incorrect.

```python
length = 10
width = 4

area = length + width

print(area)
```

Output:

```text
14
```

The program runs successfully, but the formula is wrong.

Correct formula:

```python
area = length * width
```

Output:

```text
40
```

Logic errors are often harder to find because Python may not display an error message.

---

## 10. How to read an error

When Python shows an error:

1. Do not panic.
2. Read the final line of the error.
3. Identify the error type.
4. Look at the mentioned line number.
5. Check spelling, quotations, parentheses and indentation.
6. Correct one problem at a time.
7. Run the program again.

Treat errors as information:

> The program is showing where your understanding or syntax needs correction.

---

## 11. How to think before writing code

For every problem, answer these questions:

```text
1. What is the problem asking?
2. What is the input?
3. What should the output be?
4. What calculation or decision is required?
5. What steps would I follow manually?
6. What unusual cases might occur?
```

### Example problem

Calculate the total marks for three subjects.

```text
Input:
Maths, Science and Social marks

Process:
Add the three marks

Output:
Total marks
```

Algorithm:

```text
1. Store the three marks.
2. Add them.
3. Store the result.
4. Print the result.
```

Only after this should we write the program.

---

## 12. The 80/20 rule for learning Python

The 80/20 rule means focusing first on the small set of concepts responsible for most practical results.

The essential foundations are:

- Variables
- Input and output
- Conditions
- Loops
- Functions
- Strings
- Lists
- Dictionaries
- Problem decomposition
- Testing and debugging

Do not try to memorise every Python method immediately.

For each concept:

1. Understand its purpose.
2. Learn its basic syntax.
3. Trace one example.
4. Write it independently.
5. Solve a small problem.
6. Review mistakes.

Master the commonly used concepts before studying rare features.

---

## 13. The Feynman technique

The Feynman technique tests whether you genuinely understand something.

### Step 1: Learn

Read the concept and study an example.

### Step 2: Explain simply

Explain it as if teaching a young child.

Avoid copying textbook sentences.

### Step 3: Find knowledge gaps

When you cannot explain a step, identify exactly what is unclear.

### Step 4: Review and simplify

Study that part again and explain it using simpler words.

### Example teach-back

Question:

> What is a program?

Good simple explanation:

> A program is a list of instructions that tells a computer what to do and in which order.

Question:

> What is an algorithm?

Good simple explanation:

> An algorithm is the step-by-step solution we prepare before writing code.

If Veena can explain a concept clearly without looking at notes, she probably understands it.

---

## 14. Day 0 complete mental model

```text
Problem
   ↓
Understand the input and output
   ↓
Solve it manually
   ↓
Write an algorithm
   ↓
Convert the steps into Python
   ↓
Python interpreter executes the code
   ↓
Observe the output
   ↓
Test and correct errors
   ↓
Explain the solution in simple words
```

---

# Feynman teach-back questions

Explain these aloud without looking at the answers:

1. What is programming?
2. What is Python?
3. What does the Python interpreter do?
4. What is the difference between code and output?
5. Explain Input → Process → Output.
6. What is an algorithm?
7. Why should we plan before coding?
8. What is a syntax error?
9. What is a runtime error?
10. What is a logic error?
11. Why are errors useful?
12. How does Python normally execute a program?

---

# Day 0 independent challenge

## Rectangle Area System

Without writing Python code first:

1. Identify the input.
2. Identify the process.
3. Identify the output.
4. Write the algorithm.
5. Write the expected result when:

```text
Length = 8
Width = 5
```

Then write the Python program and explain every line aloud.

## Completion standard

Day 0 is complete when Veena can independently explain:

> A problem is converted into an algorithm, the algorithm is converted into Python code, the interpreter executes the code, and the program produces output or an error that helps us improve it.
