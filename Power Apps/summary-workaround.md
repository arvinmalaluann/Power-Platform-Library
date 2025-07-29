# PowerApps Multi-Value Item Summary Workaround

This PowerApps formula provides a workaround approach for summarizing items that include **multiple values in a single line**, specifically focusing on a `Department` field that contains delimited values. The script processes raw data, flattens multi-value fields, and groups/summarizes items by department and duration open.

---

## 💡 Purpose

In scenarios where a single field (e.g., `Department`) contains **multiple comma-separated values**, conventional grouping and filtering methods in PowerApps fall short. This script splits these values, normalizes them, and generates a grouped summary based on the number of days an item has been open.

---

## 📋 Overview of the Steps

1. **Fetch Items from RADAR API**
   Use a custom connector or service call (`RADAR-FetchItems`) to retrieve item data.

   ```powerapps
   UpdateContext({
       returnItems: 'RADAR-FetchItems'.Run().bodyvalue
   });
   ```

2. **Parse the JSON result into a table**

   ```powerapps
   UpdateContext({parsedResults: Table(ParseJSON(returnItems))});
   ```

3. **Flatten and transform the parsed data**
   Normalize fields like `ID`, `Title`, `TargetDateClosure`, `DaysOpen`, and clean up the `Department` string to remove unusual delimiters.

   ```powerapps
   ClearCollect(
       ignore_actionItems,
       AddColumns(
           parsedResults,
           ID, Text(ThisRecord.Value.ID),
           Title, Text(ThisRecord.Value.Title),
           TargetDateClosure, Text(ThisRecord.Value.TargetDateClosure),
           DaysOpen, Text(ThisRecord.Value.DaysOpen),
           Department, Substitute(
                           Substitute(
                               Substitute(
                                   Text(ThisRecord.Value.Department),
                                   "+_+", ""
                               ),
                               "<delim>", ","
                           ),
                           ",,", ","
                       )
       )
   );
   ```

4. **Split multi-value department fields into individual rows**
   Use `Split()` and `ForAll()` to flatten each record by department.

   ```powerapps
   Clear(ignore_colTemp);
   ForAll(
       ignore_actionItems As outerRecord,
       Collect(
           ignore_colTemp,
           AddColumns(
               Split(outerRecord.Department, ",") As dept,
               ID, outerRecord.ID,
               Title, outerRecord.Title,
               Department, Trim(dept.Value),
               DaysOpen, Value(outerRecord.DaysOpen)
           )
       )
   );
   ```

5. **Store the flattened data**

   ```powerapps
   ClearCollect(flatDepartmentItems, ignore_colTemp);
   ```

6. **Group and summarize by department and age of the item**
   Group records by department and calculate how many fall into age brackets:

   * Over 14 days
   * Between 8 and 14 days
   * Between 1 and 7 days

   ```powerapps
   ClearCollect(
       colGroupedCounts,
       AddColumns(
           GroupBy(
               flatDepartmentItems,
               Department, DepartmentName
           ),
           Over14,
           CountRows(Filter(DepartmentName, Int(DaysOpen) > 14)),
           Between8and14,
           CountRows(Filter(DepartmentName, Int(DaysOpen) >= 8 && Int(DaysOpen) <= 14)),
           Between1and7,
           CountRows(Filter(DepartmentName, Int(DaysOpen) >= 0 && Int(DaysOpen) <= 7))
       )
   );
   ```

---

## 🛠️ Key Techniques Used

* `ParseJSON()` and `Table()` to convert API response into usable data
* `AddColumns()` and `Split()` to normalize nested or delimited fields
* `GroupBy()` and `CountRows()` for summary statistics
* String sanitization with `Substitute()`

---

## 📊 Output

The final output is a **collection (`colGroupedCounts`)** summarizing item counts per department by the number of days the item has been open, divided into meaningful age buckets.

---

## 📌 Notes

* This approach assumes departments are delimited using `"+"`, `"<delim>"`, or double commas — and cleans them accordingly.
* Can be extended for additional fields or filtering logic as needed.
