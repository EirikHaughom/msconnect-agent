---
description: Look up the current user's Microsoft role title, discipline, IC level, and role description from WorkIQ and the Microsoft Role Library.
---

# Role Lookup

Look up the current user's Microsoft role information using WorkIQ and the Microsoft Role Library.

## Steps

### 1. Get Job Title, Discipline, and Level

Ask WorkIQ: _"What is my current job title, role discipline, and IC level at Microsoft?"_

Extract:
- **Job title** (e.g., Principal Technical Program Manager)
- **Discipline** (e.g., Technical Program Management)
- **IC level** (e.g., L66)
- **Organization** (e.g., Industry Solutions Norway)
- **Manager** name
- **Country / location** (e.g., Norway, Germany, Japan)

### 2. Find Role Description

Ask WorkIQ: _"Search SharePoint for documents from msrolelibrary.microsoft.com about the role description for [job title]. What are the key accountabilities, responsibilities, and expectations?"_

Extract:
- **Key accountabilities** (3-6 items)
- **Core responsibilities** grouped by category
- **Level-specific expectations** (what distinguishes this level from adjacent levels)
- **Measures of success** if available

> **Note:** The Microsoft Role Library (msrolelibrary.microsoft.com) is the authoritative source, but WorkIQ accesses role descriptions through SharePoint-hosted interpretations and standards pages that reference it. These are reliable for practical use.

### 3. Present and Confirm

Present the role summary to the user in this format:

```
**Role:** [job title]
**Discipline:** [discipline]
**Level:** [IC level]
**Organization:** [org]
**Country:** [country]
**Locale languages:** [languages — derived from country, e.g., "Norwegian, English" for Norway]

**Key Accountabilities:**
- [accountability 1]
- [accountability 2]
- ...
```

Ask the user to confirm this is correct, or to provide corrections (e.g., if they recently changed roles, or if their day-to-day differs from their formal title).

### 4. Return Confirmed Role

Return the confirmed role description for use by the calling agent or workflow.

## Fallback

If WorkIQ returns no role description results:
- Ask the user to describe their role accountabilities manually
