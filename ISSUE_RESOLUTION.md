# Issue Resolution: Backslash at end of comment lines swallowed following code

## Problem Description
In Mako templates, single-line comments start with `##`.
Previously, `mako.lexer.Lexer.match_control_line` grouped both `%` control lines and `##` comment lines together under a regular expression that supported Python line continuation:
```python
r"(?<=^)[\t ]*(%(?!%)|##)[\t ]*((?:(?:\\\r?\n)|[^\r\n])*)"
r"(?:\r?\n|\Z)"
```
When a comment line ended with a backslash `\` (such as a Windows file path `C:\path\to\` or a trailing backslash), the lexer treated the backslash followed by a newline as a line continuation (`(?:\\\r?\n)`). As a result, the subsequent code or template line was swallowed into the comment node and omitted from execution or rendering.

## Root Cause
Single-line comments (`##`) do not have line continuation syntax; backslashes on comment lines are literal characters. Only Python control lines starting with `%` should support line continuation.

## Solution
Updated `match_control_line` in `mako/lexer.py` to differentiate between `%` and `##`:
- `%` control lines continue to match `((?:(?:\\\r?\n)|[^\r\n])*)`, allowing line continuation with backslashes.
- `##` comment lines match `([^\r\n]*)`, stopping at the newline regardless of any trailing backslash.

Added regression tests in `test/test_lexer.py` verifying that comments ending with backslashes terminate at the newline and do not swallow subsequent template text or control statements.
