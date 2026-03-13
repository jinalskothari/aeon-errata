# Aeon's Surreal Renaissance — Overall / Book-wide Errata & Suggestions

Reported by: Jinal Kothari
Total items: 3

These issues are not specific to a single chapter and apply to the book as a whole.

---

### 1. Scope of Surrealist blocks can be confusing

**Category:** Suggestion
**Impact:** Medium
**Fix suggested:** Yes

Some Surrealist blocks only work correctly if a previous block has already been executed in the reader's session. This dependency is not always made clear, and a reader who jumps to a specific section or re-opens the book may find that examples fail with no obvious explanation.

**Suggested fix:** Make each Surrealist block self-contained by bundling all prerequisite statements (table definitions, seed data, etc.) internally — even if they are hidden from the visible display. The reader would see only the new snippet but the block would internally ensure all dependencies are met before running.

This would establish a clear mental model: every block is independent and runnable on its own.

---

### 2. `RETURN` keyword shown where it is not required

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

### 3. Window title sometimes shows wrong chapter

**Category:** Error
**Impact:** Minimal

The browser/tab window title, which is supposed to reflect the current chapter number and name, occasionally displays the wrong chapter as the reader navigates through the book. This appears to be an intermittent rendering issue rather than a consistent one.
