1.
```rust
use std::fmt;

/* Define the Wrapper type */
struct Wrapper(Vec<String>);

// Display is an external trait
impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    // Vec is an external type, so you cannot implement Display trait on Vec type
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

2.
```rust
/* Make it workd */
struct Meters(u32);

fn main() {
    let i: u32 = 2;
    assert_eq!(i.pow(2), 4);

    let n = Meters(i);
    // The `pow` method is defined on `u32` type, we can't directly call it
    assert_eq!(n.0.pow(2), 4);
}
```

3.
```rust
/* Make it work */
struct Years(i64);

struct Days(i64);

impl Years {
    pub fn to_days(&self) -> Days {
        Days(self.0 * 365)
    }
}


impl Days {
    pub fn to_years(&self) -> Years {
        Years(self.0 / 365)
    }
}

// an age verification function that checks age in years, must be given a value of type Years.
fn old_enough(age: &Years) -> bool {
    age.0 >= 18
}

fn main() {
    let age = Years(5);
    let age_days = age.to_days();
    println!("Old enough {}", old_enough(&age));
    println!("Old enough {}", old_enough(&age_days.to_years()));
}
```

4.
```rust
use std::ops::Add;
use std::fmt::{self, format};

struct Meters(u32);
impl fmt::Display for Meters {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "There are still {} meters left", self.0)
    }
}

impl Add for Meters {
    type Output = Self;

    fn add(self, other: Meters) -> Self {
        Self(self.0 + other.0)
    }
}
fn main() {
    let d = calculate_distance(Meters(10), Meters(20));
    assert_eq!(format!("{}",d), "There are still 30 meters left");
}

/* implement calculate_distance  */
fn calculate_distance(start:Meters,end:Meters) -> Meters {
    start+end
}
```

5.
```rust
enum VeryVerboseEnumOfThingsToDoWithNumbers {
    Add,
    Subtract,
}

/* Fill in the blank */
type Operations = VeryVerboseEnumOfThingsToDoWithNumbers;

fn main() {
    // We can refer to each variant via its alias, not its long and inconvenient
    // name.
    let x = Operations::Add;
}
```

6.
```rust
enum VeryVerboseEnumOfThingsToDoWithNumbers {
    Add,
    Subtract,
}

type Op = VeryVerboseEnumOfThingsToDoWithNumbers;

impl VeryVerboseEnumOfThingsToDoWithNumbers {
    fn run(&self, x: i32, y: i32) -> i32 {
        match self {
            Op::Add => x + y,
            Op::Subtract => x - y,
        }
    }
}

fn main(){}
```

7.
```rust
/* Make it work with const generics */
fn my_function<const n: usize>() -> [u32; n] {
    [123; n]
}

fn main() {
    let arr = my_function::<100>();
    println!("{:?}",arr);
}
```

8.
```rust
/* Make it work with slice references */
fn main() {
    let s: &str = "Hello there!";

    let arr: [u8; 3] = [1, 2, 3];
}
```

9. way1
```rust
/* Make it work in two ways */
use std::fmt::Display;
fn foobar(thing: impl Display) {}

fn main() {
}
```

9. way2
```rust
/* Make it work in two ways */
use std::fmt::Display;
fn foobar(thing: Box<dyn Display>) {}

fn main() {
}
```

9. way3
```rust
/* Make it work in two ways */
use std::fmt::Display;
fn foobar(thing: &dyn Display) {}

fn main() {
}
```
