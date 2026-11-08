# REFLECTION

## 1. The Dangling Else

The dangling else problem occurs when an `if` statement is placed inside another `if` statement and it is unclear which `if` the `else` belongs to. According to section 3.1 of the language specification, an `else` belongs to the nearest unmatched `if`.

In my implementation, this is handled through recursion in `if_statement()`. At lines 175–176, the parser first parses the then-branch by calling `self.statement()`. If that branch contains another `if`, the inner `if_statement()` gets to check for and consume the `else` before the outer function continues. This is why the recursion decides which `if` receives the `else`, rather than the grammar explicitly deciding it.

For `if (false) if (true) print "a"; else print "b";`, the `else` belongs to the inner `if`.

**Code reference:** `src/parser.rs:171–186`

**Actual AST output:** (if false (if true (print "a") (print "b")))

If the `else` belonged to the outer `if`, the inner `if` would have no else-branch, while the outer `if` would contain the else-branch.

## 2. Assignment Parsing

Assignment is not LL(1) because when the parser first reads an expression such as `a`, it cannot immediately know whether it is just a variable expression or the beginning of an assignment. The left side must therefore be parsed as an ordinary expression before the parser can decide what it represents.

In my implementation, the `assignment()` function first parses the left-hand side using `self.or()`. It then checks whether an equals sign follows. At line 230, the parser checks whether the expression is an `Expr::Variable`. If it is, the parser creates an assignment expression. If it is not, the parser reports an error.

For example, `1 = 2;` is invalid because the number `1` is not a variable that can receive a new value. The error message is `Cannot assign to this expression.`

This approach allows valid variable assignments while rejecting invalid assignment targets.

**Code reference:** `src/parser.rs:225–235`

## 3. Error Recovery and Git History

My parser uses `synchronise()` to recover from syntax errors. It stops after a semicolon or when it reaches one of these tokens: `Fun`, `Var`, `If`, `While`, `Print`, and `Return`. These tokens are useful because they mark the beginning of a new declaration or statement. The parser advances before checking, which ensures that recovery makes progress.

An example is `tests/phase-2/invalid/expect_semicolon_after_value.kobo`. It contains `print 1` followed by `print 2;`. The expected error is `[line 2] Error at 'print': Expect ';' after value.` Without the call to `synchronise()` at line 18, the parser would continue and also include the second statement, `(print 2)`, in the AST.

My Git history shows the development of this function:

- **Incorrect commit:** `9fddcb210337181c218742163b6b636e79c2bdaf`. The recovery code only checked for a semicolon:
  ```rust
  self.depth = 0;
  while !self.at_end() {
      self.current += 1;
      if self.previous().kind == TokenType::Semicolon {
          return;
      }
  }
  ```

- **Fixed commit:** `de55b35148d63f94fc5d12b81826c5570fe284a4`. I added checks for statement-starting keywords and improved token advancement.

**Code reference:** `src/parser.rs:408–430`