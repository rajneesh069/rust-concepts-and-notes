# Unit Type `()`

In Rust, `()` is called the **unit type**. It’s similar to `void` in C or C++, but in Rust it’s a real type with exactly one possible value—also written `()`.

---

### Key points about `()`

1. **Singleton type**

   - There is exactly one value of type `()`, namely `()`.
   - You can think of it as “nothing” or “no meaningful value.”

2. **Returned by statements and side-effecting code**

   - Any statement or expression that you terminate with a semicolon becomes a statement that yields `()`.
   - For example:

     ```rust
     let x = {
         println!("Hello");
         42;       // the `;` turns `42` into a statement whose value is `()`
     };
     // Here `x` has type `()`, not `i32`.
     ```

3. **Default return type for functions that don’t explicitly return something**

   - If you write

     ```rust
     fn foo() {
         println!("hi");
     }
     ```

     it’s actually `fn foo() -> () { … }` under the hood. You rarely write `-> ()`, because it’s implied.

4. **Used in places where you only care about side effects**

   - Methods like `println!`, `.push()`, or `.write_all()` return `()`.
   - If you have `let r = println!("…");`, then `r` is `()`.

5. **Distinguished from the never type `!`**

   - `()` is a real, completed value.
   - The never type `!` (pronounced “never”) represents computations that never return (e.g., `panic!()`, `return`, or an infinite loop).

---

### Why does Rust use a unit type?

- **Uniformity:** In Rust, _everything_ is an expression and thus has a type. Even “do this then that” blocks need to report a type. If you write a block of code purely for side effects, it reports `()` so the type system stays consistent.
- **Pattern matching:** You can match on `()`, though you rarely need to:

  ```rust
  let val: () = ();
  match val {
      () => println!("got unit!"),
  }
  ```

---

#### Analogy for C++ programmers

- Think of `()` as C++’s `void`, but **typed**—it’s a real value you can bind to a variable (of type `()`), pass around (rarely), and match on.
- Where C++ functions that return nothing are declared `void foo()`, Rust functions that return nothing are simply `fn foo()` (implicitly `-> ()`).

---

In practice, you’ll encounter `()` whenever you call a function or method that “does something” but doesn’t produce a meaningful result: it quietly returns the unit value to satisfy the type system.
