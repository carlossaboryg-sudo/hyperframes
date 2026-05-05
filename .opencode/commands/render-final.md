---
description: High-quality final render of HyperFrames composition
agent: build
subtask: true
---

First run lint and validate on $ARGUMENTS:

```bash
npx hyperframes lint $ARGUMENTS
npx hyperframes validate $ARGUMENTS
```

If both pass without errors, render at high quality:

```bash
npx hyperframes render $ARGUMENTS --quality high --fps 30
```

If any errors exist, fix them first before rendering.
If $ARGUMENTS is empty, work on the current project directory.
Report the output file path and file size.
