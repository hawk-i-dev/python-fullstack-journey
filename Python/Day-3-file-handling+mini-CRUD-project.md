Day 3 is Python file handling + mini CRUD project. This is important because full stack apps are mostly CRUD: create, read, update, delete data.

**Day 3 Goal**
You should understand:

- Reading and writing files
- JSON data
- `try / except`
- Lists of dictionaries
- Building a menu-driven program

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-03-file-handling
cd day-03-file-handling
code .
```

Create `contact_book.py`:

```python
import json

FILE_NAME = "contacts.json"


def load_contacts():
    try:
        with open(FILE_NAME, "r") as file:
            return json.load(file)
    except FileNotFoundError:
        return []


def save_contacts(contacts):
    with open(FILE_NAME, "w") as file:
        json.dump(contacts, file, indent=4)


def add_contact(contacts):
    name = input("Name: ")
    phone = input("Phone: ")
    email = input("Email: ")

    contact = {
        "name": name,
        "phone": phone,
        "email": email,
    }

    contacts.append(contact)
    save_contacts(contacts)
    print("Contact added.")


def list_contacts(contacts):
    if len(contacts) == 0:
        print("No contacts found.")
        return

    for index, contact in enumerate(contacts, start=1):
        print(f"\nContact {index}")
        print(f"Name: {contact['name']}")
        print(f"Phone: {contact['phone']}")
        print(f"Email: {contact['email']}")


def search_contacts(contacts):
    search_name = input("Enter name to search: ").lower()

    found = False

    for contact in contacts:
        if search_name in contact["name"].lower():
            print(f"\nName: {contact['name']}")
            print(f"Phone: {contact['phone']}")
            print(f"Email: {contact['email']}")
            found = True

    if not found:
        print("No matching contact found.")


def show_menu():
    print("\nContact Book")
    print("1. Add contact")
    print("2. List contacts")
    print("3. Search contact")
    print("4. Exit")


contacts = load_contacts()

while True:
    show_menu()
    choice = input("Choose an option: ")

    if choice == "1":
        add_contact(contacts)
    elif choice == "2":
        list_contacts(contacts)
    elif choice == "3":
        search_contacts(contacts)
    elif choice == "4":
        print("Goodbye.")
        break
    else:
        print("Invalid choice.")
```

Run:

```powershell
python contact_book.py
```

Test it properly:

1. Add 2 contacts.
2. Exit the program.
3. Run it again.
4. Choose list contacts.

If old contacts are still there, file handling is working.

**Challenge**
Add option `5. Delete contact`.

Expected behavior:

- Show all contacts with numbers.
- Ask which contact number to delete.
- Remove that contact from the list.
- Save the updated contacts to `contacts.json`.

Hint:

```python
contacts.pop(index)
save_contacts(contacts)
```

But remember: user sees contact number starting from `1`, Python list index starts from `0`.

After finishing:

```powershell
git status
git add .
git commit -m "Day 3 file handling contact book"
```

Day 3 is complete when you can explain what `contacts.json` is and why we use `json.dump()` and `json.load()`.
