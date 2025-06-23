# Function parameters and their working in C, Rust and TS

It gets to the heart of how different programming languages manage data and execution! The answer isn't uniform across Rust, C, and TypeScript, and hinges on concepts like **pass-by-value**, **pass-by-reference**, and **ownership/borrowing**.

---

### C: Primarily Pass-by-Value (with "pointers" for pass-by-reference effect)

In C, the fundamental mechanism for passing arguments to functions is **pass-by-value**.

- **Temporary Variables:** Yes, when you call a function in C, **a new temporary variable is created on the function's stack frame for each parameter**. This variable is initialized with a _copy_ of the value of the argument passed.
- **Deletion:** These temporary variables are part of the function's stack frame. When the function finishes execution, its stack frame is popped, and these temporary variables are effectively "deleted" (their memory is reclaimed).
- **Impact:**
  - Changes made to the parameter _inside_ the function do **not** affect the original argument in the calling scope, because you're working on a copy.
  - To achieve a "pass-by-reference" effect (where changes _do_ affect the original), you pass the **address** of the variable (using a pointer `*`). The pointer itself is still passed by value (a copy of the address), but because you're dereferencing it, you're modifying the original memory location.

**Example (C):**

```c
#include <stdio.h>

void modify_value(int x) { // x is a copy of 'a'
    printf("Inside modify_value (before): x = %d\n", x);
    x = 100; // Changes only the copy 'x'
    printf("Inside modify_value (after): x = %d\n", x);
} // x is destroyed here

void modify_through_pointer(int *ptr) { // ptr is a copy of the address of 'b'
    printf("Inside modify_through_pointer (before): *ptr = %d\n", *ptr);
    *ptr = 200; // Changes the value at the address ptr points to (original 'b')
    printf("Inside modify_through_pointer (after): *ptr = %d\n", *ptr);
} // ptr is destroyed here

int main() {
    int a = 10;
    printf("In main (before modify_value): a = %d\n", a);
    modify_value(a);
    printf("In main (after modify_value): a = %d\n", a); // a is still 10

    int b = 20;
    printf("\nIn main (before modify_through_pointer): b = %d\n", b);
    modify_through_pointer(&b); // Pass the address of b
    printf("In main (after modify_through_pointer): b = %d\n", b); // b is now 200

    return 0;
}
```

---

### Rust: Ownership, Borrowing, and Pass-by-Value/Reference

Rust explicitly manages memory and ownership. Function parameters are a direct reflection of Rust's ownership system.

