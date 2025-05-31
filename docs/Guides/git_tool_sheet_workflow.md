
## Work Instruction System: Tool Sheets and CNC Programs with Git Version Control

**Project Title**: Git-Based Tool Sheet and CNC Program Integration  
**Author**: Brian F.  
**Company**: [U.S. Aerospace Company]  
**Date**: 2025-05-30  
**Compliance**: ITAR / DFARS / NIST 800-171 (CUI-Restricted Data)

---

### Objective
To establish a compliant, version-controlled system that maintains accurate setup documentation, CNC programs, and tooling data through Git. This approach enhances traceability, reduces errors, and supports audit-readiness.

---

### Compliance Requirements
- Prohibit use of public Git repositories (e.g., GitHub.com)
- Host Git repositories internally (e.g., GitLab, SSH-accessible server)
- Ensure data remains within U.S. jurisdiction with access restrictions
- Require commit history traceable to user and timestamp

---

### Tools Used
| Tool               | Purpose                          |
|--------------------|-----------------------------------|
| Git                | Version control for all files     |
| Zettlr             | Markdown editor for setup sheets  |
| Internal Git Server| Secure on-prem Git access         |
| Markdown           | Portable and human-readable format|
| CNC Machines       | Mazak (EIA format programs)       |

---

### Recommended Repository Structure
```
/tool-data/
├── 12345-Widget/
│   ├── RevA/
│   │   ├── setup-sheet.md
│   │   ├── op10_mainprogram.eia
│   │   ├── tool-list.csv
│   │   └── fixture-photo.jpg
│   ├── RevB/
│   │   ├── setup-sheet.md
│   │   ├── op10_mainprogram.eia
│   │   └── tool-list.csv
│   └── README.md
```

- Each revision folder contains a complete set of related files
- Operators may view or print the latest commit data
- Engineers can inspect differences using `git diff` or revert when needed

---

### Workflow Example

#### Checking the Latest Revision
1. Navigate to the appropriate part folder (e.g., `tool-data/12345-Widget`)
2. Use `git log` or `git status` to verify you are working on the latest revision folder (e.g., `RevB/`)
3. Confirm with the lead engineer if any uncertainty exists

#### Editing the Setup Sheet
1. Open `setup-sheet.md` using Zettlr
2. Make the required changes (e.g., clamp offset update, tool usage notes)
3. Save the file

#### Editing the CNC Program
1. Open the part file in Esprit CAM software
2. Make applicable edits to the machining operations or post-processing parameters
3. Post-process the program and export the updated `.eia` file
4. Save the updated file into the appropriate revision folder

#### Committing Changes to Git
```bash
cd tool-data/12345-Widget

git add RevB/
git commit -m "RevB: Updated clamp location and tool #12 feedrate"
git push origin main
```

---

### Common Git Commands and Examples

These commands support traceability and review of documentation and code changes:

#### View commit history
```bash
git log --oneline --graph
```
Example:
```
* a8cd9f2 RevB: Updated clamp location and tool #12 feedrate
* 4fa23d1 RevA: Initial release of part 12345-Widget
```

#### View file-level edit history
```bash
git log -- setup-sheet.md
```
Example:
```
commit a8cd9f2a1b24c0fa5e9f4d1a28c7f92b3b7d2923
Author: Brian F. <brian@example.com>
Date:   Mon May 27 14:21:36 2025 -0500

    RevB: Updated clamp location and tool #12 feedrate
```

#### Compare two revisions
```bash
git diff RevA RevB
```
Example:
```
--- a/RevA/setup-sheet.md
+++ b/RevB/setup-sheet.md
@@ -12,7 +12,7 @@
- Clamp offset set to 1.00 in from edge
+ Clamp offset updated to 1.25 in from edge per operator feedback
```

#### View line-by-line author info
```bash
git blame setup-sheet.md
```
Example:
```
a8cd9f2 (Brian F. 2025-05-27 14:21:36 -0500) Clamp offset updated to 1.25 in from edge per operator feedback
4fa23d1 (Brian F. 2025-05-25 09:12:02 -0500) Tool #12 RPM set to 3200
```

#### Check current branch and file status
```bash
git status
```
Example:
```
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   RevB/setup-sheet.md
```

---

### Benefits
- Unified source of truth for each setup revision
- Full visibility and traceability of all changes
- Consistency across shifts and machines
- Conforms to ITAR and NIST 800-171 standards
- Ready for downstream automation, export, or dashboard integration

---
