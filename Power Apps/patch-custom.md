# PowerApps: Patching a New Record (With and Without a Form)

This guide covers how to **patch (create)** new records in PowerApps using two different methods:

1. **Custom controls (no form)**
2. **Using an existing form control**

---

## 1. Patching a New Record Using Custom Controls (No Form)

### 📌 Scenario
You are building a custom UI (e.g., TextInputs, Dropdowns) instead of a Form control, and want to save the data to a data source like SharePoint, Dataverse, or SQL.

### ✅ Example Controls
- TextInput: `txtName`, `txtAge`
- Dropdown: `drpGender`

### 🧪 Sample Data Source
Assume a SharePoint list called `Employees` with columns:
- `Title` (Text) → used for `Name`
- `Age` (Number)
- `Gender` (Choice)

### 🧩 Patch Syntax

```powerapps
Patch(
    Employees,
    Defaults(Employees),
    {
        Title: txtName.Text,
        Age: Value(txtAge.Text),
        Gender: drpGender.Selected.Value
    }
)
````

### 🔍 Explanation

* `Patch()` is used to create or update records.
* `Defaults(Employees)` creates a blank record structure.
* Inside the curly braces `{}`, map each field to the value from the control.

---

## 2. Patching a New Record Using an Existing Form

### 📌 Scenario

You are using a Form control (e.g., `Form1`) connected to a data source and you want to submit the form data programmatically.

### ✅ Setup

* A form control: `Form1`
* Data source: `Employees`
* Mode: New (`NewForm(Form1)`)

### 🧩 Patch Syntax

```powerapps
Patch(
    Employees,
    Defaults(Employees),
    Form1.Updates
)
```

Or, more commonly for simplicity:

```powerapps
SubmitForm(Form1)
```

### 🔍 Explanation

* `Form1.Updates` retrieves all field values from the form.
* `SubmitForm(Form1)` is a shortcut that validates and submits the form data.

---

## 🧠 Tips

* Use `If(Form1.Valid, SubmitForm(Form1))` to ensure data is valid before submission.
* When using custom controls, handle validation manually before using `Patch()`.
* Always use `Value()` to convert text inputs to numbers, and `Selected.Value` for dropdowns or choices.

---

## 💡 Common Errors and Fixes

| Issue                         | Fix                                               |
| ----------------------------- | ------------------------------------------------- |
| Delegation warning            | Ensure fields are supported by your data source   |
| Text input for number field   | Wrap input with `Value(txtInput.Text)`            |
| Dropdown not saving correctly | Use `drp.Selected.Value` or `drp.Selected.Result` |
| Form not submitting           | Check if `Form1.Valid = true` before submission   |

---

## 🛠 Sample Button Code for Custom Controls

```powerapps
If(
    !IsBlank(txtName.Text) && !IsBlank(txtAge.Text),
    Patch(
        Employees,
        Defaults(Employees),
        {
            Title: txtName.Text,
            Age: Value(txtAge.Text),
            Gender: drpGender.Selected.Value
        }
    ),
    Notify("Please complete all fields.", NotificationType.Error)
)
```

---

## ✅ Summary

| Method                 | Use When                                | Best For             |
| ---------------------- | --------------------------------------- | -------------------- |
| Custom Patch (no form) | You want full UI control, custom layout | Advanced scenarios   |
| Form + Patch/Submit    | You prefer a managed UI with validation | Simpler, quick forms |
