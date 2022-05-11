```rust
macro_rules! myprint {
    ($x: expr) => {
        println!("{}", $x)
    };
    ($x: expr, $($y: expr),+) => {
        if $x % 2 == 1 {
            println!("{}", $x);
            myprint!($($y),+);
        }else {
            myprint!($($y),+);
            println!("{}", $x);
        }
    };
}

fn main() {
    // we want: 1, 3, 5, 6, 4, 2
    myprint!(1, 2, 3, 4, 5, 6);
    // want: 1, 3, 5, 4, 2
    myprint!(1, 2, 3, 4, 5);
}
```
