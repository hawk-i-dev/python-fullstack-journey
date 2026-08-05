Day 2 focus: Python data structures and functions. This is where you stop writing loose code and start thinking like a developer.

**Day 2 Goal**
By end of today, you should understand:

- Lists
- Dictionaries
- Loops over data
- Functions that receive and return values
- Basic input validation
- Breaking a program into small parts

Open PowerShell:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-02-data-structures
cd day-02-data-structures
code .
```

Create `lists_practice.py`:

```python
marks = [78, 85, 62, 90, 55]

total = sum(marks)
average = total / len(marks)

print(f"Marks: {marks}")
print(f"Total: {total}")
print(f"Average: {average}")
print(f"Highest: {max(marks)}")
print(f"Lowest: {min(marks)}")

for mark in marks:
    if mark >= 60:
        print(f"{mark}: Pass")
    else:
        print(f"{mark}: Fail")
```

Run:

```powershell
python lists_practice.py
```

Create `dictionary_practice.py`:

```python
student = {
    "name": "Harikrishna",
    "degree": "Degree",
    "masters": "MCA",
    "target_role": "Python Full Stack Developer",
    "skills": ["Python", "Git", "SQL"],
}

print(student["name"])
print(student["target_role"])

student["skills"].append("Django")

for skill in student["skills"]:
    print(f"Learning: {skill}")
```

Run:

```powershell
python dictionary_practice.py
```

**Main Project: Student Marks Report**

Create `marks_report.py`:

```python
def get_marks():
    marks = []

    for subject in ["Python", "SQL", "Web", "Git", "Problem Solving"]:
        mark = int(input(f"Enter marks for {subject}: "))
        marks.append(mark)

    return marks


def calculate_total(marks):
    return sum(marks)


def calculate_average(marks):
    return sum(marks) / len(marks)


def get_grade(average):
    if average >= 90:
        return "A"
    if average >= 75:
        return "B"
    if average >= 60:
        return "C"
    if average >= 40:
        return "D"
    return "Fail"


def print_report(name, marks):
    total = calculate_total(marks)
    average = calculate_average(marks)
    grade = get_grade(average)

    print("\nStudent Report")
    print("--------------")
    print(f"Name: {name}")
    print(f"Marks: {marks}")
    print(f"Total: {total}")
    print(f"Average: {average:.2f}")
    print(f"Grade: {grade}")


student_name = input("Enter student name: ")
student_marks = get_marks()
print_report(student_name, student_marks)
```

Run:

```powershell
python marks_report.py
```

**Your Challenge**
Improve `marks_report.py` so it rejects invalid marks:

- Marks below `0` should not be accepted
- Marks above `100` should not be accepted
- If invalid, ask again

Hint: create a function named `get_valid_mark(subject)`.

**Git Commit**

```powershell
git status
git add .
git commit -m "Day 2 Python data structures"
```

Day 2 is complete only when you can explain why `marks` is a list, why `student` is a dictionary, and why we used separate functions instead of writing everything in one place.

Send me your improved `marks_report.py`, especially the validation part.
