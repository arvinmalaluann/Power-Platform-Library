# PowerApps: Patching Records (New and Existing)

This guide explains how to **patch (create or update)** records in PowerApps using three different methods:

1. **Using custom controls (no form)**
2. **Using an existing form**
3. **Updating existing records using an ID**

---

## 1. Creating a New Record Using Custom Controls (No Form)

### 📌 Scenario
You’re designing a custom UI using controls like `TextInput`, `Dropdown`, etc., and you want to save new data directly to a data source like SharePoint, Dataverse, or SQL without using a form.

### ✅ Example Controls
- `txtName` (TextInput)
- `txtAge` (TextInput)
- `drpGender` (Dropdown)

### 🧪 Sample Data Source
**SharePoint List:** `Employees`  
**Columns:**
- `Title` (Text) → used for Name
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

* `Defaults(Employees)` initializes a blank record.
* The third parameter defines the new values mapped from user inputs.

---

## 2. Creating a New Record Using a Form

### 📌 Scenario

You are using a Form control (e.g., `Form1`) connected to your data source, and want to submit a new record programmatically.

### ✅ Setup

* Form control: `Form1`
* Data source: `Employees`
* Mode: `NewForm(Form1)`

### 🧩 Patch Syntax

```powerapps
Patch(
    Employees,
    Defaults(Employees),
    Form1.Updates
)
```

Or more commonly:

```powerapps
SubmitForm(Form1)
```

### 🔍 Explanation

* `Form1.Updates` returns the current values in the form.
* `SubmitForm(Form1)` handles validation and submission.

---

## 3. Updating an Existing Record Using Its ID

### 📌 Scenario

You want to update an existing record by referencing its ID (e.g., from a variable or input), rather than relying on a form or gallery selection.

### ✅ Example

* Known ID: stored in `varRecordID`
* New values from inputs: `txtName`, `txtAge`, `drpGender`

### 🧩 Patch Syntax

```powerapps
Patch(
    Employees,
    LookUp(Employees, ID = varRecordID),
    {
        Title: txtName.Text,
        Age: Value(txtAge.Text),
        Gender: drpGender.Selected.Value
    }
)
```

### 🔍 Explanation

* `LookUp()` retrieves the record with the given ID.
* The third argument updates the fields.

✅ *This method is useful when you want to patch a record after retrieving it through an API, variable, or lookup result.*

---

## 🧠 Tips

* Use `If(Form1.Valid, SubmitForm(Form1))` to validate before submission.
* Use `Value()` to convert `TextInput` to numbers.
* Use `Selected.Value` for dropdowns or choice fields.
* Always verify that the ID exists before patching using it.

---

## 💡 Common Errors and Fixes

| Issue                         | Fix                                               |
| ----------------------------- | ------------------------------------------------- |
| Delegation warning            | Use delegable functions; avoid complex LookUps    |
| Text input for number field   | Wrap input with `Value(txtInput.Text)`            |
| Dropdown not saving correctly | Use `drp.Selected.Value` or `drp.Selected.Result` |
| Form not submitting           | Ensure `Form1.Valid` is `true` before submission  |

---

## 🛠 Sample Button Code (Custom Controls)

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

| Method                    | Use When                                 | Best For                |
| ------------------------- | ---------------------------------------- | ----------------------- |
| Custom Patch (no form)    | You want full control of UI              | Advanced custom layouts |
| Form + Patch / SubmitForm | You want built-in validation & structure | Quick forms             |
| Patch using ID            | You know which record to update directly | External triggers, APIs |
