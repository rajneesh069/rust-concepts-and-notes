# A very important concept in Rust: **NLL (Non-Lexical Lifetimes)** and how the borrow checker works in practice, rather than just the strict rule often quoted.

The fundamental rule is:
**"At any given time, you can either have one mutable reference OR any number of immutable references, but not both."**

However, the Rust compiler (specifically, its borrow checker with NLL) is smart enough to understand the _scope_ of a borrow more precisely than just the lexical block. It understands that a mutable borrow only lasts for the duration that it's _actually used_, not necessarily until the end of the block.

Let's trace the code's borrows:

```rust
fn fn1(str1: &mut String) {
    println!("{}", str1); // `str1` (mutable borrow) is used here
    str1.push('m');       // `str1` (mutable borrow) is used here
} // `str1` (mutable borrow) ends here. The borrow is no longer active.

fn fn2(str: &mut String) {
    str.pop();            // `str` (mutable borrow) is used here
    println!("{}", str)   // `str` (mutable borrow) is used here
} // `str` (mutable borrow) ends here. The borrow is no longer active.

fn main() {
    let mut str: String = String::from("Hello BRO!"); // `str` owned by main

    // 1. Call to fn1:
    // A mutable borrow (&mut str) is created and passed to fn1.
    // This borrow is active *during the execution of fn1*.
    fn1(&mut str); // `str` is mutably borrowed.
    // Immediately after `fn1` returns, the borrow created by `&mut str` ends.
    // Rust knows `str` is no longer mutably borrowed at this point.

    // 2. Creation of s2:
    // A new mutable borrow (&mut str) is created and assigned to `s2`.
    let s2 = &mut str; // `str` is now mutably borrowed by `s2`.
    // This borrow is active until the last use of `s2`.
    s2.push_str(" Mishra"); // `s2` (mutable borrow) is used here.
    // The borrow of `s2` effectively ends here because `s2` is not used again.

    // 3. Call to fn2:
    // A *new* mutable borrow (&mut str) is created and passed to fn2.
    // This is allowed because the previous borrow (`s2`) has ended.
    fn2(&mut str); // `str` is mutably borrowed.
    // Immediately after `fn2` returns, this borrow ends.
} // `str` (the String itself) goes out of scope here.
```

### Why it's Not Complaining (NLL in Action):

The key is that the compiler determines the _actual extent_ of the borrow, not just the lexical scope.

- When `fn1(&mut str)` is called, a temporary mutable reference is created, used inside `fn1`, and then _dropped_ as soon as `fn1` returns. The borrow ends.
- When `let s2 = &mut str;` is done, `s2` holds a mutable reference. This reference is used on the next line `s2.push_str(" Mishra");`. Since `s2` is not used again _after this point_, the borrow associated with `s2` effectively ends.
- Then, a _new_ mutable reference is created for `fn2(&mut str)`. This is completely independent of the previous borrows, which have already concluded.

If you were to try to use `s2` _after_ calling `fn2(&mut str)`, then you would get a compile error, because that would be two active mutable borrows (`s2` and the temporary one for `fn2`) at the same time:

```rust
fn main() {
    let mut str: String = String::from("Hello BRO!");
    fn1(&mut str);

    let s2 = &mut str;
    s2.push_str(" Mishra");

    fn2(&mut str); // ERROR: This line will start complaining, because s2 is still alive below, if it were not used again, the program won't complain!

    s2.push_str(" Another Error"); // -> line This will cause error in the program.
    /* Because `s2` is still "alive" here,
     and you just created another mutable borrow for `fn2`.
    * /

}
```

![image](/assets/2.png)
This behavior (Non-Lexical Lifetimes) was a significant improvement to the Rust compiler in Rust 2018. It makes the borrow checker more ergonomic and less restrictive, allowing many common patterns that would previously have been compile errors. It understands that a borrow doesn't necessarily last until the end of the enclosing block, but only until the last point where the reference is actually used.
