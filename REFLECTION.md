# Reflection

## 1. Fractional Part

The decision to treat a dot as the beginning of a fractional part is at `src/scanner.rs:153`. My scanner checks that the current character is `.` and that the next character is a digit before including the dot in the number. This follows section 1.4, which says a number is one or more digits optionally followed by a dot and one or more digits. For the input `5.`, my scanner first scans `5` as a `NUMBER`. Since there is no digit after the dot, the condition at line 153 is false, so the dot is not included in the number. The scanner then processes `.` separately and reports `Character is not part of any token.` This is required because Kobo has no `DOT` token and a fractional part must contain at least one digit after the decimal point.

## 2. Line Counter and EOF

I change the line counter at `src/scanner.rs:113` when the scanner encounters a normal newline, and at `src/scanner.rs:133` when a newline occurs inside a multiline string. These are the two places where `self.line` is incremented. For a file that contains real tokens and then ends with two blank lines, the EOF token still carries the line of the last real token. This is handled at `src/scanner.rs:46`, where EOF uses `last_token_line` rather than the current value of `self.line`. Section 6.1 requires this because trailing whitespace should not change the reported EOF line. Although the two blank lines increase the scanner's current line counter, they do not produce tokens, so `last_token_line` remains unchanged. For an empty or comment-only file, it remains initialized to line 1.

## 3. Failed Test and Fix

One test I failed during development was `invalid/unterminated_string.kobo`. The starter version in commit `afd6eff` still had `todo!("string")`, so it could not correctly scan or report an unterminated string. While implementing it, I also initially used the wrong wording for the error message, writing `Unterminated string.` instead of the exact message required by section 5.1. Testing showed me that scanner error text is part of the required output and must match the specification exactly. In commit `4cb3a42`, the implementation at `src/scanner.rs:139` became `self.error(start_line, "String is never closed.");`. The earlier committed line was `todo!("string")`, while the fixed committed implementation reports the required message using `start_line`. I learned that I must check both scanner behaviour and the exact specified error output, including which line number the error uses.
