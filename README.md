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

The only thing a lifetime does is tell Rust: _KEEP THIS ALIVE_!

##Example 1: Build a new object

```rust
pub struct LinesOfText<'a> {                  // <- Object declares a lifetime
     data: Vec<&'a str>                       // <- str has a lifetime assigned.
}                                             //    That lifetime was declared by
                                              //    the object, so this str will 
                                              //    stay alive as long as the object.
                                                
impl<'a> LinesOfText<'a>{                     // <- lifetime is decared with impl<'a>                          
                                              //    This is so LinesOfText<'a> can use it,
                                              //    as LinesOfText<'a> uses a lifetime
                                              //    variable.
                                              //    Does this feel clunky/dumb to you?
                                              //    it does to me too.
                                                
  pub fn new(txt: &'a str)-> LinesOfText<'a>{ // <- lifetime ENTERS _and_ EXITS function
                                              //    This shows the variable ENTERING
                                              //    will stay alive until the object
                                              //    EXITING dies.
                                              //    Notice we are using 'a which was declared
                                              //    in impl<'a>?
                                              //    This is a simplification
    LinesOfText {
        data: text.lines().collect()          // <- The splitting action doesn't create
                                              //    new strings, it borrows them from the
                                              //    orginal. 
    }                                         //    The lifetime lets us keep the borrow
  }                                           //    alive forever!
}
```

##Example 2 Borrow part of something

```rust

fn get<'a>(v:&Vec<&'a str>,i:usize)->&'a str{ // <- lifetime is declared with fn get<'a>
      v[i]                                    //    lifetime ENTERS assigned to the &' str
}                                             //    lifetime EXITS assigned to the &' str
                                              //    This function lets us immutably borrow the
                                              //    internal immutable value without
                                              //    creating new objects!
```

##Thats it!

No really. Those two usages of lifetimes are it. That's all there is. There is some more magic when dealing with trait objects and types. But in reality everything you learned above applies to that, you are just assigning lifetimes to types instead of objects/variables.

So in conclusion you decare lifetimes with 

```rust
impl<'a>
struct<'a>
enum<'a>
fn you_func<'a>
```

Then you just pay attention to what ENTERS, and what EXITS

```rust
fn foo<'a>(incoming: &'a str) -> &'a outgoing
``
