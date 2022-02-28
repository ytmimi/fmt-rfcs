# `asm!` and `global_asm!` Style Guide

## Delimiters

Prefer parentheses over brackets and braces.

```rust
asm!();
```

## Single-Template Assembly Code

`asm!` with a single assembly template can be written on a single line if the entire expression from `asm!(` to `);` plus indentation fits in the line width.

### No Arguments

```rust
asm!("nop");
```

### With Arguments

```rust
asm!("mov {}, 5", out(reg) x);
```

## Long `asm!` Expressions

Any `asm!` that needs breaking across multiple lines should always put each item on its own line, aligned one indentation level past the `asm!`, with a trailing comma after each.

```rust
asm!(
    "add {0}, {1}",
    inlateout(reg) in_out_var_a,
    in(reg) in_var_b,
    options(pure, nomem, nostack),
);
```

## Multi-Template Assembly Code

Any `asm!` with multiple assembly templates no matter how short, must place each template and any additional arguments on their own line, aligned one indentation level past the `asm!`, with a trailing comma after each template and argument.

  ```rust
  asm!(
      "instruction 1",
      "instruction 2",
      ...,
  );
  ```

## Assembly Labels

Assembly templates that start with a period (`.`) or end with a colon (`:`) are considered assembly labels.
Each non-label assembly template under a label should be block indented one level further than the label.
Use the opening `"` of each template as the base for any indentation within the assembly code, and add indentation by prepending it to the assembly template.
That way, Rust formatting can keep the strings aligned and your assembly code will remain aligned within them:

```rust
asm!(
    "1:",
    "    instruction 1",
    "    instruction 2",
    "2:",
    "    instruction 3",
);
```

## Assembly Operands With Registers

Never place any space or line breaks inside of `in(reg)`, `out(reg)`, `inout(reg)`, `in("regname")`, `out("regname")`, or similar; always treat them as a single atomic unit.

If an `inout` or `inlateout` operand with both an `in` and `out` expression are too long to fit on one line, break before (never after) the `=>`, and indent the `=>` and `out` expression one level further. keep the opening of the `in` expression on the same line as the operand. Format the `in` expression with the same indentation as the operand, and format the `out` expression with the same indentation as the `=>`. Format both the `in` and `out` expression over multiple lines if necessary.

```rust
asm!(
    "instruction {}",
    inout(reg) very_long_in_expression
        => very_long_out_expression,
)

asm!(
    "instruction {}",
    inout(reg) function_name()
        .method_name(method_arg)
        .further_chained_method()
        => very_long_out_expression,
);

asm!(
    "instruction {}",
    inout(reg) function_name(a, b, c)
        => very_long_out_expression,
);

asm!(
    "instruction {}",
    inout(reg) long_function_name(
        long_function_argument_expression
    )
        => very_long_out_expression,
);

```

* If an `in(reg)`, `out(reg)`, `lateout(reg)` followed by a single expression is too long, break between the `)` and the expression and indent one level, then format from there; however, if the expression can be broken internally, follow same the rules for when to leave the head of the expression on the same line as the `in(reg)` or similar as if `in(reg)` were the opener of a function:
```rust
asm!(
    "instruction {}",
    in(reg)
        extremely_long_unbreakable_expression,
);

asm!(
    "instruction {}",
    in(reg) function_name()
        .method_name(method_arg)
        .further_chained_method(),
);

asm!(
    "instruction {}",
    out(reg) long_function_name(
        long_function_argument_expression,
    ),
);
  ```

## Assembly Operands Without Registers

Always place the keyword `sym` or `const` on the same line as the expression that follows it. Format the expression over multiple lines if necessary.

### Sym

```rust
asm!(
    "instruction {}",
    sym some_long_symbol_name,
);
```

### Const

```rust
asm!(
    "instruction {}",
    const extremely_long_unbreakable_expression,
);

asm!(
    "instruction {}",
    const function_name()
        .method_name(method_arg)
        .further_chained_method(),
);

asm!(
    "instruction {}",
    const long_function_name(
        long_function_argument_expression
    ),
);
```

## Assembly Operands With Named Arguments

* For named arguments like `name i= n(reg) expression`, line-break it as you would an assignment steatment with the same indentiaton, treating the `in(reg) expression` as the right-hand side of the assignment, and following all the same rules as above.


## Clobber ABIs and Options

Both clobber's and options should be formatted like nested function calls.

* `clobber_abi(...)` goes on one line if it fits; otherwise, format it as though it were a nested function call to a function named `clobber_abi`.
* `options(...)` goes on one line if it fits; otherwise, format it as though it were a nested function call to a function named `options`.