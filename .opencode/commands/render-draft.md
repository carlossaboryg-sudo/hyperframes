---
description: Fast draft render of HyperFrames composition
agent: build
subtask: true
---

Render the HyperFrames composition in $ARGUMENTS using draft quality for fast iteration:

```bash
npx hyperframes render $ARGUMENTS --quality draft --fps 30
```

If $ARGUMENTS is empty, render the current project directory.
Report the output file path and file size.
