# 📘 Power Apps: Transforming JSON to Table

This guide outlines the step-by-step process on how to transform a JSON response into a usable table format in Power Apps. This approach is useful when consuming a JSON payload from a Power Automate Flow and converting it into a readable collection for display and interaction.

---

## 🧩 Steps to Convert JSON to Table in Power Apps

### 1. 🔁 Trigger the Flow and Store the Response

Trigger your Power Automate flow and store the returned JSON payload into a local variable using `UpdateContext`.

```powerapps
UpdateContext({
    searchResults: 'KMS-Search'.Run(
        searchKeyword,
        "Not set",  // Department
        "Not set",  // Group
        "Not set",  // Knowledge Area
        "Not set",  // Category
        "Not set",  // Subcategory
        varPath     // File path or relevant filter
    ).payload
});
```

> ℹ️ *The data inside the run function are the arguments.*

---

### 2. 🧮 Parse JSON to Table

Parse the raw JSON using `ParseJSON()` and wrap it in a `Table()` function, then store it in another local variable:

```powerapps
UpdateContext({parsedResults: Table(ParseJSON(searchResults))});
```

---

### 3. 📥 Populate Collection with Cleaned Data

Finally, convert the parsed table into a structured collection using `ClearCollect()` and `AddColumns()`. This makes it easier to reference each property individually.

```powerapps
ClearCollect(
    searchResultsCleaned,
    AddColumns(
        parsedResults,
        Name, Text(ThisRecord.Value.name),
        Extension, Text(ThisRecord.Value.extension),
        Description, Text(ThisRecord.Value.description),
        KnowledgeArea, Text(ThisRecord.Value.knowledgeArea),
        Category, Text(ThisRecord.Value.category),
        SubCategory, Text(ThisRecord.Value.subCategory),
        Created, Text(ThisRecord.Value.created),
        LastModified, Text(ThisRecord.Value.lastModified),
        LastModifiedBy, Text(ThisRecord.Value.lastModifiedBy),
        FolderPath, Text(ThisRecord.Value.folderPath),
        Type, Text(ThisRecord.Value.type),
        ID, Text(ThisRecord.Value.id),
        Group, Text(ThisRecord.Value.group),
        Department, Text(ThisRecord.Value.dept)
    )
);
```

---

## 💡 Notes and Best Practices

* `ClearCollect()` creates a **global collection**, making it accessible across multiple screens.
* On the other hand, `UpdateContext()` is used for **local variables**, which ensures data isolation between screens.
* This design prevents data conflicts when multiple screens run the same logic simultaneously.
* Always **clear the collection** (or recreate it) when navigating screens to maintain clean and predictable data states.

---

## ✅ Outcome

After following the steps above, you will have:

* A clean and structured collection named `searchResultsCleaned`.
* Each JSON field mapped into its own readable column.
* Data ready to be displayed in a gallery, table, or used for further logic in your app.
