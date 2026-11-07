# CA1 Reflection

### Question 1: Fractional Parts and Number Literals
In `src/scanner.rs`, the scanner checks whether a dot (`.`) is followed by a digit using `self.peek() == '.' && self.peek_next().is_ascii_digit()`. This two-character lookahead ensures that a dot is only consumed as part of a decimal number if there is at least one digit immediately following it.

For the input `5.`, the scanner first consumes `5` as a `NUMBER` token. When it reaches `.`, it looks ahead to `peek_next()`. Because there is no digit following the dot, the check fails. The scanner stops processing the number token at `5` and leaves `.` to be processed as a separate character (which results in a scanning error). Section 1.4 of the specification explicitly mandates this behavior by stating that fractional parts require at least one digit after the decimal point, meaning standalone trailing dots like `5.` or leading dots like `.5` are not valid floating-point number literals.


### Question 2: Line Counter and EOF Token Line Number
The line counter `self.line` is modified in `src/scanner.rs` at two specific locations:
1. When encountering a newline character (`\n`) during normal token scanning.
2. When encountering a newline character inside a multi-line string literal.

For a file ending with two blank lines, the `EOF` token carries the line number of the **last real token** in the file, rather than the file's final line number. For instance, if the last token is on line 10 followed by two empty lines, the `EOF` token carries line 10. Section 6.1 specifies this rule so that compiler output and error reporting consistently reference meaningful source locations associated with actual code, rather than trailing empty whitespace at the end of a file. If a file contains no tokens at all, `EOF` defaults to line 1.


### Question 3: Debugging and Git History
During development, I initially failed the `tests/phase-1/invalid/unterminated_string.kobo` test case. 

My misunderstanding was related to error reporting for multi-line strings. When a string opened on line 1 but remained unclosed through line 3, my scanner reported the unterminated string error on `self.line` (line 3) because `self.line` had been incremented while consuming the newlines inside the string. Section 1.5 and Section 5.1 require unterminated string errors to be reported at the line where the string **opened**, not where the file ended.

I fixed this in `src/scanner.rs` by saving `let start_line = self.line;` at the beginning of `string()` and passing `start_line` to `self.error()` if `at_end()` is reached.

- **Commit where the code was incorrect:** cab9440
  `self.error(self.line, "String is never closed.");`
- **Commit where the fix was applied:** 9aeda50
  `self.error(start_line, "String is never closed.");`