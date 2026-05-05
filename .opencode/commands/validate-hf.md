---
description: Validate HyperFrames composition in headless Chrome
agent: hf-validate
subtask: true
---

Run `npx hyperframes validate` on the project directory $ARGUMENTS.
Check for console errors, WCAG contrast issues, missing assets.
Report all findings. If $ARGUMENTS is empty, validate the current project directory.
