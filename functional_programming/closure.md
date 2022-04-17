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

2.
```rust
/* Make it work 
- Dont use `_reborrow` and `_count_reborrowed`
- Dont modify `assert_eq`
*/
fn main() {
    let mut count = 0;

    let mut inc = move || {
        count += 1;
        println!("`count`: {}", count);
    };

    inc();


    let _reborrow = &count; 

    inc();

    // The closure no longer needs to borrow `&mut count`. Therefore, it is
    // possible to reborrow without an error
    //let _count_reborrowed = &mut count; 

    assert_eq!(count, 0);
}
```

3. way1
```rust
/* Make it work in two ways, none of them is to remove `take(movable)` away from the code
*/
fn main() {
     let movable = Box::new(3);

     let consume = || {
         println!("`movable`: {:?}", movable);
         take(movable);
     };

     consume();
     //consume();
}

fn take<T>(_v: T) {}
```

3. way2
```rust
/* Make it work in two ways, none of them is to remove `take(movable)` away from the code
*/
fn main() {
     let movable = &Box::new(3);

     let consume = || {
         println!("`movable`: {:?}", *movable);
         take(movable);
     };

     consume();
     consume();
}

fn take<T>(_v: T) {}
```

4. 
```rust
fn main() {
    let example_closure = |x| x;

    let s = example_closure(String::from("hello"));

    /* Make it work, only changeg the following line */
    let n = example_closure("5".to_string());
}
```

5. way1
```rust
/* Make it work by change the trait bound, in two ways*/
fn fn_once<F>(func: F)
where
    F: FnOnce(usize) -> bool,
{
    println!("{}", func(3));
    //println!("{}", func(4));
}

fn main() {
    let x = vec![1, 2, 3];
    fn_once(|z|{z == x.len()})
}
```

5. way2
```rust
/* Make it work by change the trait bound, in two ways*/
fn fn_once<F>(func: F)
where
    F: FnOnce(usize) -> bool + Copy,
{
    println!("{}", func(3));
    println!("{}", func(4));
}

fn main() {
    let x = vec![1, 2, 3];
    fn_once(|z|{z == x.len()})
}
```

6.

解释：因为str作为参数传入闭包，exec无法提前探测闭包内变量的生命周期，所以需要显示告诉编译器str的生命周期不短与exec，否则exec无法保证讲str，即题目中的“hello”传入闭包

```rust
fn main() {
    let mut s = String::new();

    let update_string =  |str| s.push_str(str);

    exec(update_string);

    println!("{:?}",s);
}

/* Fill in the blank */
fn exec<'a, F: FnMut(&'a str)>(mut f: F)  {
    f("hello")
}
```

7.
```rust
/* Fill in the blank */

// A function which takes a closure as an argument and calls it.
// <F> denotes that F is a "Generic type parameter"
fn apply<F>(f: F) where
    // The closure takes no input and returns nothing.
    F: FnOnce() {

    f();
}

// A function which takes a closure and returns an `i32`.
fn apply_to_3<F>(f: F) -> i32 where
    // The closure takes an `i32` and returns an `i32`.
    F: Fn(i32) -> i32 {

    f(3)
}
fn main() {
    use std::mem;

    let greeting = "hello";
    // A non-copy type.
    // `to_owned` creates owned data from borrowed one
    let mut farewell = "goodbye".to_owned();

    // Capture 2 variables: `greeting` by reference and
    // `farewell` by value.
    let diary = || {
        // `greeting` is by reference: requires `Fn`.
        println!("I said {}.", greeting);

        // Mutation forces `farewell` to be captured by
        // mutable reference. Now requires `FnMut`.
        farewell.push_str("!!!");
        println!("Then I screamed {}.", farewell);
        println!("Now I can sleep. zzzzz");

        // Manually calling drop forces `farewell` to
        // be captured by value. Now requires `FnOnce`.
        mem::drop(farewell);
    };

    // Call the function which applies the closure.
    apply(diary);

    // `double` satisfies `apply_to_3`'s trait bound
    let double = |x| 2 * x;

    println!("3 doubled: {}", apply_to_3(double));
}
```

8.
```rust
/* Fill in the blank */
fn main() {
    let mut s = String::new();

    let update_string = |str| -> String {s.push_str(str); s };

    exec(update_string);
}

fn exec<'a, F: FnOnce(&'a str) -> String>(mut f: F) {
    f("hello");
}
```

9.
```rust

/* Implement `call_me` to make it work */
fn call_me<F:Fn()>(f:F) {
    f();
}

fn function() {
    println!("I'm a function!");
}

fn main() {
    let closure = || println!("I'm a closure!");

    call_me(closure);
    call_me(function);
}
```

10.
```rust
/* Fill in the blank using two approches,
 and fix the errror */
fn create_fn() -> impl FnOnce(i32) -> i32 {
    let num = 5;

    // how does the following closure capture the evironment variable `num`
    // &T, &mut T, T ?
    move |x| x + num
}


fn main() {
    let fn_plain = create_fn();
    fn_plain(1);
}
```

11.

解释：这个题其实重点和闭包没关系，主要是因为x会有不同的返回值（虽然我们看起来一样，但是编译器不知道），所以这个部分要不然只返回一个闭包，如果加入if判断条件，不同的条件下要返回同一个闭包的不同执行结果，可以使用智能指针返回，编译器认为返回值全部都是Box智能指针类型，运行时再根据Box在堆中的指向执行，即可跳过编译器检查返回不同的结果

```rust
/* Fill in the blank and fix the error*/
fn factory(x:i32) -> Box<dyn FnOnce(i32) -> i32> {

    let num = 5;

    if x > 1{
        Box::new(move |x| x + num)
    } else {
        Box::new(move |x| x + num)
    }
}
```
