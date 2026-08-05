Day 5 is testing + code quality. This is the first “senior mindset” day: do not only write code, prove it works.

**Day 5 Goal**
You should understand:

- What automated tests are
- Why functions should return values, not only print
- How to use `pytest`
- How to format and lint Python code
- How to think in test cases

Start:

```powershell
cd "$env:USERPROFILE\Documents\python-fullstack-journey"
.\.venv\Scripts\Activate.ps1
mkdir day-05-testing
cd day-05-testing
code .
```

Create this structure:

```text
day-05-testing/
├── contact_utils.py
└── test_contact_utils.py
```

Create `contact_utils.py`:

```python
def create_contact(name, phone, email):
    name = name.strip()
    phone = phone.strip()
    email = email.strip()

    if not name:
        return None

    if not phone:
        return None

    return {
        "name": name,
        "phone": phone,
        "email": email,
    }


def find_contacts(contacts, search_text):
    search_text = search_text.strip().lower()

    matches = []

    for contact in contacts:
        if search_text in contact["name"].lower():
            matches.append(contact)

    return matches


def delete_contact_by_index(contacts, contact_number):
    index = contact_number - 1

    if index < 0 or index >= len(contacts):
        return None

    return contacts.pop(index)
```

Create `test_contact_utils.py`:

```python
from contact_utils import create_contact, delete_contact_by_index, find_contacts


def test_create_contact_success():
    contact = create_contact("Hari", "9876543210", "hari@example.com")

    assert contact == {
        "name": "Hari",
        "phone": "9876543210",
        "email": "hari@example.com",
    }


def test_create_contact_removes_extra_spaces():
    contact = create_contact("  Hari  ", "  9876543210  ", "  hari@example.com  ")

    assert contact["name"] == "Hari"
    assert contact["phone"] == "9876543210"
    assert contact["email"] == "hari@example.com"


def test_create_contact_without_name_returns_none():
    contact = create_contact("", "9876543210", "hari@example.com")

    assert contact is None


def test_create_contact_without_phone_returns_none():
    contact = create_contact("Hari", "", "hari@example.com")

    assert contact is None


def test_find_contacts_by_name():
    contacts = [
        {"name": "Hari", "phone": "111", "email": "hari@example.com"},
        {"name": "Krishna", "phone": "222", "email": "krishna@example.com"},
    ]

    matches = find_contacts(contacts, "hari")

    assert len(matches) == 1
    assert matches[0]["name"] == "Hari"


def test_find_contacts_is_case_insensitive():
    contacts = [
        {"name": "Harikrishna", "phone": "111", "email": "hari@example.com"},
    ]

    matches = find_contacts(contacts, "HARI")

    assert len(matches) == 1


def test_delete_contact_by_valid_number():
    contacts = [
        {"name": "Hari", "phone": "111", "email": "hari@example.com"},
        {"name": "Krishna", "phone": "222", "email": "krishna@example.com"},
    ]

    deleted_contact = delete_contact_by_index(contacts, 1)

    assert deleted_contact["name"] == "Hari"
    assert len(contacts) == 1


def test_delete_contact_by_invalid_number_returns_none():
    contacts = [
        {"name": "Hari", "phone": "111", "email": "hari@example.com"},
    ]

    deleted_contact = delete_contact_by_index(contacts, 5)

    assert deleted_contact is None
    assert len(contacts) == 1
```

Install test tools if you did not install them on Day 0:

```powershell
python -m pip install pytest black ruff
```

Run tests:

```powershell
pytest
```

Expected result:

```text
8 passed
```

Run formatter:

```powershell
black .
```

Run linter:

```powershell
ruff check .
```

**Your Challenge**
Add two more tests:

- `find_contacts()` should return empty list when nothing matches
- `delete_contact_by_index()` should return `None` when contact number is `0`

Commit:

```powershell
git status
git add .
git commit -m "Day 5 add tests for contact utilities"
```

Day 5 is complete when you can explain:

- What `assert` does
- Why tests should be automated
- Why `create_contact()` returns `None` instead of printing error messages
- Why we separated logic from user input/output
