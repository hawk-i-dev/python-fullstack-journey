Day 6 is Object-Oriented Programming. This is needed before Django/FastAPI because real backend code uses objects, models, services, and structured data.

**Day 6 Goal**
You should understand:

- Class
- Object
- `__init__`
- `self`
- Instance variables
- Methods
- Converting objects to dictionaries
- Why OOP helps organize larger apps

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-06-oop
cd day-06-oop
code .
```

Create `oop_basics.py`:

```python
class Student:
    def __init__(self, name, degree, masters, target_role):
        self.name = name
        self.degree = degree
        self.masters = masters
        self.target_role = target_role
        self.skills = []

    def add_skill(self, skill):
        self.skills.append(skill)

    def print_profile(self):
        print("\nStudent Profile")
        print("---------------")
        print(f"Name: {self.name}")
        print(f"Education: {self.degree} + {self.masters}")
        print(f"Target Role: {self.target_role}")
        print(f"Skills: {', '.join(self.skills)}")


student = Student("Harikrishna", "Degree", "MCA", "Python Full Stack Developer")

student.add_skill("Python")
student.add_skill("Git")
student.add_skill("SQL")

student.print_profile()
```

Run:

```powershell
python oop_basics.py
```

Now create the main project: `task_manager.py`.

```python
import json

FILE_NAME = "tasks.json"


class Task:
    def __init__(self, title, status="pending"):
        self.title = title
        self.status = status

    def mark_completed(self):
        self.status = "completed"

    def to_dict(self):
        return {
            "title": self.title,
            "status": self.status,
        }


class TaskManager:
    def __init__(self):
        self.tasks = []

    def add_task(self, title):
        if not title.strip():
            print("Task title cannot be empty.")
            return

        task = Task(title.strip())
        self.tasks.append(task)
        self.save_tasks()
        print("Task added.")

    def list_tasks(self):
        if not self.tasks:
            print("No tasks found.")
            return

        for index, task in enumerate(self.tasks, start=1):
            print(f"{index}. {task.title} - {task.status}")

    def complete_task(self, task_number):
        index = task_number - 1

        if index < 0 or index >= len(self.tasks):
            print("Invalid task number.")
            return

        self.tasks[index].mark_completed()
        self.save_tasks()
        print("Task marked as completed.")

    def save_tasks(self):
        task_data = []

        for task in self.tasks:
            task_data.append(task.to_dict())

        with open(FILE_NAME, "w") as file:
            json.dump(task_data, file, indent=4)

    def load_tasks(self):
        try:
            with open(FILE_NAME, "r") as file:
                task_data = json.load(file)

            for item in task_data:
                task = Task(item["title"], item["status"])
                self.tasks.append(task)

        except FileNotFoundError:
            self.tasks = []
        except json.JSONDecodeError:
            self.tasks = []


def show_menu():
    print("\nTask Manager")
    print("1. Add task")
    print("2. List tasks")
    print("3. Complete task")
    print("4. Exit")


def main():
    manager = TaskManager()
    manager.load_tasks()

    while True:
        show_menu()
        choice = input("Choose an option: ").strip()

        if choice == "1":
            title = input("Task title: ")
            manager.add_task(title)
        elif choice == "2":
            manager.list_tasks()
        elif choice == "3":
            try:
                task_number = int(input("Enter task number: "))
                manager.complete_task(task_number)
            except ValueError:
                print("Please enter a valid number.")
        elif choice == "4":
            print("Goodbye.")
            break
        else:
            print("Invalid choice.")


if __name__ == "__main__":
    main()
```

Run:

```powershell
python task_manager.py
```

Test these cases:

- Add 2 tasks
- List tasks
- Complete task number `1`
- Exit and run again
- Confirm tasks are still saved
- Try completing invalid task number
- Try entering text instead of number

**Important Explanation**
In this code:

`Task` represents one task.

`TaskManager` manages many tasks.

`self.tasks` is a list of `Task` objects.

`to_dict()` is needed because JSON cannot directly save Python objects. JSON can save dictionaries, lists, strings, numbers, and booleans.

**Challenge**
Add option `5. Delete task`.

Rules:

- Ask for task number
- Validate the number
- Delete the task
- Save updated tasks to `tasks.json`

Use this idea inside `TaskManager`:

```python
def delete_task(self, task_number):
    index = task_number - 1

    if index < 0 or index >= len(self.tasks):
        print("Invalid task number.")
        return

    deleted_task = self.tasks.pop(index)
    self.save_tasks()
    print(f"Deleted task: {deleted_task.title}")
```

Commit after finishing:

```powershell
git status
git add .
git commit -m "Day 6 object oriented task manager"
```

Day 6 is complete when you can explain what `self` means and why `TaskManager` is better than keeping everything in one long script.
