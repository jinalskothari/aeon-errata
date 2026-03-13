# Aeon's Surreal Renaissance — Errata & Enhancements

Reported by: Jinal Kothari
Date: 2026-03-13
Total items: 20

---

## Chapter 4: The security door

### 1. Repeating examples of functions

**Category:** Enhancement
**Impact:** Minimal

In the section introducing function naming conventions, the same function names (`array::sort()` and `rand::uuid::v7()`) appear in both the initial illustrative list and the follow-up "other function names" list, making the latter feel redundant.

**Context (as it appears in the book):**

> Some function paths are longer if they represent a further refinement of its behaviour, such as sorting or the type of output.
>
> ```
> array::sort()
> array::sort::asc()
> array::sort::desc()
>
> rand::uuid()
> rand::uuid::v4()
> rand::uuid::v7()
> ```
>
> Here are some other function names to give you an idea of the variety of functions available.
>
> ```
> array::sort()        ← repeated
> string::len()
> string::semver::set::major()
> value::diff()
> rand::uuid::v7()     ← repeated
> ```

---

### 2. Difference in shown vs. actual output for `@ with entries()`

**Category:** Factual Inaccuracy
**Impact:** Medium

The output shown in the book for the `SELECT @.{...}.entries() FROM town` query does not match what Surrealist actually returns.

**Query:**

```surql
CREATE town:sukh SET name = "Sukh", population = 2955;
CREATE town:the_naimo SET
    name = "The Naimo",
    population = 7490,
    data = {
        location: "Northwest of Toria",
        geography: "Coastal town"
    };

SELECT @.{
    name,
    population
}.entries() FROM town;
```

**Shown in book:**

```json
[
  [
    [ "name", "Sukh" ],
    [ "population", 2955 ]
  ],
  [
    [ "name", "The Naimo" ],
    [ "population", 7490 ]
  ]
]
```

**Actual output:**

```json
[
    {
        "": [
            [ "name", "Sukh" ],
            [ "population", 2955 ]
        ]
    },
    {
        "": [
            [ "name", "The Naimo" ],
            [ "population", 7490 ]
        ]
    }
]
```

---

### 3. Example queries fail in the datetimes section

**Category:** Error
**Impact:** Maximum
**Fix suggested:** Yes

Section: *Datetimes, string prefixes, durations, and sleeping*

Both examples in this section produce an error: `Incorrect arguments for function time::format(). Argument 1 was the wrong type. Expected 'datetime' but found 'NONE'`.

The root cause is that `at` (a field being set in the same `CREATE` statement) is not yet available as a value when `at_historical` tries to use it via `time::format(at, ...)`.

**Example 1 (failing):**

```surql
LET $date = "0003-04";
LET $info = "First ever democratic elections in Toria";

CREATE ONLY event SET
    at = (IF string::is_datetime($date) {
        <datetime>$date
    } ELSE {
        <datetime>($date + "-15")
    }),
    at_historical = time::format(at, "Month %m Day %d, %Y years since Toria's glorious founding"),
    info = $info;
```

**Fix:** Compute `at` into a variable first, then use it in `CREATE`.

```surql
LET $date = '0003-04';
LET $info = 'First ever democratic elections in Toria';
LET $at = (IF string::is_datetime($date) { <datetime> $date } ELSE { <datetime> ($date + '-15') });
CREATE ONLY event SET
    at = $at,
    at_historical = time::format($at, "Month %m Day %d, %Y years since Toria's glorious founding"),
    `info` = $info;
```

**Example 2 (failing):**

```surql
LET $date = "0424-04-24T08:00:00Z";
LET $info = "Start of day 50 of the project to restore civilization";

CREATE ONLY event SET
    at = IF string::is_datetime($date) {
        <datetime>$date
    } ELSE {
        <datetime>$date + "-15"
    },
    at_historical = time::format(at, "Month %m Day %d, %Y glorious years since Toria's founding"),
    info = $info;
```

**Fix:**

```surql
LET $date = "0424-04-24T08:00:00Z";
LET $info = "Start of day 50 of the project to restore civilization";
LET $at = (IF string::is_datetime($date) { <datetime> $date } ELSE { <datetime> ($date + '-15') });
CREATE ONLY event SET
    at = $at,
    at_historical = time::format($at, "Month %m Day %d, %Y years since Toria's glorious founding"),
    `info` = $info;
```

