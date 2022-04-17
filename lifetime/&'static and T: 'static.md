1. way1
```rust
/* Fill in the blank in two ways */
fn main() {
    let v = "hello";
    need_static(v);

    println!("Success!")
}

fn need_static(r : &'static str) {
    assert_eq!(r, "hello");
}
```

1. way2
```rust
/* Fill in the blank in two ways */
fn main() {
    let v:&'static str = "hello";
    need_static(v);

    println!("Success!")
}

fn need_static(r : &'static str) {
    assert_eq!(r, "hello");
}
```

2.
```rust
#[derive(Debug)]
struct Config {
    a: String,
    b: String,
}
static mut config: Option<&mut Config> = None;

/* Make it work without changing the function signatures of `init`*/
// 解释一下，这里需要让config对象的生命周期延长至static，因为你标注了static，但这个只是告诉编译器config实例的生命周期是staitc，不代表config实例的生命周期真的是static
// Box::leak就是用来泄漏对象内存的，让对象通过内存泄漏的方式延长其生命周期至staitc
fn init() -> Option<&'static mut Config> {
    let res = Box::new(Config {
        a: "A".to_string(),
        b: "B".to_string(),
    });
    Some(Box::leak(res))
}


fn main() {
    unsafe {
        config = init();

        println!("{:?}",config)
    }
}
```

3.
```rust
fn main() {
    // Make a `string` literal and print it:
    let static_string = "I'm in read-only memory";
    println!("static_string: {}", static_string);

    // When `static_string` goes out of scope, the reference
    // can no longer be used, but the data remains in the binary.

    println!("static_string reference remains alive: {}", static_string);
}
```

5.
```rust
/* Make it work */
use std::fmt::Debug;

fn print_it<T: Debug + 'static>( input: T) {
    println!( "'static value passed in is: {:?}", input );
}

fn print_it1( input: impl Debug + 'static ) {
    println!( "'static value passed in is: {:?}", input );
}


fn print_it2<T: Debug + 'static>( input: &T) {
    println!( "'static value passed in is: {:?}", input );
}

fn main() {
    // i is owned and contains no references, thus it's 'static:
    let i:i32 = 5;
    print_it(i);

    // oops, &i only has the lifetime defined by the scope of
    // main(), so it's not 'static:
    print_it(i);

    print_it1(i);

    // but this one WORKS !
    print_it2(&i);
}
```

6.
```rust
use std::fmt::Display;

fn main() {
  let mut string = "First".to_owned();

  string.push_str(string.to_uppercase().as_str());
  print_a(&string);
  print_b(&string);
  print_c(&string); // Compilation error
  print_d(&string); // Compilation error
  print_e(&string);
  print_f(&string);
  print_g(&string); // Compilation error
}

fn print_a<T: Display + 'static>(t: &T) {
  println!("{}", t);
}

fn print_b<T>(t: &T)
where
  T: Display + 'static,
{
  println!("{}", t);
}

fn print_c<'a>(t: &'a dyn Display) {
  println!("{}", t)
}

fn print_d<'a>(t: &'a impl Display) {
  println!("{}", t)
}

fn print_e(t: &(dyn Display + 'static)) {
  println!("{}", t)
}

fn print_f(t: &(impl Display + 'static)) {
  println!("{}", t)
}

fn print_g(t: &str) {
  println!("{}", t);
}
```
