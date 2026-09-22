# Reflection

## 1. Fractional Part

The decision to treat a dot as the beginning of a fractional part is at `src/scanner.rs:153`. My scanner checks that the current character is `.` and that the next character is a digit before including the dot in the number. This follows section 1.4, which says a number is one or more digits optionally followed by a dot and one or more digits. For the input `5.`, my scanner first scans `5` as a `NUMBER`. Since there is no digit after the dot, the condition at line 153 is false, so the dot is not included in the number. The scanner then processes `.` separately and reports `Character is not part of any token.` This is required because Kobo has no `DOT` token and a fractional part must contain at least one digit after the decimal point.

## 2. Line Counter and EOF

I change the line counter at `src/scanner.rs:113` when the scanner encounters a normal newline, and at `src/scanner.rs:133` when a newline occurs inside a multiline string. These are the two places where `self.line` is incremented. For a file that contains real tokens and then ends with two blank lines, the EOF token still carries the line of the last real token. This is handled at `src/scanner.rs:46`, where EOF uses `last_token_line` rather than the current value of `self.line`. Section 6.1 requires this because trailing whitespace should not change the reported EOF line. Although the two blank lines increase the scanner's current line counter, they do not produce tokens, so `last_token_line` remains unchanged. For an empty or comment-only file, it remains initialized to line 1.

## 3. Failed Test and Fix

One test I failed was `invalid/unterminated_string.kobo`. In commit `afd6eff`, my `string()` function was still unimplemented and contained `todo!("string")`, so the scanner could not correctly handle an unterminated string. I had not yet implemented the requirement that the error must be reported using the line where the string started and with the exact message specified in section 5.1. I fixed this in commit `4cb3a42`. The relevant final line is `src/scanner.rs:139`: `self.error(start_line, "String is never closed.");`. Therefore, the line in the failing commit was `todo!("string")`, while the corrected implementation reports the required error using `start_line`. This taught me that handling an unterminated string requires remembering its opening line rather than simply using the scanner's current line when the end of the source is reached.