With the fix applied, the subsequent query also works as expected:

```surql
LET $today = <datetime>"0424-04-25";
SELECT $today - at AS time_passed, * FROM event;
```

---

### 4. Example of un-parsable input only shows the first error

**Category:** Factual Inaccuracy
**Impact:** Minimal
**Fix suggested:** Yes

The following block contains three separate statements intentionally designed to fail parsing:

```surql
RETURN d'27000024-06-06T12:00:00Z';
RETURN r'person:::::aeon';
RETURN u'01912b31-d67d-704b-b49d-WRONGINPUT';
```

However, because the first statement fails to parse, the remaining two are never evaluated. The reader only sees one error, not the three that the book implies they will see.

**Fix:** Split each statement into its own Surrealist block so each error is shown independently.

---

### 5. Possibly unintended rendering of the Author's TODO

**Category:** Enhancement
**Impact:** Minimal

In the section at: `chapter-04#tip-check-the-documentation`

A TODO note from the author appears to be visible in the rendered book. This is likely leftover draft content that was not intended for publication. A screenshot was attached in the original report.

---

### 6. Update reference to `type::thing()` with `type::record()`

**Category:** Factual Inaccuracy
**Impact:** Medium

In the section *Record functions to find table names and IDs*, and in the answer to Practice Q1, the book refers to `type::thing()`. This function has been renamed/superseded by `type::record()` in current versions of SurrealDB.

**Affected text (examples):**

> So how did we know that `record::tb()` and `type::thing()` were available to use, and how many other functions are there?

> Practice Q1 Answer: You can use the `type::thing()` function to construct their record IDs.

---

### 7. Practice Question 3 does not motivate the use of `record::tb()`

**Category:** Enhancement
**Impact:** Medium
**Fix suggested:** Yes

Practice Q3 asks:

> How would you select each of these towns with a single field called `description` that automatically has an output like "A town called Toria" or "A town called Abeston"?

Because the data only comes from the `town` table (which will always appear after `FROM`), it never occurs to the reader to use `record::tb()`. The function is redundant in this context.

**Suggested fix:** Extend the example to select from both `town` and `village` so that `record::tb()` is genuinely needed to distinguish the table name.

**Setup:**

```surql
CREATE town SET name = 'Toria';
CREATE village SET name = 'Sukh';
CREATE village SET name = 'Abeston';
```

**Question:** How would you select each of these records with a single field called `description` that automatically outputs "A town called Toria" or "A village called Sukh"?

**Answer:**

```surql
SELECT VALUE 'A ' + record::tb(id) + ' called ' + name AS description FROM town, village;
```

---

## Chapter 5: A long road to travel

### 8. Adding schema-non-compliant data fails silently — book implies otherwise

**Category:** Factual Inaccuracy
**Impact:** Significant

The book implies that inserting non-compliant data into a `SCHEMAFULL` table either partially succeeds or behaves differently from what actually occurs. In practice, the operation fails entirely and no record is created.

**Query:**

```surql
DEFINE TABLE OVERWRITE person TYPE ANY SCHEMAFULL PERMISSIONS NONE;
CREATE person SET
    name = "Random person",
    age = 25,
    other = {
        some: "other",
        random: "data"
    };
```

**Actual result:**

```
"Found field 'age', but no such field exists for table 'person'"
```

No record is created. The book's description of this behaviour does not match the actual database behaviour.

---

### 9. Author TODO visible just before Q3

**Category:** Enhancement
**Impact:** Minimal

A draft TODO note from the author is visible in the rendered chapter just before Practice Question 3. This appears to be unintentional. Screenshots were attached in the original report.

---

### 10. Un-runnable code snippet

**Category:** Enhancement
**Impact:** Minimal
**Fix suggested:** Yes

The following snippet appears in the chapter with a copy button, suggesting it is runnable:

```surql
CREATE |event:1..=10| SET ...
SELECT * FROM event:1..=5;
```

The `SET ...` placeholder is not valid SurrealQL and will cause a parse error if the reader attempts to run it. Since the snippet has a copy button, readers will reasonably expect it to be executable.

**Suggested fix:** Remove the `SET ...` portion, or replace the ellipsis with a concrete field assignment.

