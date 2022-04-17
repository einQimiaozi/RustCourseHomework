1.
```rust
/* Annotate struct with lifetime:
1. `r` and `s` must has different lifetimes
2. lifetime of `s` is bigger than that of 'r'
*/
struct DoubleRef<'a,T> {
    r: &'a T,
    s: &'a T
}
fn main() {
    println!("Success!")
}
```
