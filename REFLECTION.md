# Reflection — CA2 Parser

## 1. Dangling Else

The dangling else is resolved in my `if_statement` function at `src/parser.rs:217–223`. At line 217, the parser recursively calls `self.statement()` to parse the then branch. If that statement is another `if` statement, the inner `if_statement` gets the opportunity to check for an `else` before the outer `if_statement` does. Lines 219–223 then check whether an `else` belongs to the current if statement. This means the nearest unmatched if receives the else. The grammar alone does not determine this association; the recursive structure of the parser does. For `if (false) if (true) print "a"; else print "b";`, the resulting tree is `(if false (if true (print "a") (print "b")))`. If the else belonged to the outer if instead, the tree would be `(if false (if true (print "a")) (print "b"))`.

## 2. Assignment

The assignment rule is implemented in `src/parser.rs:288–307`. At line 289, the parser first parses the left-hand side as an ordinary expression using `self.or()`. It then checks for an equals sign at line 291. This check cannot happen before parsing the left-hand side because the parser must first determine what kind of expression it has encountered. For example, `a = 2;` is valid because the left-hand side is a variable, while `1 = 2;` is invalid because the left-hand side is a literal. At line 295, the parser checks whether the parsed expression is an `Expr::Variable`. If it is, the parser constructs an assignment expression. Otherwise, line 303 reports the error. The required error message for `1 = 2;` is `Cannot assign to this expression.`

## 3. Error Recovery and Git History

My recovery function is `synchronise`, located at `src/parser.rs:542–572`, and it is called at `src/parser.rs:18` whenever parsing a declaration fails. It stops after a semicolon, at the end of the file, or before the statement-starting tokens `fun`, `var`, `if`, `while`, `print`, and `return`. These tokens provide recovery points that allow the parser to continue with another declaration or statement. One relevant test is `tests/phase-2/invalid/expect_semicolon_after_value.kobo`. It contains `print 1` followed by `print 2;`. With recovery, the parser reports `[line 2] Error at 'print': Expect ';' after value.` If the call to `synchronise()` at line 18 is removed, the parser remains at the same `print` token and repeatedly encounters the same error instead of advancing, causing the test to time out.

**Git history evidence:** My current repository history does not contain two separate parser commits showing the incorrect and corrected versions of the recovery code. I have not invented commit hashes or historical lines to represent commits that do not exist.