- **Pass-by-Value (Ownership Transfer):** If you pass a type like `String`, `Vec<T>`, or a custom `struct` that _owns_ its data, the parameter takes **ownership** of the argument.
  - **Temporary Variables:** A conceptual "new variable" is created, but it's more accurate to say the _ownership_ is transferred. The original variable in the caller's scope becomes invalid after the call (it's "moved").
  - **Deletion:** When the function finishes, the parameter variable (which now owns the data) is dropped, and the memory it owned is deallocated.
- **Pass-by-Reference (Borrowing):** This is the most common way to pass complex data without transferring ownership.
  - You pass a **reference** (`&T` for immutable, `&mut T` for mutable). The reference itself is a small, fixed-size value (like a pointer in C), which is effectively passed by value.
  - **Temporary Variables:** A new variable holding the _reference_ is created on the stack frame.
  - **Deletion:** This temporary reference variable is deleted when the function returns. The original data that the reference pointed to is _not_ dropped by the function; it remains owned by the caller.
- **Pass-by-Copy (for `Copy` types):** For types that implement the `Copy` trait (like primitives: `i32`, `bool`, `char`, fixed-size arrays), they are passed by value, and a true bitwise copy is made. The original variable remains valid. This is similar to C's pass-by-value for primitives.

**Example (Rust):**

```rust
fn take_ownership(s: String) { // s takes ownership of the String
    println!("Inside take_ownership: {}", s);
} // s is dropped here, and the String data is deallocated

fn borrow_immutable(s: &String) { // s is an immutable reference (borrow)
    println!("Inside borrow_immutable: {}", s);
} // s (the reference) is dropped here, original String remains

fn borrow_mutable(s: &mut String) { // s is a mutable reference (borrow)
    println!("Inside borrow_mutable (before): {}", s);
    s.push_str(" world!"); // Modify the original String
    println!("Inside borrow_mutable (after): {}", s);
} // s (the reference) is dropped here, original String remains

fn copy_primitive(mut x: i32) { // x is a copy of 'a' (i32 implements Copy)
    println!("Inside copy_primitive (before): x = {}", x);
    x = 100; // Changes only the copy 'x'
    println!("Inside copy_primitive (after): x = {}", x);
} // x is dropped here

fn main() {
    let my_string = String::from("Hello");
    take_ownership(my_string); // my_string is MOVED here, no longer usable in main after this
    // println!("{}", my_string); // ERROR: use of moved value: `my_string`

    let another_string = String::from("Rust");
    borrow_immutable(&another_string); // Pass an immutable reference
    println!("In main (after immutable borrow): {}", another_string); // another_string is still valid

    let mut third_string = String::from("Coding");
    borrow_mutable(&mut third_string); // Pass a mutable reference
    println!("In main (after mutable borrow): {}", third_string); // third_string is modified

    let a = 10;
    println!("In main (before copy_primitive): a = {}", a);
    copy_primitive(a);
    println!("In main (after copy_primitive): a = {}", a); // a is still 10
}
```

---

### TypeScript (JavaScript): Mostly Pass-by-Value for Primitives, Pass-by-Reference for Objects

TypeScript (and JavaScript, which it compiles to) has a more nuanced approach, often described as "pass-by-value for primitives, pass-by-sharing for objects."

- **Primitives (numbers, strings, booleans, `null`, `undefined`, `symbol`, `bigint`):**

  - **Temporary Variables:** Yes, a new temporary variable is created in the function's scope, initialized with a _copy_ of the primitive value.
  - **Deletion:** These variables are cleaned up when the function returns (garbage collected).
  - **Impact:** Changes to primitive parameters _inside_ the function do not affect the original argument.

- **Objects (including arrays, functions, custom objects):**
  - **Pass-by-Sharing (or "Pass-by-Value of Reference"):** A new temporary variable is created on the stack frame for the parameter. This variable receives a _copy of the reference_ (the memory address) to the original object. Both the original variable and the parameter variable now point to the _same object_ in memory.
  - **Temporary Variables:** A new variable holding the _reference_ is created.
  - **Deletion:** This temporary reference variable is garbage collected when the function returns. The original object in memory is _not_ deleted unless there are no more references to it.
  - **Impact:**
    - If you **modify the properties** of the object (e.g., `obj.prop = 5`), the original object _is_ affected because both variables point to the same object.
    - If you **reassign the parameter variable** to a completely _new object_ (e.g., `obj = { new: 'data' }`), this change _does not_ affect the original variable in the caller, because you've only made the parameter variable point to a new object, leaving the original variable pointing to the old one.

**Example (TypeScript):**

```typescript
function modifyPrimitive(x: number) {
  // x is a copy of 'a'
  console.log("Inside modifyPrimitive (before): x =", x);
  x = 100; // Changes only the copy 'x'
  console.log("Inside modifyPrimitive (after): x =", x);
}

function modifyObject(obj: { value: number }) {
  // obj is a copy of the reference to 'b'
  console.log("Inside modifyObject (before): obj.value =", obj.value);
  obj.value = 200; // Modifies the original object 'b' points to
  console.log("Inside modifyObject (after): obj.value =", obj.value);
}

function reassignObject(obj: { value: number }) {
  // obj is a copy of the reference to 'c'
  console.log("Inside reassignObject (before): obj =", obj);
  obj = { value: 300 }; // Reassigns 'obj' parameter to a *new* object.
  // This does NOT affect the original 'c'.
  console.log("Inside reassignObject (after): obj =", obj);
}

function main() {
  let a = 10;
  console.log("In main (before modifyPrimitive): a =", a);
  modifyPrimitive(a);
  console.log("In main (after modifyPrimitive): a =", a); // a is still 10

  let b = { value: 20 };
  console.log("\nIn main (before modifyObject): b.value =", b.value);
  modifyObject(b);
  console.log("In main (after modifyObject): b.value =", b.value); // b.value is now 200

  let c = { value: 30 };
  console.log("\nIn main (before reassignObject): c.value =", c.value);
  reassignObject(c);
  console.log("In main (after reassignObject): c.value =", c.value); // c.value is still 30 (not 300)
}

main();
```

---

### Conclusion

The understanding that "temporary variables are created and deleted" is a good starting point, but the nuances of how values (or references to values) are copied or moved, and how memory is managed, vary significantly:

- **C:** Always pass-by-value. Pointers are used to mimic pass-by-reference. Stack frames handle parameter memory.
- **Rust:** Explicit ownership and borrowing. Values can be moved (transfer ownership) or borrowed (pass references). Types implementing `Copy` are truly copied. Stack frames for references/copies, heap for owned data.
- **TypeScript (JS):** Primitives are copied. Objects are "passed by sharing" (a copy of the reference). Garbage collection handles memory.
