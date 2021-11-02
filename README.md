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

After looking through [Don't override user provided `--rootdir` in pytest args · Pull Request #17509 · microsoft/vscode-python](https://github.com/microsoft/vscode-python/pull/17509), I set `"python.testing.cwd": "repo/rootdir"`. That discovered tests without error, because `--rootdir` defaults to `cwd`:

```
> ~/Dev/vscode-pytest-workspace/venv/bin/python ~/.vscode-insiders/extensions/ms-python.python-2021.12.1409662628-dev/pythonFiles/testing_tools/run_adapter.py discover pytest -- --rootdir ~/Dev/vscode-pytest-workspace/repo/rootdir -s --cache-clear
cwd: ~/Dev/vscode-pytest-workspace/repo/rootdir
```

But running a test causes an error: `Test result not found for: ./package/test_module.py::test_function`:

```
Running tests (pytest): /Users/bhrutledge/Dev/vscode-pytest-workspace/repo/rootdir/package/test_module.py::test_function
Running test with arguments: --rootdir /Users/bhrutledge/Dev/vscode-pytest-workspace --override-ini junit_family=xunit1 --junit-xml=/var/folders/bw/ttgk1hcs0k989q2yzmxkzn8c0000gp/T/tmp-65261qZongVTbsrkA.xml ./package/test_module.py::test_function
Current working directory: /Users/bhrutledge/Dev/vscode-pytest-workspace/repo/rootdir
Workspace directory: /Users/bhrutledge/Dev/vscode-pytest-workspace
Run completed, parsing output
Test result not found for: ./package/test_module.py::test_function

Total number of tests expected to run: 1
Total number of tests run: 0
Total number of tests passed: 0
Total number of tests failed: 0
Total number of tests failed with errors: 0
Total number of tests skipped: 0
Total number of tests with no result data: 1
Finished running tests!
```

It seems `--rootdir` when running tests is still `${workspaceFolder}`, which results in the wrong path being saved to the junit XML:

```xml
<testcase classname="repo.rootdir.package.test_module" name="test_function" file="repo/rootdir/package/test_module.py" line="2" time="0.000" />
```
