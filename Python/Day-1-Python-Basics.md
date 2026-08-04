Assumption: Day 0 setup is complete. If not, finish the install/verification first. Day 1 is about Python fundamentals plus professional workflow.

**Day 1 Goal**
By end of today, you should be able to:

- Run Python from VS Code terminal
- Understand variables, data types, input, conditions, loops, functions
- Build one small command-line project
- Commit your work to Git

**Start**
Open PowerShell:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-01-python-basics
cd day-01-python-basics
code .
```

Create a file named `basics.py`.

```python
name = input("Enter your name: ")
age = int(input("Enter your age: "))

print(f"Hello {name}, you are {age} years old.")

if age >= 18:
    print("You are eligible to apply for jobs.")
else:
    print("Focus on learning first.")

skills = ["Python", "Git", "HTML", "CSS", "JavaScript"]

print("\nYour full stack path:")
for skill in skills:
    print(f"- Learn {skill}")

def calculate_experience(start_year, current_year):
    return current_year - start_year

experience = calculate_experience(2026, 2036)
print(f"\nTarget experience mindset: {experience} years")
```

Run it:

```powershell
python basics.py
```

**Main Project**
Create `student_profile.py`.

```python
def get_student_data():
    name = input("Name: ")
    degree = input("Degree: ")
    masters = input("Masters: ")
    target_role = input("Target role: ")
    current_skill = input("Current strongest skill: ")

    return {
        "name": name,
        "degree": degree,
        "masters": masters,
        "target_role": target_role,
        "current_skill": current_skill,
    }


def print_profile(profile):
    print("\nCandidate Profile")
    print("-----------------")
    print(f"Name: {profile['name']}")
    print(f"Education: {profile['degree']} + {profile['masters']}")
    print(f"Target Role: {profile['target_role']}")
    print(f"Strongest Skill: {profile['current_skill']}")

    if profile["target_role"].lower() == "python full stack developer":
        print("Plan: Python, Django/FastAPI, SQL, React, Git, deployment")
    else:
        print("Plan: First clarify your target role.")


student = get_student_data()
print_profile(student)
```

Run:

```powershell
python student_profile.py
```

**Git Commit**
After both programs run correctly:

```powershell
git status
git add .
git commit -m "Day 1 Python basics"
```

**Today’s Concepts**
Learn these clearly, not just by reading:

- `str`, `int`, `float`, `bool`
- `input()`
- `print()`
- `if / else`
- `for` loop
- function with `def`
- dictionary `{}`

**Day 1 Completion Checklist**

- `basics.py` runs without error
- `student_profile.py` runs without error
- You understand every line you typed
- Git commit is done
- You can explain what a variable, function, loop, and dictionary are

After completing this, send me either your code or any error you get. Then Day 2 will be Python problem solving: lists, dictionaries, functions, and 10 interview-style beginner problems.
