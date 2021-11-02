# vscode-pytest-workspace

Trying to get pytest working in a deeply-nested Python project.

Tests should only be discovered from `repo/rootdir/package`:

```
$ pwd
/Users/bhrutledge/Dev/vscode-pytest-workspace/repo/rootdir

$ pytest
============================= test session starts ==============================
platform darwin -- Python 3.9.7, pytest-6.2.5, py-1.10.0, pluggy-1.0.0
rootdir: /Users/bhrutledge/Dev/vscode-pytest-workspace/repo/rootdir, configfile: pytest.ini, testpaths: package
collected 1 item

package/test_module.py .                                                 [100%]

============================== 1 passed in 0.00s ===============================
```

However, the "Python: Configure Tests" command only allows `repo` to be selected. Furthermore, `--rootdir` seems to default to `${workspaceFolder}`.
This results in a collection error for a test in `repo/rootdir/scripts`:

```
> ~/Dev/vscode-pytest-workspace/venv/bin/python ~/.vscode-insiders/extensions/ms-python.python-2021.12.1409662628-dev/pythonFiles/testing_tools/run_adapter.py discover pytest -- --rootdir ~/Dev/vscode-pytest-workspace -s --cache-clear repo
cwd: ~/Dev/vscode-pytest-workspace
Error 2021-11-01 21:24:41: Error discovering pytest tests:
 [r [Error]: ============================= test session starts ==============================
platform darwin -- Python 3.9.7, pytest-6.2.5, py-1.10.0, pluggy-1.0.0
rootdir: /Users/bhrutledge/Dev/vscode-pytest-workspace
collected 1 item / 1 error

<Package package>
  <Module test_module.py>
    <Function test_function>

==================================== ERRORS ====================================
_____________ ERROR collecting repo/rootdir/scripts/test_error.py ______________
ImportError while importing test module '/Users/bhrutledge/Dev/vscode-pytest-workspace/repo/rootdir/scripts/test_error.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.9/3.9.7_1/Frameworks/Python.framework/Versions/3.9/lib/python3.9/importlib/__init__.py:127: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
repo/rootdir/scripts/test_error.py:1: in <module>
    import not_found
E   ModuleNotFoundError: No module named 'not_found'
=========================== short test summary info ============================
ERROR repo/rootdir/scripts/test_error.py
!!!!!!!!!!!!!!!!!!!! Interrupted: 1 error during collection !!!!!!!!!!!!!!!!!!!!
====================== 1 test collected, 1 error in 0.04s ======================

Traceback (most recent call last):
  File "/Users/bhrutledge/.vscode-insiders/extensions/ms-python.python-2021.12.1409662628-dev/pythonFiles/testing_tools/run_adapter.py", line 22, in <module>
    main(tool, cmd, subargs, toolargs)
  File "/Users/bhrutledge/.vscode-insiders/extensions/ms-python.python-2021.12.1409662628-dev/pythonFiles/testing_tools/adapter/__main__.py", line 100, in main
    parents, result = run(toolargs, **subargs)
  File "/Users/bhrutledge/.vscode-insiders/extensions/ms-python.python-2021.12.1409662628-dev/pythonFiles/testing_tools/adapter/pytest/_discovery.py", line 44, in discover
    raise Exception("pytest discovery failed (exit code {})".format(ec))
Exception: pytest discovery failed (exit code 2)

	at ChildProcess.<anonymous> (/Users/bhrutledge/.vscode-insiders/extensions/ms-python.python-2021.12.1409662628-dev/out/client/extension.js:17:38446)
	at Object.onceWrapper (events.js:422:26)
	at ChildProcess.emit (events.js:315:20)
	at maybeClose (internal/child_process.js:1048:16)
	at Process.ChildProcess._handle.onexit (internal/child_process.js:288:5)]
```
