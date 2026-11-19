# CI/CD and the Smoke Test

INFO 465, Week 14. This README goes with the weather example from class.

## What this is

This repository shows how a team gets a computer to check every pull request before a person reviews it. The check is a small test called a smoke test, and GitHub Actions runs it for us each time a pull request is opened.

## Why it is useful

A reviewer can approve a change that looks fine and still breaks the app. In class we renamed one function in `build_db.py`. The change was one line long and looked harmless, but the app could no longer start. The smoke test caught it in about 30 seconds, before anyone had read the code.

## The two ideas

**CI** stands for continuous integration. Every time someone opens a pull request, GitHub starts a fresh machine, gets the code with that change in it, installs the packages, and runs the test. A pass shows up as a green check on the pull request. A failure shows up as a red X.

**CD** stands for continuous deployment. Once a change is merged to `main`, it reaches the live app without anyone copying files. Streamlit Community Cloud already does this for you. It watches the repository and redeploys the app whenever `main` changes.

Together they make one pipeline:

```
open a pull request
   |
   v
CI runs the smoke test
   |
   v
a teammate reviews
   |
   v
the pull request is merged
   |
   v
the app redeploys          (CD)
   |
   v
the user sees the new version
```

## What a smoke test is

A smoke test is the cheapest test you can write. It does not check whether the answers are right. It only checks whether the program runs from start to finish.

It is different from a unit test. A unit test checks one function and checks that its answer is correct. A smoke test runs the whole program and checks that something came out.

## The two files

### `smoke_test.py`

This file sits at the top of the repository, next to `build_db.py`.

```python
# Smoke test: builds the database and checks that WEATHER has rows.
import sqlite3
from build_db import build

build()

conn = sqlite3.connect("project.db")
rows = conn.execute("SELECT COUNT(*) FROM WEATHER").fetchone()[0]
conn.close()

print("WEATHER rows:", rows)
assert rows > 0, "The database built, but WEATHER is empty"
```

The test can fail in two ways. If `build_db.py` is broken, Python stops with an error at the `import` line or at `build()`. If the database builds but the table is empty, the `assert` line stops the program with an error. Either kind of error turns the check red.

The test does not call the weather API and it does not need a key. It builds the database from the snapshots that are already saved in the repository.

### `.github/workflows/ci.yml`

This file tells GitHub Actions when to run the test and how.

```yaml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  smoke-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r requirements.txt
      - run: python smoke_test.py
```

| Line | What it does |
|---|---|
| `on: pull_request` | Runs the job when a pull request is opened or updated. |
| `branches: [main]` | Only for pull requests aimed at `main`. |
| `smoke-test:` | The name of the job. This is the name you see on the pull request. |
| `checkout`, `setup-python`, `pip install` | Get the code, get Python, and get the packages. |
| `python smoke_test.py` | Runs the test. |

Indentation matters in a YAML file. If the spacing is off, GitHub cannot read the file and the check never appears.

## How to try it

1. Make a new branch.

   ```
   git switch main
   git pull
   git switch -c add-ci
   ```

2. Add the two files shown above. Run the test by hand to make sure it works.

   ```
   python smoke_test.py
   ```

   You should see `WEATHER rows:` followed by a number.

3. Commit and push.

   ```
   git add .
   git commit -m "Add CI smoke test"
   git push -u origin add-ci
   ```

4. Open a pull request. A push does not open one for you. GitHub often shows a yellow bar with a **Compare & pull request** button after a push to a new branch. If the bar is not there, go to the **Pull requests** tab, click **New pull request**, and choose your branch.

5. Scroll down to the box above the merge button. A yellow dot next to `CI / smoke-test` means the test is running. It should turn into a green check in about 30 seconds.

## When the check is red

Click the name of the check and open the last step. The error message at the bottom tells you what went wrong. Fix the problem on the same branch, commit, and push again. The pull request stays open and the check runs again on its own.

## Adding this to your team project

This is optional and it is not graded.

Copy the two files into your team repository. In `smoke_test.py`, change `WEATHER` to the name of one of your own tables. If your Python files are in a `src` folder, put `smoke_test.py` in `src` and change the last line of the workflow to `python src/smoke_test.py`. Then open a pull request the same way you always do.

## Does a green check replace the code review?

No. The test tells you that the program runs and that the change did not break something that used to work. The reviewer still has to decide whether this is the change the story asked for, whether the acceptance criteria are met, and whether a teammate can read the code.

## Where to get help

Ask in the class channel or bring your question to class. If the check never shows up on your pull request, open the **Actions** tab in your repository. A workflow file that GitHub cannot read is listed there with an error.

GitHub's own documentation for GitHub Actions is at https://docs.github.com/en/actions.

## Who maintains this

The INFO 465 instructor maintains this example for the Fall 2026 class at VCU.
