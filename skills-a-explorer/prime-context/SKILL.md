---
name: prime-context
description: Prime Claude with project guidelines and coding standards from documentation files
allowed-tools: Bash(sed:*)
---

Here are all coding standards of our project. Follow them strictly. Verify that all code adheres to the documented standards before considering a task complete.

@README.md
!`find docs/standards/ -type f | sed 's|^|@|'`

For maximum efficiency, whenever you need to perform multiple independent operations, invoke all relevant tools simultaneously rather than sequentially.

Just reply "Ready!" then continue what you were doing.