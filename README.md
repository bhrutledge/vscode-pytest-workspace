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

Setting `--rootdir` to `${workspaceFolder}/repo/rootdir` (the parent directory of `pytest.ini`, matching [pytest's behavior](https://docs.pytest.org/en/stable/customize.html#initialization-determining-rootdir-and-configfile)) causes tests to be discovered from that directory, instead of `package`:

```
> ~/Dev/vscode-pytest-workspace/venv/bin/python ~/.vscode-insiders/extensions/ms-python.python-2021.12.1409662628-dev/pythonFiles/testing_tools/run_adapter.py discover pytest -- -s --cache-clear --rootdir ~/Dev/vscode-pytest-workspace/repo/rootdir ~/Dev/vscode-pytest-workspace/repo/rootdir
cwd: ~/Dev/vscode-pytest-workspace/repo/rootdir
Error 2021-11-01 21:49:35: Error discovering pytest tests:
 [r [Error]: ============================= test session starts ==============================
platform darwin -- Python 3.9.7, pytest-6.2.5, py-1.10.0, pluggy-1.0.0
rootdir: /Users/bhrutledge/Dev/vscode-pytest-workspace/repo/rootdir, configfile: pytest.ini
collected 1 item / 1 error

<Package package>
  <Module test_module.py>
    <Function test_function>

==================================== ERRORS ====================================
____________________ ERROR collecting scripts/test_error.py ____________________
ImportError while importing test module '/Users/bhrutledge/Dev/vscode-pytest-workspace/repo/rootdir/scripts/test_error.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.9/3.9.7_1/Frameworks/Python.framework/Versions/3.9/lib/python3.9/importlib/__init__.py:127: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
scripts/test_error.py:1: in <module>
    import not_found
E   ModuleNotFoundError: No module named 'not_found'
=========================== short test summary info ============================
ERROR scripts/test_error.py
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

To avoid that error, I can add `--ignore scripts` to `pytestArgs`, which allows test discovery and running to succeed:

```
> ~/Dev/vscode-pytest-workspace/venv/bin/python ~/.vscode-insiders/extensions/ms-python.python-2021.12.1409662628-dev/pythonFiles/testing_tools/run_adapter.py discover pytest -- -s --cache-clear --rootdir ~/Dev/vscode-pytest-workspace/repo/rootdir --ignore scripts ~/Dev/vscode-pytest-workspace/repo/rootdir
cwd: ~/Dev/vscode-pytest-workspace/repo/rootdir

> ~/Dev/vscode-pytest-workspace/venv/bin/python -m pytest --override-ini junit_family=xunit1 --junit-xml=/var/folders/bw/ttgk1hcs0k989q2yzmxkzn8c0000gp/T/tmp-65261jEOVCVjMxFJo.xml --rootdir ~/Dev/vscode-pytest-workspace/repo/rootdir --ignore scripts ./package/test_module.py::test_function
cwd: ~/Dev/vscode-pytest-workspace/repo/rootdir

Running tests (pytest): /Users/bhrutledge/Dev/vscode-pytest-workspace/repo/rootdir/package/test_module.py::test_function
Running test with arguments: --override-ini junit_family=xunit1 --junit-xml=/var/folders/bw/ttgk1hcs0k989q2yzmxkzn8c0000gp/T/tmp-65261jEOVCVjMxFJo.xml --rootdir /Users/bhrutledge/Dev/vscode-pytest-workspace/repo/rootdir --ignore scripts ./package/test_module.py::test_function
Current working directory: /Users/bhrutledge/Dev/vscode-pytest-workspace/repo/rootdir
Workspace directory: /Users/bhrutledge/Dev/vscode-pytest-workspace
Run completed, parsing output
./package/test_module.py::test_function Passed

Total number of tests expected to run: 1
Total number of tests run: 1
Total number of tests passed: 1
Total number of tests failed: 0
Total number of tests failed with errors: 0
Total number of tests skipped: 0
Total number of tests with no result data: 0
Finished running tests!
```

However, given that `pytest` works from the command line (thanks to the `testpaths` option in `repo/rootdir/pytest.ini`), adding `--ignore` shouldn't be necessary.
