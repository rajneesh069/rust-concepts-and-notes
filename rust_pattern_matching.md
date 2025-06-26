# Pattern Matching in Rust: What You Need to Know

Pattern matching is a core feature in Rust that allows you to destructure complex types (like enums, structs, tuples) and extract their values, often in conjunction with conditional logic.
Here are the primary constructs where patterns are used:

1. match Expressions: The most exhaustive and powerful form of pattern matching.
2. if let Expressions: A more concise way to handle a single successful match.
3. while let Loops: As seen in your example, for repeatedly handling successful matches.
4. for Loops: Iterating over collections.
5. let Statements: For destructuring and binding variables.
6. Function Parameters: For destructuring arguments directly.
   Let's look at each, focusing on common patterns.

7. match Expressions (The Exhaustive Match)

match allows you to compare a value against a series of patterns and execute code based on the first matching pattern. It must be exhaustive, meaning all possible cases for the value must be covered.

```rust
    enum Message {
        Quit,
        Move { x: i32, y: i32 },
        Write(String),
        ChangeColor(i32, i32, i32),
    }

    fn handle_message(msg: Message) {
        match msg {
            // Simple variant without data
            Message::Quit => {
                println!("The Quit variant has no data to destructure.");
            }
            // Struct-like variant with named fields
            Message::Move { x, y } => {
                println!("Move to x: {}, y: {}", x, y);
            }
            // Tuple-like variant with a single value
            Message::Write(text) => {
                println!("Text message: {}", text);
            }
            // Tuple-like variant with multiple values
            Message::ChangeColor(r, g, b) => {
                println!("Change color to R:{}, G:{}, B:{}", r, g, b);
            }
        }
    }

    // Common patterns in match:
    fn check_number(n: i32) {
        match n {
            1 => println!("One!"),
            2 | 3 => println!("Two or Three!"), // Multiple patterns
            4..=10 => println!("Between 4 and 10 (inclusive)!"), // Ranges
            _ => println!("Something else!"), // Catch-all pattern (like default)
        }
    }

    // Matching on Options/Results
    fn process_optional_value(opt: Option<i32>) {
        match opt {
            Some(value) => println!("Got a value: {}", value),
            None => println!("No value here!"),
        }
    }
```

2. `if let` Expressions (Single Case Matching)

if let is syntactic sugar for a match expression that only cares about one specific pattern. It's used when you want to do something if a value matches a pattern, and optionally do something else if it doesn't (using else).

```rust
    let config_max = Some(3u8); // Option<u8>

    // if let is like saying: "if config_max matches the Some(max) pattern..."
    if let Some(max) = config_max {
        println!("The maximum is configured to be {}", max);
    } else {
        println!("No maximum configured.");
    }

    // Equivalent to:
    // match config_max {
    //     Some(max) => println!("The maximum is configured to be {}", max),
    //     _ => println!("No maximum configured."),
    // }
```

3. `while let` Loops (Repeated Single Case Matching)

This is what was boggling your mind! while let is essentially if let inside a while loop. It continues looping as long as the value being checked matches the specified pattern.

```rust
    let mut stack = Vec::new();
    stack.push(1);
    stack.push(2);
    stack.push(3);

    // The loop continues as long as stack.pop() returns Some(value)
    while let Some(top) = stack.pop() { // `pop()` returns Option<T>
        println!("Popped: {}", top);
    }
    // Output:
    // Popped: 3
    // Popped: 2
    // Popped: 1
```

4. `for` Loops (Iterating over Patterns)

The for loop also uses patterns implicitly.

```rust
    let v = vec!['a', 'b', 'c'];

    // (val) is a simple binding pattern. It matches each element yielded by the iterator.
    for val in v {
        println!("{}", val);
    }

    // You can also destructure directly if the iterator yields a tuple, for example:
    let pairs = vec![(1, 'a'), (2, 'b'), (3, 'c')];
    for (num, char) in pairs { // (num, char) is a tuple pattern
        println!("Number: {}, Char: {}", num, char);
    }
```

5. `let` Statements (Destructuring)

let statements are not just for simple variable binding; they use patterns to destructure values and bind parts of them to new variables.

```rust
    let (x, y, z) = (1, 2, 3); // Tuple pattern
    println!("x: {}, y: {}, z: {}", x, y, z); // Output: x: 1, y: 2, z: 3

    struct Point {
        x: i32,
        y: i32,
    }
    let p = Point { x: 10, y: 20 };
    let Point { x: my_x, y: my_y } = p; // Struct pattern (can use shorthand {x, y} if field names match var names)
    println!("My X: {}, My Y: {}", my_x, my_y); // Output: My X: 10, My Y: 20

    // You can also use _ for ignored parts, or .. for ignored remaining parts
    let (a, _, c) = (100, 200, 300); // Ignore middle element
    println!("a: {}, c: {}", a, c); // Output: a: 100, c: 300

    let numbers = [1, 2, 3, 4, 5];
    let [first, second, ..] = numbers; // Array/slice pattern with rest
    println!("First: {}, Second: {}", first, second); // Output: First: 1, Second: 2
```

