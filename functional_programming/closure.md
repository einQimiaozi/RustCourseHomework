1.
```rust
/* Make it work with least changing */
fn main() {
    let color = &String::from("green");

    let print = move || println!("`color`: {}", *color);

    print();
    //print();

    // `color` can be borrowed immutably again, because the closure only holds
    // an immutable reference to `color`. 
    let _reborrow = &color;

    println!("{}",*color);
}
```
