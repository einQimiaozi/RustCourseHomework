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

2.
```rust
/* Adding trait bounds to make it work */
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a:'b, 'b> ImportantExcerpt<'a> {
    fn announce_and_return_part(&'a self, announcement: &'b str) -> &'b str {
        println!("Attention please: {}", announcement);
        self.part
    }
}

fn main() {
    println!("Success!")
}
```

3.
```rust
/* Adding trait bounds to make it work */
fn f<'a:'b, 'b>(x: &'a i32, mut y: &'b i32) {
    y = x;                      
    let r: &'b &'a i32 = &&0;   
}

fn main() {
    println!("Success!")
}
```

4.

关于本题hrtb的理解：

如果使用trait bound作为参数，那么在执行具体的函数前其实编译器无法确定会使用什么trait bound，因此也无法确定该trait bound的生命周期

所以会出现zero生存时间不够长的错误，其实是因为编译器无法推断f的生命周期造成的

hrtb解决的问题就是让编译器知道a跟随f的生命周期，不论f的生命周期是什么
```rust
/* Adding HRTB to make it work!*/
fn call_on_ref_zero<F>(f: F) where for<'a> F: Fn(&'a i32) {
    let zero = 0;
    f(&zero);
}

fn main() {
    println!("Success!")
}
```

5.
```rust
/* Make it work by reordering some code */
fn main() {
    let mut data = 10;
    let ref1 = &mut data;
    let ref2 = &mut *ref1;

    *ref2 += 2;
    *ref1 += 1;

    println!("{}", data);
}
```

6.

解释：

- 原本list的生命周期是a，get_interface的返回值Interface实例生命周期也是a

- 所以在main中list.get_interface().noop()之后Interface实例的生命周期没结束，创建Interface实例的list是一个可变引用，该list的生命周期也不会结束，直到main函数结束

- 所以use_list(&list)这个不可变引用会报错，解决方案是给Interface加一个生命周期b，让get_interface的可变引用list的生命周期和Interface一样也是b，这样调用noop之后Interface被回收，list的可变引用也会被回收

```rust
struct Interface<'b, 'a: 'b> {
    manager: &'b mut Manager<'a>
}

impl<'b, 'a: 'b> Interface<'b, 'a> {
    pub fn noop(self) {
        println!("interface consumed");
    }
}

struct Manager<'a> {
    text: &'a str
}

struct List<'a> {
    manager: Manager<'a>,
}

impl<'a> List<'a> {
    pub fn get_interface<'b>(&'b mut self) -> Interface<'b, 'a>
        where 'a: 'b {
        Interface {
            manager: &mut self.manager
        }
    }
}

fn main() {

    let mut list = List {
        manager: Manager {
            text: "hello"
        }
    };

    list.get_interface().noop();

    println!("Interface should be dropped here and the borrow released");

    use_list(&list);
}

fn use_list(list: &List) {
    println!("{}", list.manager.text);
}
```
