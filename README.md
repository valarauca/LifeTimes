# LifeTimes Guide
A simple guide to lifetimes.

The only usage of lifetimes is to keep things alive. To understand when/how things die we need to look at the borrow checker.

##Borrow Checker

Rust will automatically kill objects when it passes out of lexiconial scope. In simplest terms this means when you decrease your indentention level, you kill any objects that were created in that indentation level.

Here is a simple example.

```rust
let v: Vec<usize> = vec![5,10,15,20];
{                                              // <- start lexiconial scope block
   let b: Box<&Vec<usize>> = Box::new(&v);     // <- create an object
}                                              // <- scope ends, object dies
```

##What a lifetime means

A life means, keep this thing alive. When something passes out of scope, Rust will automatically de-allocate it. A lifetime prevents this behavior.


##Lifetime paramters location and you

There are generally speaking only a few places you can declare a lifetime.

```rust
fn foo<'a>()
struct Bar<'a>
enum Foobar<'a>
impl<'a> SomeTrait for Bar<'a>
impl<'a> Bar<'a> { 
pub trait MyTrait<'a>
```

These are where lifetimes are declared, or created. 

##Lifetime usage

Depending where declare a lifetime depends how it interacts. Above we outlined 6 examples. For example I'll walk thought them.


###Functions

Here we can see foobar is attempting converting a `[u8]` buffer into a `&str` buffer. 

```rust
use std::str::from_utf8;
fn foobar<'a>(x: &'a [u8]) -> Option<&'a str> {
    match from_utf8(x) {
       Ok(x) => Some(x),
       Err(_) => None
    }
}
```

This is the simplest life time. The `[u8]` buffer is _hopefully_ converted into a `&str` buffer. No allocations are made, we just check there aren't any illegal utf8 values.

###Functions, continued

Now things get a bit more complication. We need to compare the difference between:

```rust
fn foo(x: &mut Bar, ...);
fn foo<'a>(x: &'a mut Bar, ...);
fn foo<'a>(x: &mut Bar<'a>, ...);
```

Each of these behave differently, and have different sematics. Because you are starting different things. Let us explore.

```rust
fn foo(x: &mut Bar, ...);
```

This just just means you are doing interior mutation. For the C++/Java folks this means you are accessing private, or encapsulated data. There are not many rules arounding a `x: &mut T`. This is as far as the compiler is conserned a normal regular borrow.

```rust
fn foo<'a>(x: &'a mut Bar, ...);
```

This is not a borrow. It is a move. Remember our `foobar<'a>(x: &'a [u8]) -> Option<&'a str>` string conversion tool? The underlying buffer `&[u8]` does not exist any more. This is a MOVE not a borrow. When you see `x: &'a T` you can quitely in your head think `x: T`.

```rust
fn foo(x: &mut Bar<'a>, ...);
```

This borrow is generally what people think of when they do the above. When you have a structure like

```rust
struct BufferCollector<'a>
    pub buffers: Vec<&'a [u8]>
}
```

And you want to extract a buffer. The function

```rust
fn get_buffer_at_index<'a>(i: usize, ptr: &mut BufferCollector<'a>) -> Option<&'a [u8]> {
   let len = ptr.buffers.len();
   if i >= len {
      return None;
   }
   Some(ptr.buffer[i])
}
```

Does this. Your buffer stay live. And your `BufferCollector` won't accediently get moved. 