---

## Chapter 6: Inspiration from the past

### 11. Missing image — group of people

**Category:** Error
**Impact:** Minimal

An image described as *"A black and white woodblock-style image of a group of people..."* is referenced in the book text but does not render. The surrounding narrative text reads correctly, but the image placeholder is empty.

---

### 12. Missing image — German folk tale ship scene

**Category:** Error
**Impact:** Minimal

An image described as *"A scene from a German folk tale showing a fantastic ship..."* is referenced in the book but does not render. The surrounding narrative text reads:

> Looking up, you see one of the pictures from one of the books that has given you such comfort during this time.
>
> *[missing image]*
>
> The people of the old world were so good at combining precision with beauty, at blending technology and art. How did they do it?

---

### 13. `Building with Kind: cat` — field definition missing from query block

**Category:** Example not working
**Impact:** Medium
**Fix suggested:** Yes

In the section on building with `KIND: cat`, the Surrealist query block does not include the `DEFINE FIELD` statement that is required for the example to work as intended. Readers who run only the shown block will not see the expected behaviour.

**Fix:** Include the field definition in the same query block as the `KIND: cat` definition so the example is self-contained.

---

### 14. Cat-food relationship examples do not work

**Category:** Example not working
**Impact:** Medium

The examples demonstrating graph relationships between cats and food fail to produce the expected results. The queries appear correct as written but do not execute successfully in the embedded Surrealist blocks. No additional details were captured beyond the failure itself.

---

### 15. Query does not show intended result — data not present in Surrealist block

**Category:** Example not working
**Impact:** Medium

A query block in this chapter does not return the expected results because the data it depends on (inserted in an earlier block) is not present when this block runs in isolation. Screenshots were attached in the original report showing the discrepancy.

---

### 16. `INFO FOR DB` returns tables and functions unrelated to the chapter

**Category:** Example not working
**Impact:** Medium

When the book instructs the reader to run `INFO FOR DB` to verify that a graph edge table (`parent_of`) exists, the output also contains tables and functions from previous chapters (e.g., `order`, `product`, `fn::pound_to_usd`, etc.) that are unrelated to the current chapter's content.

This makes it harder to find the relevant output and may confuse readers who expect a clean result.

**Context:** The expected `parent_of` table is indeed present, but it is buried among unrelated schema definitions from the reader's accumulated session state.

---

### 17. Unnecessary cast to string — field is already a string

**Category:** Enhancement
**Impact:** Minimal

In the following query, `in.name` and `out.name` are already strings. The explicit `<string>` casts are redundant.

```surql
SELECT
    VALUE
        "Parent: "     + <string>in.name  +
        ". Daughter: " + <string>out.name
FROM parent_of;
```

The casts do not cause an error but may mislead readers into thinking such casts are necessary when concatenating record field values.

---

## Overall / Book-wide

### 18. Scope of Surrealist blocks can be confusing

**Category:** Suggestion
**Impact:** Medium
**Fix suggested:** Yes

Some Surrealist blocks only work correctly if a previous block has already been executed in the reader's session. This dependency is not always made clear, and a reader who jumps to a specific section or re-opens the book may find that examples fail with no obvious explanation.

**Suggested fix:** Make each Surrealist block self-contained by bundling all prerequisite statements (table definitions, seed data, etc.) internally — even if they are hidden from the visible display. The reader would see only the new snippet but the block would internally ensure all dependencies are met before running.

This would establish a clear mental model: every block is independent and runnable on its own.

---

### 19. `RETURN` keyword shown where it is not required

**Category:** Suggestion
**Impact:** Medium

Throughout the book, examples use the explicit `RETURN` keyword in situations where it is optional:

```surql
RETURN <datetime>"2024-06-06";
```

This is equivalent to:

```surql
<datetime>"2024-06-06";
```

It is worth considering whether consistently showing the optional `RETURN` keyword helps or hinders a reader's understanding of when it is actually required.

---

### 20. Window title sometimes shows wrong chapter

**Category:** Error
**Impact:** Minimal

The browser/tab window title, which is supposed to reflect the current chapter number and name, occasionally displays the wrong chapter as the reader navigates through the book. This appears to be an intermittent rendering issue rather than a consistent one.
