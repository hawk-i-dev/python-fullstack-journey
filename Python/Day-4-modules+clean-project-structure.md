Day 4 is Python modules + clean project structure. Today you move from “one file script” to “small application structure”.

**Day 4 Goal**
You should understand:

- What a module is
- How to import your own code
- Why we split code into files
- How real projects are organized
- Basic error handling improvement

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-04-modules
cd day-04-modules
code .
```

Create this structure:

```text
day-04-modules/
├── main.py
├── storage.py
├── contacts.py
└── contacts.json
```

Create `storage.py`:

```python
import json

FILE_NAME = "contacts.json"


def load_contacts():
    try:
        with open(FILE_NAME, "r") as file:
            return json.load(file)
    except FileNotFoundError:
        return []
    except json.JSONDecodeError:
        return []


def save_contacts(contacts):
    with open(FILE_NAME, "w") as file:
        json.dump(contacts, file, indent=4)
```

Create `contacts.py`:

```python
from storage import save_contacts


def add_contact(contacts):
    name = input("Name: ").strip()
    phone = input("Phone: ").strip()
    email = input("Email: ").strip()

    if not name or not phone:
        print("Name and phone are required.")
        return

    contact = {
        "name": name,
        "phone": phone,
        "email": email,
    }

    contacts.append(contact)
    save_contacts(contacts)
    print("Contact added.")


def list_contacts(contacts):
    if not contacts:
        print("No contacts found.")
        return

    for index, contact in enumerate(contacts, start=1):
        print(f"\nContact {index}")
        print(f"Name: {contact['name']}")
        print(f"Phone: {contact['phone']}")
        print(f"Email: {contact['email']}")


def search_contacts(contacts):
    search_name = input("Enter name to search: ").strip().lower()

    matches = []

    for contact in contacts:
        if search_name in contact["name"].lower():
            matches.append(contact)

    if not matches:
        print("No matching contact found.")
        return

    list_contacts(matches)


def delete_contact(contacts):
    list_contacts(contacts)

    if not contacts:
        return

    try:
        contact_number = int(input("Enter contact number to delete: "))
        index = contact_number - 1

        if index < 0 or index >= len(contacts):
            print("Invalid contact number.")
            return

        deleted_contact = contacts.pop(index)
        save_contacts(contacts)
        print(f"Deleted contact: {deleted_contact['name']}")

    except ValueError:
        print("Please enter a valid number.")
```

Create `main.py`:

```python
from contacts import add_contact, delete_contact, list_contacts, search_contacts
from storage import load_contacts


def show_menu():
    print("\nContact Book")
    print("1. Add contact")
    print("2. List contacts")
    print("3. Search contact")
    print("4. Delete contact")
    print("5. Exit")


def main():
    contacts = load_contacts()

    while True:
        show_menu()
        choice = input("Choose an option: ").strip()

        if choice == "1":
            add_contact(contacts)
        elif choice == "2":
            list_contacts(contacts)
        elif choice == "3":
            search_contacts(contacts)
        elif choice == "4":
            delete_contact(contacts)
        elif choice == "5":
            print("Goodbye.")
            break
        else:
            print("Invalid choice.")


main()
```

Create empty `contacts.json` with this content:

```json
[]
```

Run:

```powershell
python main.py
```

**Your Task**
Test these cases:

- Add contact with name, phone, email
- Add contact without name
- Search existing contact
- Search non-existing contact
- Delete contact with correct number
- Delete contact with invalid number
- Close app and reopen it

**Important Concept**
This line imports functions from another file:

```python
from contacts import add_contact, delete_contact, list_contacts, search_contacts
```

That means `contacts.py` is now a module.

**Professional Improvement**
At the end of `main.py`, change this:

```python
main()
```

to this:

```python
if __name__ == "__main__":
    main()
```

Reason: it prevents `main()` from running automatically if another file imports `main.py`.

Commit:

```powershell
git status
git add .
git commit -m "Day 4 organize contact book into modules"
```

Day 4 is complete when you can explain this in your own words:

“`main.py` controls the program flow, `contacts.py` handles contact operations, and `storage.py` handles file reading/writing.”