6. Function Parameters (Destructuring Arguments)

You can use patterns directly in function parameters.

```rust
    // Tuple pattern in function signature
    fn print_coordinates((x, y): (i32, i32)) {
        println!("Coordinates: ({}, {})", x, y);
    }

    // Struct pattern in function signature
    struct User {
        id: u32,
        name: String,
    }
    fn display_user_id(User { id, .. }: User) { // Only bind 'id', ignore others
        println!("User ID: {}", id);
    }

    fn main() {
        print_coordinates((10, 20));
        let u = User { id: 123, name: String::from("Alice") };
        display_user_id(u); // Ownership of 'u' is moved into the function here
    }
```

---

## How `while let Some(val) = iter.next()` is Evaluated as a Boolean Condition

This is the key insight for `if let` and `while let`.
In Rust, if let and while let don't literally evaluate to a boolean true or false in the way if (x > 5) does. Instead, they operate on the success or failure of a pattern match.
Here's the mental model for while let Some(val) = iter.next():

1.  Evaluate the expression on the right-hand side: iter.next() is called. This returns an Option<&i32> (either Some(&value) or None).

2.  Attempt to match the pattern: The result of iter.next() is then compared against the pattern Some(val).

3.  Conditional Execution based on Match Success:

    If the expression successfully matches the Some(val) pattern:

        	- The loop's condition is considered "true".
        	- The inner value (&value) is extracted and bound to the variable val (creating val within the loop's scope).
        	- The code inside the while loop's body ({ ... }) is executed.
        	- Then, the loop repeats, going back to step 1.
        -

    If the expression does not match the Some(val) pattern:

        	- This happens when iter.next() returns None.
        	- The loop's condition is considered "false".
        	- The loop terminates immediately, and the program continues after the while block.

    It's not a boolean true/false in the traditional sense, but a branching mechanism based on whether a pattern successfully applies.

---

## Is this "Let in a While Statement" Possible in TS/C++?

### TypeScript/JavaScript:

- No direct syntactic equivalent to while let's pattern matching and binding.

- You can perform assignments within while conditions, which is the closest you get to the variable binding part:

```ts
// TypeScript/JavaScript (simulated Optional/Iterator)
class JsIterator<T> {
  /* ... same as before ... */
}

let iter = new JsIterator([1, 2, 3]);
let val: number | undefined; // Declare variable outside the loop
while ((val = iter.next()) !== undefined) {
  // Assign and check
  console.log(val); // 'val' is now guaranteed to be a number inside the loop
}
```

Here, (val = iter.next()) performs the assignment, and !== undefined performs the boolean check. It's functional, but less integrated than Rust's pattern matching.

### C++:

- C++17 and later offers if (init; condition) and while (init; condition) statements which allow initialization within the condition. This is the closest C++ gets syntactically to something like this, especially when combined with std::optional.

```cpp
      #include <iostream>
      #include <vector>
      #include <optional>

      // Dummy iterator that returns std::optional<int>
      class CppIter {
      public:
          CppIter(const std::vector<int>& data) : data_(data), index_(0) {}
          std::optional<int> next() {
              if (index_ < data_.size()) {
                  return data_[index_++];
              }
              return std::nullopt;
          }
      private:
          const std::vector<int>& data_;
          size_t index_;
      };

      int main_cpp() {
          std::vector<int> numbers = {10, 20, 30};
          CppIter iter(numbers);

          // C++17 'if (init; condition)' (similar for while)
          if (std::optional<int> val = iter.next(); val.has_value()) {
              std::cout << "Got value (if): " << val.value() << std::endl;
          }

          // C++20 'while (init)' where init is a declaration that can convert to bool
          // Or older C++ versions where assignment is used in condition, like TS
          iter = CppIter(numbers); // Reset iterator

          while (std::optional<int> val_opt = iter.next()) { // 'val_opt' is created in loop scope
              // 'val_opt' implicitly converts to bool (true if has_value(), false if std::nullopt)
              std::cout << "Got value (while): " << val_opt.value() << std::endl;
          }

          return 0;
      }
```

The while (std::optional<int> val_opt = iter.next()) in C++20 is quite close! The val_opt variable is scoped to the loop body, and std::optional has an implicit conversion to bool (which is true if it contains a value, false otherwise). So, it's very similar in functionality to Rust's while let Some(val) = ....

**Key Difference: Rust's `while let` leverages its built-in pattern matching directly for arbitrary enums (not just Option or Result which are like std::optional), which is more general and deeply integrated into the language's type system. C++'s solution is more specific to std::optional or types that define a boolean conversion operator.**
