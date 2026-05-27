# Fixing the bug

If you completed Phase 1, you should have seen some test failures in your output. Our users are very attentive too
and had already filed an issue. It's on GitHub, let's go and see.

The first goal here is to understand the main idea of the issue and find the questions
you already have when reading it for the first time.

[Open the issue]($issueLink)

## Understand the issue

So you have read the issue. Now let's make sure the issue description is clear.

First, what exactly is the problem: is it wrong logic in production code, wrong tests, or something else?
What is expected behavior? What is actual? Write down your answers — even a
single sentence for each. You'll need them when you write your PR description later.

## Reproduce the problem

Before diving into the code, let's isolate the specific failing test by running:

```bash

uv run pytest tests/test_validation_error_status_code.py -v
```

Read the failure output carefully — it tells you exactly what status code was returned versus what was expected.
Look for lines starting with `AssertionError` or `assert` — they show the expected value on one side and the actual
value on the other.

> **Note:** If the test passes instead of failing, make sure you're on the correct branch and haven't already applied
> a fix.

## Explore the codebase

Now that you've seen the failure, it's time to find the problematic code.

The test told you what status code it *expected* — now search the codebase for where that status code is set.

1. Open the failing test file and read what it expects. If you're not sure where it is, use
   [Search Everywhere](project-hub://search-everywhere) (<kbd>Shift</kbd><kbd>Shift</kbd>) to locate it by name. Look for `assert` statements — they
   tell you the expected status code and response body.
2. You now know the status code the test expects. Use [Find in Files](project-hub://search) (<kbd>⌘</kbd><kbd>Shift</kbd><kbd>F</kbd>) to search for the
   *wrong* status code that is currently being returned — it must be set somewhere in the application code.
   Search for it as a plain number (e.g. `400`) and look through the results to find where it is returned for
   validation errors. You can use the **File Mask** field (e.g. `*.py`) to limit results to Python files and
   cut through the noise.
3. Open that file and read the surrounding code. You should see a function that builds an HTTP response — that's
   the place where the status code needs to change.

<details id="code-base-exploration-hint" >
<summary>Open here if you need an answer</summary>

Search for the status code number that the test currently receives (shown in the assertion failure output).
You should find it inside a handler function in the `fastapi/` source directory — that is the file you need to edit.

</details>

> **Tip:** [Find in Files](project-hub://search) (<kbd>⌘</kbd><kbd>Shift</kbd><kbd>F</kbd>) searches across the whole codebase at once — it's much faster than
> opening files one by one. Try searching for the numeric status code to narrow down
> the right handler quickly. Use the **File Mask** field (e.g. `*.py`) to skip non-Python files and keep results
> manageable.

## Plan your change

Now that you've found the handler that sets the status code, it's time to plan your change.

When you planed your fix, verify that:

1. you have found the **root cause** — and not fixing the symptom.
2. you are introducing the  **minimal change** needed?
3. that you are not braking anything else.

<details id="fix-location-hint" >
<summary>I don't get where to write my fix, help needed!</summary>
By now, your search should have shown you the handler where the status code gets set — that's the file you need to edit.
</details> 

> **Important:** Don't change the tests. The tests are correct — they describe expected behavior per the HTTP
> specification and FastAPI's own documentation. Your job is to fix the application code.

## Implement and test

Before editing any code, create a new branch for your fix:

```bash

git checkout -b fix/validation-status-code
```

Make your fix, then verify:

```bash

# Run just the affected tests first for a faster feedback loop
uv run pytest tests/test_validation_error_status_code.py -v

# Then run the full test suite to make sure nothing else broke
uv run bash scripts/test.sh
```

> **Note:** If `scripts/test.sh` doesn't work, you can run `uv run pytest` to execute the full test suite instead.

Check that:

- All previously failing tests now pass
- All other tests still pass
- The fix is a minimal, focused change

If new tests fail after your change that weren't failing before, your fix may be too broad. Revisit Step 4 and make
sure you're only changing what's necessary.

> **If you're stuck:** Re-read the error output from pytest — it shows exactly which value was expected and which was
> actually returned. If that's not enough, try adding `print()` statements in the handler code to inspect what status
> code is being set and why. Run the targeted test again to see your debug output.

> **Optional:** Can you find other test files that also cover validation error handling? Do they all pass now too?
> Try searching for `422` in `tests/` — that's the HTTP status code for "Unprocessable Entity", which is what
> validation errors should return.

## What's next?

When all tests pass, you're ready to submit your work. Head over to **Phase 3: Creating a Pull Request**.

## Useful links

- [FastAPI Contributing Guide](https://fastapi.tiangolo.com/contributing/)
- [FastAPI Exception Handling Docs](https://fastapi.tiangolo.com/tutorial/handling-errors/)
- [Architecture Overview](../CLAUDE.md)
