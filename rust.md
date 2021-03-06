# RUST 2015 (+ 2018 update)

# Side Notes
- Statements vs expressions: statements perform an action but do not return a value. Expr evaluate to a resulting value. But expressions can be part of statements (ex: `6` expr from `let = 6;` statement)
- Generic type work best for homogeneous collections but trait bounds work best with multiple possible types
- Convert decimals to binaries: count right to left, from 2^0 to 2^X... add 2^X when a 1 is found. Nice ref to converters (https://www.culture-informatique.net/conversion-binaire-decimal-hexadecimal-main/)
- Giving to the compiler a wrong return type (`let f : u32 = String::from("hello.txt");`) will led to the compiler giving out what type return was excepted

# Code Conventions
- variables are snake case (lowercase + underscore)
- variable signature is optional
- function parameters signature is mandatory
- function return signature is optional
- use the `use` keyword to bring parent module of an elem (fun, struct, enum...) into scope, not directly the element
- exception: if two types have the same name, use `use` on its parent or rename it with `use std::io::Result as IoResult;`

# INSTALLATION AND CARGO COMMANDS (1.1)
On macOS: `curl https://sh.rustup.rs -sSf | sh`
On Windows: `https://www.rust-lang.org/install.html`
## Main commands:
`rustup update`
`rustc --version`; `rustc x.y.z` to hotswap versions
`rustup doc` offline doc
`rustc main.rs && ./main` to compile then execute by hand (`./main.exe` on Windows).
## Cargo (automatically installed with rustup):
`cargo --version`
`cargo rustc`(... `-- -Z print-type-sizes`) to pass arbitrary `rustc` flags (local to a given crate)  
`cargo new pjname` creates a binary (vs library) project with a Cargo.toml file + basic main. use `--lib` to create a lib or `--bin` to explicitly create an executable crate
`cargo build` compiles and creates an exe in `./target/debug/pjname` + creates a Cargo.lock the first time to keep a track of dependencies versions used
`cargo build --release` compiles with optimizations and creates exe in `./target/release/pjname`
`cargo check` checks code but does not actually creates the exe. Faster
`cargo run` compiles + runs it (== `cargo build && ./target/debug/pjname`)
`cargo install` for easy installation of tools (ex: `cargo_update` or `mdbook`)
NB: 2 kinds of errors when compiling. Compilation errors and runtime errors are checked by the `cargo run` command

## Rust features (2018)
###### Projects
Rust projects can either be: a package (set of one+ crates), or since Rust 2018 a workspace (set of packages).
To create one, add `[workspace]` (*Cargo.toml*). With workspaces, packages can be developped individually but they share the same dependencies
```
[workspace]
members = ["path/to/member1", "path/to/member2", "path/to/member3/*"]
exclude = ["path1", "path/to/dir2"] // optional line
```
###### Examples
Examples show people how to use a crate. Can either be a single-file of multiple files
```
my-package
 └──src
     └── lib.rs // code here
 └──examples 
     └── simple-example.rs // a single-file example
     └── complex-example
        └── helper.rs
        └── main.rs // a more complex example that also uses `helper` as a submodule
```
###### `[patch]` to replace dependencies
If a dependencie has a bug, we can use a (eventually corrected) local version of the dependency with patch in the *cargo.toml* file:
```
[dependencies]
foo = "1.2.3"

[patch.crates-io]
bar = { path = '/path/to/bar' }
```
###### Use another registry than crates.io
```
[source.crates-io]
replace-with = 'my-awesome-registry'

[source.my-awesome-registry]
registry = 'https://github.com/my-awesome/registry-index'
```
###### Wildcard dependencies death
Dependencies must now refer to a range: `regex = "1.0.0"` using `> >= < ^ ~`

# MAIN CONCEPTS (3)
- Keywords:
    Keywords names are reserved unless a "raw identifier" is used: `r#keyword`
- Variables
    `let`. By default immutable unless `mut` is used. Limited to their scope.
    Different than `const MAX_POINTS: u32 = 100_000;` that are global and can only be set to constant expr (!= fun call / runtime computed variables).
    Shadowing (`let x = 5; let x = x + 5;`) is another way to reassign. Differences: redeclared every change + copy (vs ref) + type can change.

## Data types (3.2)
Scalar types:
- integers:
    i8 / 16 / 32 / 64 / 128 / isize or u8.../ usize
    Size is the archi depending size ex: 64bit achi = i64.
    Int literals: decimal - hexa (`0x`) - octa (`0o`) - binary(`0b`) - byte (u8 only)
    `_` is a separator.
    NB: u8 = 0 - 255 ; default Int is i32 ; in debug mode overflow is tested and leads to 'panic'. But in release mode: i8 -> 256 becomes 0.
- floats:
    f32 / 64. Default is f64 = double.
- bools:
    one byte size.
- chars:
    whatever unicode size. `'Z'` or `U+0000` or even smileys.

Compound types:
- Tuples:
    groups of several values that can have different types.
    `let tup: (i32, f64, u8) = (500, 6.4, 1);`.
    Can be destructured: `let (x, y, z) = tup;`.
    Access to elements : `tup.1`
- Arrays:
    A single type by array and fixed length
    Arrays are on the stack.
    `let arr: [i32; 5] = [1, 2, 3, 4, 5];`
    Access to elements: `arr[0]`.
    NB: out of bound access is checked by the compiler.

## Functions (pervasives) (3.3)
function return = NO SEMICOLON.
Several values can be returned via tuples.
```
fn main() {
    let (x, y) = get_new_xy(5, 6);
}

fn get_new_xy(x: i32, y: i32) -> i32 {
    println!("The value of x is {}", x); // calls a macro (`!`) not a function.
    (x + 5, y + 2)
}
```
Rust does not care where functions are defined.

## Control Flow (3.5)
- `if cond {}` Expr: condition must be a bool. Example:
```
let number = if condition {
        5
    } else {
        6 // ! must return the same type or ERR.
    };
```
- `loop {}` = forever until `break`
- `for number in (1..4) {}` = safest = more commonly used
- `while cond {}`


# OWNERSHIP (4)
## Definition (4.1)
Central and unique Rust feature, ownership exists to manage heap data. No need for garbage collector.
Rules:
- Each value in Rust has a variable that’s called its owner.
- There can only be one owner at a time.
- When the owner goes out of scope, the value is dropped.
- Value and variable are distinct as the variable can be rewritten after its value was freed.
Ex:
`let s = "hello";` is a string literal (hardcoded) vs `let mut s1 = String::from("hello");` that can be edited and is located on the heap.
String is composed of 3 variables in the stack: a ptr to the heap ; a len ; a capacity (nbr of allocated bytes from the system).
- Move copy: `let s2 = s1;` `s1` variable (pointer, len, capacity) is MOVED into `s2` and `s1` variable becomes invalid (but can be reassigned). s2 is a copy of the stack but still, is a reference to the same heap data.
- Deep clone: `let s2 = s1.clone();` copies the whole data (heap), creating a new pointer.
Some types have the `Copy` trait such as: integer, bool, float, char. Tuples too if they contain only `Copy` types. Means that after `let x = 5; let y = x;`, `x` is still valid.
Ownership is transmitted by returning the variable or passing it to a function as an argument. So be careful when passing a heap variable to a function: return it (tedious) or use references

## References (borrowing) (4.2)
Allows to pass values without passing ownership.
```
let len = get_len(&s1);

fn get_len(s: &String) -> usize {
    s.len
}
```
`s` contains only the ptr to s1.
##### Mutable references
A borrowed value cannot be modified unless `mut` is used: `...fn get_len(s: &String) -> usize {...`. Ofc the initial `String` has to be `mut` too.
Care, `let r1 = &mut s; let r2 = &mut s;` is forbidden: only one mut ref of a data at a given scope.
Can have multiple immutable ref but not a mut while having an immutable one. Security measure, ex:
```
let first = &v[0]; // immutable borrow
v.push(6); // mutable borrow
```
will not compile because if the new element pushed implies a reallocation, the reference to the first would then point to a deallocated zone = crash.
##### Dangling references
Cannot create a variable in a scope and return a reference to this variable since it will be dropped. You must return the variable directly

## Slices (4.3)
It is possible to have a reference of only a portion of a String: (all following statements are equal): 
```
let s = String::from("hello");
let len = s.len();

let hello = &s[0..5];
let hello = &s[0..=4];
let hello = &s[..];
let hello = &s[..len];
```
A String can be passed as a literal: `&s[..]` which has a `&str` type rather than `&String` type
Ofc slice works with every type of collections, not just `String`s


# STRUCTURES (5)
###### Rust 2018 side note
Rust can now force struct alignment in memory by adding before a struct declaration: `#[repr(align(16))]`

## 5.1 Defining and instanciating structs
Like tuples but each variable is named. 
```
Struct User { // definition
    username: String,
    email: String
}

let mut user1 = User { // initialisation. Not necessarily mutable
    email: String::from("test@gmail.com"),
    username: String::from("test")
};

user1.email = String::from("updatedemail@gmail.com");

fn build_user(email: String, username: String) -> User {
    User { email, username } // shorthand if param and field name are the same
}
```
Note that either the entire struct is mutable or it is not at all, fields cannot
##### Creating instances from other instances with stuct update syntax
`let user2 = User { ..user1 };`
##### Tuple structs
Structs without named fields: `struct Color(i32, i32, i32); let black = Color(0, 0, 0);` vs tuple `let color = (0, 0, 0);`
Difference with tuples: struct is named and cannot be swapped with an equivalent typed struct as a function param for example.
##### Unit-like structs
Empty `()` structs. Useful to implement traits on a type within no data needs to be stored
##### Ownership of struct data
It is easier to make sure a struct owns its data (fe by using `String` rather than string literals) but structs can store references owned by something else, by using lifetime specifiers.

## Debug a struct (5.2)
```
#[derive(Debug)] // needed to println
struct Rectangle { width: u32, height: u32 }

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    println!("rect1 is {:?}", rect1); // `:?` identifier needed too or `:#?` for another output format
}
```

## Method Syntax (5.3)
Methods are fun defined in the context of a struct, their first param is always `&self` (the instance of the struct).
```
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 { // (self) is sugar for (self: &Self)
        self.width * self.height
    }
}
```
and is called by: `rect1.area();`.
`self` is only borrowed here bc it just reads, does not write (`&mut self`) / does not need ownership (which is possible).
Rust has a feature called 'automatic de/referencing', adding `& mut *` to match the object signature so `->` is not needed to access an method from a pointer like it would be in C(++...).
##### Associated functions
functions != methods: are part of the struct BUT they do not have a `self` param. Often use to create a new instance of the struct.
It is called by `rect1::my_fun();`


# ENUMS AND PATTERN MATCHING (6)
###### Rust 2018 side note
`union` is a new keyword that works as `enum` but untagged at runtime. Care as they are by definition `unsafe` to use. Limited usage, but they interpolate with C unions. More memory efficients

## `enum` definition (6.1)
```
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```
Accessed by: `let changeColor = Message::ChangeColor(0, 255, 0);`.
Can be used as struct fields or params: `fn route(message: Message) { }`, called `route(Message::Quit);`.
Enums and enum fields can contain data (including methods, structs, other enums)
Methods are declared as they are in structs, using `impl` and can be called on any elem of the enum: `Message::Write(String::from("hello")).call();`
##### Option type (enum)
Replaces the `null` value that cause too many errors.
```
enum Option<T> {
    Some(T),
    None,
}

let some_string = Some(5);
let absent_number: Option<i32> = None;
```
Option type must be specified if `None` is used bc it cannot be inferred.
Be careful! This does not work:
```
let x: i8 = 5;
let y: Option<i8> = Some(5);
let sum = x + y; // adding an int to an option -> compilation error.
```

## The `match` control flow operator (6.2)
Code can be executed in a `match` expr, receive params and return an expr:
```
enum Coin {
    Penny,
    Nickel,
    Quarter(UsState : String), 
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1
        Coin::Nickel => 5,
        Coin::Quarter(state) => {
            println!("State quarter from {}", state);
            25
        },
    }
}
```
`match` is commonly used against `enum`s.
##### Matching with `Option<T>`
```
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
        None => None,
    }
}
```
##### Match is exhaustive
...so all cases must be handled (if not: compilation error)
##### The placeholder `_`
It matches any value
```
    match u8_value {
        1 => 1
        5 => 5,
        _ => (), // if not 1 or 5 do nothing (unit)
    }
```

## Concise control flow with `if let` (6.3)
`match` one pattern while ignoring the rest. Useful when only one particular value is interesting.
```
if let Some(3) = some_u8_val {
    println!("three");
}

// same thing that
match some_u8_val {
    Some(3) => println!("three"),
    _ => (),
}
```
Can have a `else` expr but main usage is to match only 1 case. Kind of sugar for `match`


# PACKAGES, CRATES AND MODULES (7)
Rust features related to scope.
- A path is the name of an item such as a struct, function, or module.
- A module and the `use` keyword let you control the scope and privacy of paths.
- A crate is a tree of modules that produce a binary / library or executable.
- A package is a Cargo feature that let you build, test, and share united sets of crates. (like nodejs packages)
- A workspace is a set of packages that share the same *cargo.lock* (big projects)

- `mod` declares a new module. A module is a namespace that contains definitions of functions, types, structs, enums... Module's code can follow its declaration or in another file
- by default, functions, types, constants, and modules are private unless prefixed with `pub` that makes it visible outside its namespace.
- `use` brings modules or module's definitions into scope

## Packages and crates for making libraries and executables (7.1)
Crate = binary / library.
A crate has a root, a source file that defines how to build the crate
A package has a Cargo.toml file that describes how to build one+ crates
`cargo new` creates a package (bc it has a Cargo.toml file)
`cargo new communicator` will create a library crate with only (*src/lib.rs*). Library crates can be used as dependencies. `cargo run` will not work, use `cargo build`
Add `--bin` to create a binary crate instead (*src/main.rs*). Multiple binaries is made possible by placing files in *src/bin/*, each file will be a separate binary crate
A package can have 0 / 1 binary crate but +inf libs. But if it has both (a lib and a binary) it has 2 crates with the same names

## The module system to control scope; and the filesystem (7.2)
create a (nested) module with crate roots:
```
mod sound {
    mod instrument {
        fn guitar() {}
    }
    mod voice {}
}
```
access it via: `sound::instrument::guitar();` (would need the `pub` keyword here and there)
##### Paths
- absolute: `crate::sound::voice`
- relative: `sound::voice` and uses `self` `super` or an identifier in the current module.
##### Separating modules into different files
*Updated Rust 2018*, `mod.rs` file no longer needed. File rules (that work recursively):
Rust 2015:
```
├── sound
│   ├── instrument.rs (contains `sound::instrument` definition without `mod instrument {` wrapper, already defined in `mod.rs`)
│   └── mod.rs (contains `sound` definition and `instrument` declaration (`mod instrument;`))
```
Rust 2018:
```
├── sound
│   ├── instrument.rs (contains `sound::instrument` definition without `mod instrument {` wrapper, already defined in `mod.rs`)
└── sound.rs (contains `sound` definition and `instrument` declaration (`mod instrument;`))
```
##### `extern crate` to import crates / libraries / modules (obsolete)
*Rust 2018 update*: no longer needer, only mention the crate in the *Cargo.toml* `[dependencies]` list, no need to import it in the actual code (excepted for the `sysroot` crate)
In *src/main.rs*, bring the `sound` module/library into scope with `extern crate sound;`. Note that the `extern crate` should always be in our root module (*src/main.rs* or *src/lib.rs*) even if called from a submodule... then refer to them as if the crates were top-level modules

## Controlling visibility with `pub` (7.2#new_version)
Without `pub` library modules will compile with `never used` functions warnings as they cannot be used outside of the lib
Add `pub` before a function / module definition to make it accessible outside a crate
Even if `pub`, module components stay private: all `mod` children (functions) are private by default
The library itself can from the inside call its own private current (current scope) and ancestor components:
*src/sound/mod.rs* file:
```
pub mod instrument {
    pub fn clarinet() {}
    fn pv_drum() {}
}

mod pv_mod {
    pub fn pub_err() {}
}

fn test() {
    sound::instrument::clarinet(); // will compile as `clarinet()` and its parent are public
    sound::instrument::pv_drum(); // will not compile as `pv_drum()` is not public
    sound::pv_mod::pub_err(); // will not compile as `pv_mod` is not public
}
```
`sound` module is private but appears public to the calling function `test()` as it is on the same level.
Any of its public children are therefore recursively accessible to `test()`, so `clarinet()` can be called.
However `pv_drum()` is private to `test()`. Neither is `pub_err()` even if it public as `pv_mod` is private to `test()` (is not on the same level, nor is an ancestor)
##### Using `pub` with structs and enums
If `pub` is used before `struct` its fields are still private, even simple variables.
If an `enum` is public its elements are too

## Importing names with `use`
##### Concise imports with `use`
`use` can be called from everywhere (function, root, module etc). It is useful to shorten paths.
Naked `use` brings the absolute (NOT relative) path into scope (crate name)
###### `use crate`
`crate` refers to the current (vs external) crate: `use crate::sound::instrument;` brings the absolute path (crate root) into scope
`use sound::instrument;` works (bc `sound` is the crate name) but is a bad practice as it is not explicit
###### `use self` to import using the relative path
`self` refers to the current module: `use self::sound::instrument;`. Only valid way to refer to a relative path
###### `use super` to call parent module
`super` refers to the parent module (ie `../` to the path). As said above, all `super` components are public for the caller
###### `use ::` to explicitly refer to an external crate
`::` explicitly refers to an external (vs local) crate (useful if the root module has the same name as a crate)
###### One `use` for several paths
`use std::io::{self, Write};` brings into scope the `std::io` and `std::io::Write` modules
###### Bringing all public definitions into scope (`*` Glob operator)
`use std::collections::*;`. To use sparingly (mainly in `tests` module)
##### Idiomatic `use` paths for functions vs. other items
`use` path for functions / structs / enums etc is a bad practice. It is better to use `use` on modules to keep it clear that path definition is not local
##### Renaming types brought into scope with the `as` keyword
if two types have the same name, either: bring its parent to scope (exception of the previous rule) or rename it: `use std::io::Result as IoResult;`
##### Re-exporting names with `pub use`
```
mod performance_group {
    pub use crate::sound::instrument;
}
```
`instrument` becomes available not only for the `performance_group` module but for others to bring it into their scope.
##### Using external packages
*Cargo.toml file*
```
[dependencies]
rand = "0.5.5"
```
makes rand available from everywhere: `use rand::needed_trait` is needed to bring a trait of a package into scope
Exception: `std` is a built-in dependency but still needs to be bring into scope with `use`


# COMMON COLLECTIONS (8)
Collections are stored on the heap vs tuples and build-in arrays. So the amount of data stored does not need to be known at compile time

## Vectors `Vec<T>` (8.1)
Store several values of any SINGLE data type.
##### Create
`let v: Vec<i32> = Vec::new();` but if there are initial values, use the `vec!` macro: `let v = vec![1, 2, 3];` (type is inferred)
##### Update
```
let mut v = vec![1, 2, 3];
v.push(5);
```
There is a pop method too that removes and returns the last elements.
##### Drop
The vector and all its elements are dropped when out of scope.
##### Reading elements
`&v[9]` (would take ownership without `&`) or `v.get(9)` gets the ninth elem
If the index does not exist:
- `[..]` will panic. Use it if non-existant indexes should not be passed.
- with the accessor `get` it will return `None`. Should be used with the `match` expr: 
    ```
    match v.get(9) {
        Some() => println("Exists!"),
        None => println("Does not exist."),
    }
    ```
    Use it if it is normal to hit non-existant indexes.

Impossible to hold an immutable ref to a vector and try to add elems: 
```
let first = &v[0]; // immutable borrow not for `first` to take ownership of the data
v.push(6); // mutable borrow
```
will not compile because if the new element pushed implies a reallocation, the reference to the first would then point to a deallocated zone = crash.
##### Iterate
read: `for i in &v {...}`
write: `for i in &mut v { *i += 50; }`. Note that i has to be dereferenced before it can be used with the `+=` operator.
##### Using Enums to store multiple types
```
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```
Nice trick to use any exhaustive types. 

## Strings (8.2)
Collection of UTF-8 encoded chars.
##### Definition
By opposition to *string slices* `str` which are stored in the binary output of the program, `String`s are stored in the heap.
##### Creation
`let mut s = String::new();` creates an empty string
`let s = "test".to_string();` OR `let s = String::from("test");` create a `String` from a literal
##### Update
###### Append
`s.push_str("bar");` or `s.push('c');` (`push` can only take one char).
###### Concatenate
```
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");
let s = s1 + "-" + &s2 + "-" + &s3;
```
`+` operator calls `fn add(self, s: &str) -> String {`.
It works even if `&s2` and `&s3` type is `&String` bc compile can *coerce* `&String` into a `&str` param.
Second argument is a reference so s2 will still be valid. But first is not, add takes ownership of the first param. Note that `&s != &s1`
Concatenation can be done without taking any ownership, using a macro: `let s = format!("{}-{}-{}", s1, s2, s3);`.
##### Indexing into Strings
In Rust a letter can be seen either as:
- bytes: [224, 164, 164, 224, 165, 135]
- scalar values: ['त', 'े'] (second value is a diacritic not a char)
- grapheme clusters (the closest thing to letters): [ "ते" ]
`let c = &"hello"[0];` is invalid as it would be a source of many bugs and a performance issue.
In memory `Strings` are a wrapper over a `Vec<u8>`, `s1[0]` would return the first byte `104` (`'h'` in utf8) and not the first char.
len of `let len = String::from("Здравствуйте").len();` is not 12 (unicode scalar value) but 24 (bytes needed to encode in UTF-8). 
##### Slice
`let s = &"Здравствуйте"[0..4];` `s -> "Зд"` luckily works in this range but it is not safe: `[0..1];` range would cause the program to panic bc the index is invalid.
Solutions:
- iterate over scalar unicode values with `chars`: `for c in "नमस्ते".chars() {`
- iterate over bytes values with `bytes`: `for b in "नमस्ते".bytes() {`
- iterate over grapheme clusters is complex and is provided by crates, not by the standard library.

## Hash Maps (8.3)
The type `HashMap<K, V>` contains a map of key (any type) / value (any other type) pairs. This type is not included in the prelude.
`HashMap`s are hashed and therefore DoS resistant (speed / safe balanced algo). But another *hasher* can be specified (`BuildHasher` trait) that can be implemented by hand or called from a library
##### Creation
`let mut scores = std::collections::HashMap::new();` creates an empty hash map.
`scores.insert(String::from("Blue"), 10);` all the keys / values (distinctly) must have the same type.
```
use std::collections::HashMap;
let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];
let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```
type of score can not be infered bc `collect()` can return different types.
`teams.iter().zip(initial_scores.iter())` creates a vector of tuples using iterators: `it.zip(it)`.
##### Ownership
For `Copy` trait types (i32...) values are copied. For owned values (`Strings`...) the `HashMap` values are moved and it becomes the owner unless a ref is used but the variable must live until the `HashMap` dies.
##### Access
`let score = scores.get(&String::from("Blue"));` As `get` returns an option the result will be `Some(&10)`.
`for (key, value) in &scores {` works.
! order is arbitrary.
##### Update
###### By overwriting a value
Simply redo `scores.insert(String::from("Blue"))` will overwrite the old value.
###### By inserting a value only if key does not exist
`scores.entry(String::from("Blue")).or_insert(50);` `entry()` returns an Enum called `Entry`.
`or_insert` keeps the old value if it exists, if not: inserts the param as the new value. And it returns a mutable ref to the new *(or not)* value.
###### By updating the new value working on the old one
```
let score = map.entry(String::from("Blue")).or_insert(0);
*score += 10;
```
blue team, if exists, sees its score updated. `score` has type `&mut V` so it needs to be deref to be updated. 
##### Print
`println!("{:?}", scores);` will output `{"Yellow": 50, "Blue": 10}`.

# Error Handling (9)
Rust does not have exceptions and instead distinguishes 2 types of errors:
- *unrecoverable* errors
    Stop the execution of the program
    Can be set by the dev
    Covered by the usage of the `panic!` macro
- *recoverable* errors
    ex: file not found.
    Problem is reported to the user but the program keeps running
    Covered by the usage of the `Result<T, E>` type

## Unrecoverable Errors with panic! (9.1)
When the `panic!` macro is executed, the program prints a failure message, Rust cleans the program (*unwinds*) then quit it.
!= *abort* which quits without unwinding and lets the operating system clean the memory. (`panic!` result can be set to *abort* in the *Cargo.toml* file)
```
[profile.release]
panic = 'abort'
```
A call to the `panic!` macro causes this message: `thread 'main' panicked at 'crash message set', src/main.rs:2:4`
When a `panic!` call comes from an external library we will need a backtracker to find the insourced origin. Example: in `let v = vec![1, 2, 3];`, if we try to access a non-existant value (`v[99]`) (*buffer overread*) -> systematic panic: `thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', /checkout/src/liballoc/vec.rs:1555:10`. Running `RUST_BACKTRACE=1 cargo run` with debug symbols enabled (meaning `--release` flag is not set) allows to catch the error from our own code: `11: panic::main at src/main.rs:4`.

## Recoverable Errors with Result (9.2)
`Result<T, E>` definition (`T` and `E` represent the value returned depending on the case):
```
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
Example: `let f = open("filename");` returns a `Result`. If it succeeds `f` will hold an instance of `Ok` that contains the `std::fs::File` (file handler). If it fails `f` will hold an instance of `Err` that contains a `std::io::Error`
##### Matching on different errors
```
fn main() {
    let f = std::fs::File::open("hello.txt");
    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Tried to create file but there was a problem: {:?}", e),
            },
            other_error => panic!("Unrecoverable problem opening the file: {:?}", other_error),
        },
    };
}
```
`Result::` specifier is not needed bc it is part of the prelude
##### Shortcuts for panic on error:
`match` alternatives:
- `let f = File::open("hello.txt").unwrap();`. `unwrap` will either return directly the result of `Ok` or automatically call the `panic!` macro.
- `let f = File::open("hello.txt").expect("Failed to open hello.txt");`. `expect` is similar to `unwrap` with a provided `panic!` message (easier to know where the error comes from)
##### propagating errors
IE return the `Result`/error to the calling code instead of handling it within the function. Can be better
###### The `?` operator
```
fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();
    let mut f = File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s) // called if everything works fine
}
```
(actually all this could be shortened to returning `fs::read_to_string("hello.txt")` which opens the file by itself)
`?` is a nice shortcut: if the `Result` is `Ok`, it is returned from the expression and the program continues. If the `Result` is an `Error`, the `Error` is returned from the hole function
! `?` can only be used inside a function that returns a `Result` or it will not compile.
Note that even the main can return a `Result`: `fn main() -> Result<(), Box<dyn Error>> {`

## To panic! or not to panic! (9.3)
Compiler does not understand logic. So when a code has 0% chances to have an `Err`: call `unwrap()` (returns the value from `Ok` or panics if `Err`) with no check is a good practice.
`unwrap()` (or `expect()`) is a good practice too when you have not yet decided how to deal with the error.
##### Guideline
When a failure is expected, even if the code is in a *bad state*, use `Result` In any other case when code is in a bad state, call `panic!`:
- When a value is invalid and either:
    - not expected to happen
    - code relies on being stable at this point
    - there is no good way to encode the information
    - it exposes to vulnerabilities
- When someone uses your code and passes a values that does not make sense
- When external code fails and there is no way to fix it (unless a failure is the expected possibility, then `Result`)

##### Pre-checks
- Prevent annoying checking by using types rather than `Option` as the compiler itself ensured the value is valid
- Use unsigned types when possible to ensure only positive values
###### Create custom types for validation
Using a module/struct:
    - implement a `new` method that does all checks and returns the good type(s). If not, `panic!` bc the contract is broken. Regrouped and secured checks (at creation).
    - implement public setters/getters and keep variables private, so that only the setter does a check when editing a value (getter example: `pub fn value(&self) -> i32 { self.value }`)


# GENERIC TYPES, TRAITS AND LIFETIMES (10)
## Generic data types (10.1)
A generic type name can represent any given type but once it has been given, it keeps this same type. ?????
##### In functions
`fn largest<T>(list: &[T]) -> T {`: `<T>` before params in the signature is mandatory so the compiler knows what `T` means
The input and output of the function must be of the same type but it can be any type.
##### In structs
`struct Point<T> { x: T, y: T, }`. Note that code will not compile if all type `T` fields are not of the same type.
But there can be different *generics*: `struct Point<T, U> { x: T, y: U, }`.
##### In enums
`enum Option<K, E> { Ok(K), Err(E), };`. Same thing.
##### In method definitions
We need to declare `<T>` after `impl` too so the compiler knows that the `T` type from `Point` is a *generic*.
Generic types declared in the implementation methods can differ from the original ones, Ex:
```
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}
```
`impl` generic type declarations can differ from method type parameters but must be the same as struct.
Note that `<T, U>` are declared paired with Struct definition, `<V, W>` are declared after that, in their own relative scope 
##### Performance of code using generics
NO performance difference at runtime between generic and its concrete equivalent type. But takes longer to compile bc of the *Monomorphization* process (recreate code with concrete types from the generic form)
```
//generic
enum Option<T> { Some(T), None, }
let integer = Some(5);
let float = Some(5.0);

// monomorphized
enum Option_i32 { Some(i32), None, }
enum Option_f64 { Some(f64), None, }
let integer = Option_i32::Some(5);
let float = Option_f64::Some(5.0);
```

## Traits: defining shared behavior (10.2)
A *trait* tells the compiler about a type's fonctionalities. Resemble to *interfaces*
Traits save runtime performance and crashes as the compiler checks that every (generic) type implements needed methods
##### Trait definition
```
pub trait Summary {
    fn summarize(&self) -> String; // note the semicolon != brackets
}
```
Now, any type that implements the `Summary` trait must have a `summarize` method with the same signature
A trait needs to be `pub` so that another crate can implement it
Implement it by adding `TraitName for` between `impl` and `StructName`, then define methods as usual:
```
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}
```
It is possible to make a trait local to a scope
! *coherence* restriction: a trait can be implemented on a type only if either the type or trait is local (to our crate)
##### Default implementations
Are overwritten by specific methods for a given trait but are useful. Defined directly in the trait definition (vs semicolon)
```
pub trait Summary {

    fn summarize_author(&self) -> String;
    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author()) // default implementation
    }
}
```
To use a default method, implement the trait but do not define or declare the default method (ignore it). By re-defining it, the implementer would override it
Note that it isn’t possible to call the default implementation from an overriding implementation of that same method
##### Traits bounds
Specify *trait bounds* on a generic type means telling the compiler a type must hold an implementation a given trait
`pub fn notify(item: impl Summary) {` announces the `item` type implement `Summary` and therefore has access to the `summarize()` method.
Note that there are 3 possible syntaxes and several traits can be `impl`:
- `pub fn notify(item: impl Summary + Display) {`
- `pub fn notify<T: Summary + Display>(item: T) {`
- Use the `where` keyword:
    ```
    pub fn notify<T, U>(item: T, foo: U) -> i32
        where T: Summary + Display,
              U: Clone + Display  
    {
    ```
##### Returning traits
`fn returns_summary() -> impl Summary {`
Says a type that implements the `Summary` trait is returned but the exact type is unknown
! If the return type (which impl the trait anyway) is unsure depending on how the function goes, it will not compile
##### Compare `largest` function with trait bound
To allow comparison and slices largest needs: 
```
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0]; // Copy trait needed to copy a <T>
    for &item in list.iter() {
        if item > largest { // PartialOrd trait needed to compare <T>s
            largest = item;
        }
    }
    largest
}
```
##### Using trait bounds to conditionally implement methods
```
impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```
Here `cmp_display` is available to an hypothetical `Pair<T>` only if the `T` type has `Display + PartialOrd` implemented.
*Blanket implementations* are extensively used: `impl<T: Display> ToString for T { }`. Here the `to_string()` method can be called on any type that implements the Display trait. That allows to turn ints to strings bc integers implement display (`let s = 3.to_string();`)

## Validating references with lifetimes (10.3)
Every reference (/variable) has a *lifetime* (`'a`, '`b'...), ie the scope for which it is valid
*Generic lifetime parameters* are explicit lifetimes that ensure a reference stays valid as long as it needs to be. IE they define the relationship between references for the borrow checker to perform 
The *borrow checker* verifies the subject of a reference (value) does not live as long as the reference (variable).
They mainly prevent from dandling references
Inferred unless the return value and at least two parameters are references. Their purpose is to connect the lifetimes of parameters and return values:
```
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
will compile because lifetimes are explicit. It would not otherwise. We are not modifying their lifetime, we are just precising these to the compiler by saying "these three variables will at least exist in this scope".
Ofc if the returned reference refers to a variable created in the function it will not compile anyway (here should be returned the owned data, not a ref).
The compiler always thinks the lifetime is equal to the smaller of the lifetimes. So this code will not compile:
```
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
```
##### Lifetime annotations in struct definitions
Struct can hold references, but only if they have a lifetime annotation.
`struct ImportantExcerpt<'a> { part: &'a str, }` it means an instance of this struct cannot outlive its shortest ref lifetime (`'a`, that has existed before the instance)
##### Lifetime elision rules
There are *input / output lifetimes* (params / return)
Lifetime annotation can be inferred only with `fn` and `impl` blocks definitions. If the 3 following rules are applied and a reference does not have a certain lifetime, the annotation will have to be specified:
- each ref param gets its own lifetime
- if there is exactly 1 ref param its lifetime is assigned to all output lifetime parameters
- if one of several params is `&self` or `&mut self` its lifetime is assigned to all output lifetime parameters
#####  Static lifetime
`'static` is a special lifetime which denotes the entire duration of the program: like all string literals (they are stored in the binary). Should be used only when we want a variable to last the entire program


# Testing (11)
Test body: set up any needed data or state -> run the code you want to test -> assert the results are what you expect

## Writing tests (11.1)
##### Test attribute
When `cargo test` is run, Rust runs every `#[test]` function and report whether each test function passes or fails
We can add as many `mod` and `fn` tests as we want
Each test is run in a new thread. If one fails it means the test function panicked.
```
#[cfg(test)]
mod tests {
    #[test] // there can be non-test functions in a test module
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }
    
    #[test] // put it before every fun to start when `cargo test` is ran
    fn another() {
        panic!("Make this test fail");
    }
}
```
`cargo test` outputs:
```
running 2 tests
test tests::exploration ... ok
test tests::another ... FAILED

failures:
---- tests::another stdout ----
    thread 'tests::another' panicked at 'Make this test fail', src/lib.rs:10:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::another

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

    Doc-tests adder // only used for documentation tests
... 
```
###### `assert!` macro
Takes a boolean. If it returns `true` test passes but if it returns false `panic!` is called and test fails (remember to negate a function if `false` means no issue)
###### Test equality with the `assert_eq!` & `assert_ne!` macros
`assert_eq!` checks for equality of 2 expr (left|excepted and right|actual) ; `assert_ne!` checks for inequality (useful when unsure what a value will be but sure what a value should not be)
For each failed test values passed are printed (easier to see why the test failed)
In some cases (locally defined structs and enums) will have to be added `#[derive(PartialEq, Debug)]` to check equality & print the values
##### Custom failure messages
A custom message can be printed by adding a 2nd+ param to `assert...!` macros. It works as the `format!` macro (variadic params):
`assert!(false, "Greeting did not contain name, value was {}", result);`
##### check for panics
The attribute `#[should_panic]` sets a test as failed if it did not call `panic!`
Declared the line after the `#[test]` attribute.
`expected` allows to check a `panic!` message includes the expected string literal: `#[should_panic(expected = "Guess value must be less than 100")]`. Rust will tell if the program panicked and the right substring was printed, or will set the test as failed.
##### Using Result<T, E> in tests
Tester can work with `Result`: returning Ok() (passed) / Err() (failed) instead of panics
Note that `#[should_panic]` does not work with it

## Running tests (11.2)
Note: ` -- ` is a separator. Pre `--` params apply to `cargo test` & post params apply to the binary
###### Running tests in parallel or consecutively
`cargo test -- --test-threads=1` means NO parallelism
Takes longer but tests will not interfere with each other if they share a state (could output a biased result)
###### Showing function output
By default the test environment captures the standard ouput of passed tests (only), ie `println!` will actually not be printed... unless `cargo test -- --nocapture` flag is set
###### Running a subset of tests by name
A filtered test can be run by calling any function that has a `#[test] attribute`: `cargo test test_fn_name`
! Tests will run on every `test` functions that have the param as a substring
! Include test module names too bc the checked string is `module_nme::fn_name`
The `#[ignore]` attribute (after `#[test]` line): these will not be tested unless `cargo test -- --ignored` is run, which will only test `ignored` functions

## Test organization (11.3)
- *Unit tests* check every module, even private interfaces
- *Integration tests* check code as an external user would
Both are important to be sure pieces of a program work both separately and together
##### Unit tests
Located in the *src* directory, in each file, alongside the code they are testing
Convention is to create a `tests` module in each file, annotated with `#[cfg(test)]` which tells Rust to compile it only when `cargo test` is run
The test being in the file, it has access to private elements that are on the same level (non-recursive)
##### Integration tests
Entirely external to the library. No need for `#[cfg(test)]` but they can only call public API functions
Create a *test* directory next to *src*, with a (for example) *tests/integration_test.rs* file that contains:
```
use crate_name;

#[test]
fn it_adds_two() {
    assert_eq!(4, crate_name::add_two(2));
}
```
When running `cargo test` 3 sections will now appear: unit / integration / doc tests but only integration test can be run: `cargo test --test integration_test` (as the filename). Multiple files (modules) can be added (ex one for each functionality tested). Each module will have its own test output section
###### Integration tests for binary crates
Rust projects should always provide both a *src/main.rs* that calls its logic from the *src/lib.rs* file to allow for integration tests. IE the executable being a standalone, `use` does not work therefore only *libraries* are exposed to the *tests* dir

# FUNCTIONAL LANGUAGE FEATURES: ITERATORS AND CLOSURES (13)
## Closures: anonymous functions (13.1)
Their purpose is to be executed later. *Closures* are peculiar anonymous functions that are storable in a variable or passable as function arguments
Unlike functions they capture their environment (can access parents scope variables)
light syntax for inline closure: `let add_one = |x| x + 1 ;` ; full syntax:
```
let closure_sum = |param1, param2| { // optional form: |param1: u32, param2: u32| -> u32
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    param1 + param2
}; // semicolon needed to end the statement

let result = closure_sum(24, 35); // called like functions
```
`closure_definition` contains the definition of the function not its result. Types are infered so annotation is optional bc unlike functions closures are not interfaces exposed to users
! The compiler infers types & traits matching the first call to a given closure so recalling it with different input or output vtypes will not compile
##### Closures capture their environment
Closure have, like functions, at least one of these traits that are inferred depending on what a closure does of its environment variables:
- `FnOnce` takes ownership and consumes the environment variables, meaning this closure can only be called once
    All closures implement it
    To take ownership, use the `move` keyword before the parameter list (`let equal_to_x = move |z| z == x;`)
- `FnMut` same but mutably: can change the environment
    Implemented if the closure does not move the captured variables
- `Fn` borrows values from the environment immutably
    Implemented for closures that do not need mutable access to the captured variables
##### Using closures with generic types and the `Fn` traits
Storing both a closure and its result in a struct is known as *cache, memoization, lazy evaluation*.
Each closure instance has its own unique anonymous type so whatever their signature is closures will have different types. Best is to use them with generics
Closures are functions-like: one+ of `FnOnce` `FnMut` or `Fn` traits must be implemented too.
```
struct Cacher<T>
    where T: Fn(u32) -> u32 // every closure will have to take a u32 and return a u32
{
    calculation: T,
    value: Option<u32>,
}

impl<T> Cacher<T>
    where T: Fn(u32) -> u32
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => {
                if v == arg {
                    v
                } else {
                    self.value = (self.calculation)(arg);
                }
            },
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            },
        }
    }
}
```
and called as:
```
fn generate_workout(intensity: u32, random_number: u32) {
    let mut expensive_result = Cacher::new(|num| {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    });

    if intensity < 25 {
        println!("Today, do {} pushups!", expensive_result.value(intensity));
        println!("Next, do {} situps!", expensive_result.value(intensity));
    } else {
        println!("Today, run for {} minutes!", expensive_result.value(intensity));
    }
}
```
TODO: use a hashMap to store several results
TODO: increase the `Cacher` flexibility by implementing more types

## Processing with iterators (13.2)
*Iterators* are a way of processing series of elements. They implement the `Iterator` trait defined as:
```
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>; // unique mandatory function to define
    // methods with default implementations elided
}
```
`iter()` takes immutable references to the values, but `into_iter()` takes ownership and returns the owned values. If references are mutable, `iter_mut()` can be used
##### Methods
- Methods that consume the iterator = all methods that call `next()`, like `sum()`: `let total: i32 = vec![1, 2, 3].iter().sum();`
- Methods that produce other iterators = *iterator adaptors* from which are created new values once consumed, like `map()`: 
```
let v1: Vec<i32> = vec![1, 2, 3]; // will stay inchanged
let v2: Vec<i32> = v1.iter().map(|x| x + 1).collect();
```
`collect()` has to be called since `map()` does not consume the iterator (just updates it). `collect()` consumes the iterator and returns a collection of the new data
###### Iterators + closures: common use case
```
#[derive(PartialEq, Debug)]
struct Shoe { size: u32, style: String, }

fn shoes_in_my_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}

let shoes = vec![
        Shoe { size: 10, style: String::from("sneaker") },
        Shoe { size: 13, style: String::from("sandal") },
        Shoe { size: 10, style: String::from("boot") },
    ];

let in_my_size = shoes_in_my_size(shoes, 10); // returns only the 10-sized shoes
```
##### Creating an iterator
```
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;

        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}
```
This offers access to `.map()`, `.filter()`, `.sum()`, `.zip()`, `.skip()` etc, methods of `Iterator`

## Performances: loops vs iterators (13.4)
Iterator are a *zero-cost abstrations* feature bc it is compiled down to low-level code. Rust *unrolls* loops.
Iterators even are slightly faster than loops


# CARGO AND CRATES.IO (14)
## Release profile (14.1)
Cargo has 4 profiles: `dev` (used with `cargo build`), `release` (`cargo build --release`), `test` (`cargo test`) and `doc` (`cargo doc`)
In *Cargo.toml* can be added [profile.*] sections as for example:
```
[profile.dev]
opt-level = 1 // [0..3]
```
The higher `opt-level` is, the most optimized is the compiled code
## Publish crates to crates.io (14.2)
##### Documentation comments
Documentation comment `///` use the Markdown format
`cargo doc --open` will generate a Markdown documentation from project code comments
```
/// Adds one to the number given
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```
And it will create a doc with a side panel for the function names with information, signature, examples, etc. Common sections: Panics, Examples, Errors, Safety
Running `cargo test` will run the code examples in your documentation as tests
`//!` To add top-level (crate) doc comments. If put after a `///` it will add into the top-level doc section relative to a function
##### Exporting as a Public API with `pub use`
Internal structure can differ from public API. A crate can have a nice internal hierarchy that is not convenient for public users: it is not requiered to (re)design it: `pub use` removes the internal organization from the public API and re-exports an item at a higher-level (from the file it is called)
`pub use UsefulType;` in *src/lib.rs* allows to avoid `use my_crate::some_module::another_module::UsefulType;` and rather do: `use my_crate::UsefulType;`
##### Set up crates.io account
Create an account on the site, get an API token then do `cargo login API_KEY_VAL` (stored in *~/.cargo/credentials*) to be allowed to publish crates
##### Crate metadatas
*Cargo.toml*
```
[package]
name = "guessing_game"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
edition = "2018"
description = "A fun game where you guess what number the computer has chosen." // display in search results
license = "MIT OR Apache-2.0" // multiple licences is ok
```
If it is not a SPDX licence, create a *licence-file* and specify its name in the `licence=` field
##### Publish, update, remove
`cargo publish`
To update a package: update the `version=` field from the *Cargo.toml* file (follow this convention *https://semver.org/*) then re-run `cargo publish`
`cargo yank --vers 1.0.1` (which can be undone: `cargo yank --vers 1.0.1 --undo`) prevents every new project to depend on that version. But projects that already used that version will continue to depend on it with no issue

## Organize large projects with workspaces (14.3)
It is the possibility to split up a project into multiple library crates (packages) that share the same *Cargo.lock* and output directory
! There will only be a single, shared, *target* directory so not every crate has to be rebuilt every time
##### Creation
Idiomatic way: create a folder for each *package* with a *Cargo.toml* file with its own package name as a workspace member:
```
add_workspace
├── Cargo.lock
├── Cargo.toml
├── add-one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```
*add-one/Cargo.toml* file:
```
[workspace]
members = [
    "add-one",
]
```
then run `cargo new add-one` from the *add-one* dir.
Update top-level *Cargo.toml* as:
```
[workspace]
members = [
    "adder",
    "add-one",
]
```
run `cargo new add-one --lib` from the *add-one* lib dir
##### Dependencies between pacakages
To use the `add-one()` function (from `*add-one/src/lib.rs*`) in the `adder` package, add to *adder/Cargo.toml* the following dependence:
```
[dependencies]
add-one = { path = "../add-one" }
```
Then in *adder/src/main.rs* `add-one()`:
```
use add_one;

fn main() {
    println!("{}, add_one::add_one(10));
}
```
Now it is possible to run `cargo build` in the top-level *add* directory... and run it with `cargo run -p adder`
##### Add external crate to a workspace
To make an external crate available to a workspace with `use cratename;`, add this to *workspace/Cargo.toml* file:
```
[dependencies]
rand = "0.3.14"
```
##### Add testing to a workspace
In *add-one/src/lib.rs* add:
```
pub fn add_one(x: i32) -> i32 { /// already added
    x + 1
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(3, add_one(2));
    }
}
```
`cargo test` will only test its own level crate, not sub-packages. Test directly from a package dir or add `cargo test -p add-one` to test it
##### Publish to crates.io
You can run `cargo publish` from every crate dir as they are separated crates. IE doing so from the top-level package will publish separately every crate. Workspaces are more of a developer tool

## Install and run binaries from crates.io (14.4/5)
`cargo install` allows to install and use binary crates locally. They are located 
Crates can either be one+ libaries (not runnable but suited to fit in other programs), or a binary (runnable program if there is a *src/main.rs*) or both
If cargo is set in the `$PATH` (should be `$HOME/.cargo/bin`), installed rust programs can be run with `cargo program-name`


# SMART POINTERS (15)
Data structures that have useful methods, have their core data on the heap and can share ownership.

## Box to point to data on the heap (15.1)
Boxes do not have capabilities, simple and light (no overhead) smart pointer. They only implement the `Deref` and `Drop` traits.
##### Allocation
`let b = Box::new(5);`, `b` contains a pointer to the value `5(i32)` allocated on the heap. Like stack data both the data and the pointer will be deallocated once out of scope
##### The `Cons` list to allow recursion
Receives a pair of arguments, the value of the current item and the next item. The last item has a `Nil` value without a next item
Rarely useful, use `Vec<T>` most of the time instead
```
enum List {
    Cons(i32, List), //  will not compile: Cons(i32, Box<List>), would work
    Nil,
}
```
This list is recursive so Rust cannot guess its definitive size so the compiler will not compile. Using a `Box<List>` would work because only a pointer will be stored on the stack, the unknown size (list) will be on the heap

## `Deref` trait to treat smart pointers like references (15.2)
Implementing the `Deref` trait allows to customize the dereference operator (`*`), therefore allows for smart pointers to work like a reference. It allows to dereference something that is not a ref. Derefencing (`*`) means pointing to the value of a reference (like in C)
```
let x = 5;
let s = &x;
let h = Box::new(x);

assert_eq!(x, *s); // are equal
assert_eq!(*s, *h); // are equal, would not be possible to use `*` without the Deref trait
```
##### Implement the `Deref` trait on a struct
```
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}
```
So doing `*h` == `*(h.deref())`
Note that `deref()` has to return a ref and not a value so `self` keeps the ownership
##### *deref coercion*
Happens automatically even on local implementations of `Deref`, when a ref to a type's value is passed to a function that was waiting for another type of parameter. It converts into the right type using `deref()`. So code can work bot with references and smart pointers. Like `&MyBox<String>` -> `&String` -> `&str`. With/out *deref coercion*:
```
let m = MyBox::new(String::from("Rust"));
hello(&(*m)[..]); // without
hello(&m); // with
```
`*m` dereferences `MyBox<String>` into `String` and `[..]` turns the `String` into a string slice.
##### mutable *deref coercion*: `DerefMut`
When `T: DerefMut<Target=U>` is implemented, `&mut T` will be converted to `&mut U`. But can too convert `&mut T` to `&U`: Rust can coerce a mutable into an immutable ref (not the opposite as it would break the borrowing rules)

## Running code on cleanup with the `Drop` Trait (15.3)
`Drop::drop()` is automatically called when a value gets out of scope and cleans memory accordingly. The implementation only adds up code to run at this moment. Ex: `Box<T>` customizes `Drop` to deallocate the space on the heap
###### Implementation
```
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Instance drops!");
    }
}
```
`println` will be called when an instance is out of scope. No need to bring `Drop` into scope as it is in the prelude
Variables are dropped in the reverse orer of their creation
##### Dropping a value early
Can be useful when using locks for example. Calling `Drop::drop()` would not compile, `std::mem::drop(T)` needs to be called instead. Its code is actually empty but since it takes ownership of the passed variable, the variable is dropped at the end of the call. Do not be scared to drop too early, Rust checks valid references so the compiler would yell

## `Rc<T>`, the *reference counting* smart pointer (15.4)
Designed to work with multiple ownerships pointing to the same heap data. Use it when it cannot be determined at compile time which owner will stop using the data last. It keeps count of every active reference to determine whether or not a value is still in use or can safely be dropped
Allows to have multiple IMmutable references (workaround: those can point to mutable data on the heap)
Note: `Rc<T>` works only on single-threaded scenarios
```
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a)); // unidiomatic: a.clone()
    let c = Cons(4, Rc::clone(&a)); // unidiomatic: a.clone()
}
```
Each call to `Rc::clone()` will increase the count, and the data will only be cleaned once count = 0. Count is automatically decreased
`Rc::strong_count(&a)` returns the active count

## `RefCell<T>` and the interior mutability pattern (15.5)
*Interior mutability* is a design pattern that allows mutability of data attached to immutable references. It uses `unsafe` code to workaround Rust's rules. Single ownership to the data. Useful when you can guarantee your program will work but the compiler cannot ensure it
##### Interior Mutability: mutable borrowing to an immutable value
Workaround if a function uses a trait that does not allow for mutability but mutability is needed: `RefCell` will wrap the variable and allow for mutability by calling `borrow_mut()` that returns a mutable reference to the value inside (`RefMut<T>`). An immutable `borrow()` works too, it increases the `RefCell` count and returns a `Ref<T>`. Ex:
```
let data = RefCell::new(5);
let x: &mut i32 = &mut r.borrow_mut();
```
##### Borrowing rules checked at runtime
Borrowing rules reminder: a program can have either (but not both) one mutable reference or any number of immutable references. References must always be valid.
But being `unsafe`, `RefCell<T>` sees the borrowing rules applied at *runtime* rather than *compile time* and errors cause panic (ex: if two mut are borrowed -> panic: `already borrowed: BorrowMutError`).
Consequence: slower and not errors/memory-safe
##### Having multiple owners of mutable data by combining `Rc<T>` and `RefCell<T>`
`Rc<T>` allows to have multiple owners who are limited to an immutable access... Unless `Rc<T>` holds a `RefCell<T>`. Would look like:
init a list of: `Rc<RefCell<i32>>;`, `let value = Rc::new(RefCell::new(5));` and create clones of `let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));` then `*value.borrow_mut() += 10;`: every clone has the same `value` value as `a`
Note: interior mutability can also be provided by `Cell<T>` (which copies the value instead of giving references to the inner value) or `Mutex<T>` (which is safe across threads)
###### Side note: update a `Rc<RefCell<T>>`
Several ways to do so:
```
let rc = Rc::new(RefCell::new(10));
{
    // all the following work
    let mut x: RefMut<i32> = rc.borrow_mut(); // OR
    let mut x = rc.borrow_mut(); // OR
    let x: &mut i32 = &mut rc.borrow_mut(); // OR
    *x += 10;
}
```
## Reference cycles can leak memory (15.6)
Ex: using `Rc<T>` and `RefCell<T>` where items refer to each other in a cycle = memory leaks, ref count will never reach 0 -> values will never be dropped:
```
let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));
let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));
if let Some(link) = a.tail() {
    *link.borrow_mut() = Rc::clone(&b);
}
```
Solution: when having cycles made up of non/ownership relationships, only the ownership relationships should affect whether or not a value can be dropped
##### Using *weak references* to prevent reference cycles by turning an `Rc<T>` into a `Weak<T>`
`Rc::downgrade()` creates a `Weak<T>`. Each `Weak<T>` clone increases the `weak_count` but does not change the `strong_count`. When an `Rc` goes out of scope if its `strong_count` is 0, it will be dropped even if its `weak_count` is not 0. Makes *weak ref* cycles safer as they can be broken
As a consequence, an `Rc<T>` could already be dropped when `Weak<T>` is called so it has to be checked for safe sake: `weak_instance.upgrade()` returns an `Option<RC<T>>` that holds a `Some(Rc<T>)` or `None` if it has already been dropped
###### Creating a tree data structure: a `Node` with child nodes
```
use std::rc::{Rc, Weak};
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
}
```
Doing this allows parent to own a mutable reference to the child Node and a `Weak<T>` ref (does not own) from the child to the parent so that parent can drop child (not the other way) and there is no refence cycle

# FEARLESS CONCURRENCY (16)
Rust handles threads has a 1:1 relation with operating system threads but M:N relation has been made possible through crates.
Issues Rust ownership / concurrency system prevents from happening:
- *Deadlocks* = both threads are waiting for each other preventing both threads from continuying
- *Race conditions* = threads are accessing data in an inconsistent order
- Bugs hard to reproduce

## Threads (16.1)
`thread::spawn()` takes a closure (that cannot take any param) and executes code in a new thread
```
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi, number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    handle.join().unwrap();

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```
will print every `handle` then main thread.
If `handle.join()` had been called after the main `for` there would have been 2 `println!` running concurrently.
Without `handle.join()` the program would have ended with the main thread, without waiting for the `handle` thread to end.
##### Ownership system of threads
Thread closures cannot capture their parent environment (borrow) bc a variable could die in the main thread before being called in the current one
`move` closure allows to take ownership of variables called within the closure (`let handle = thread::spawn(move || {`)
Ofc a variable passed to a `move` thread cannot be accessed in its former context

## Using message passing to transfer data between threads
Channels allow to send messages from several *sending* that produce values to an unique *receiver* end that consumes these values
```
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel(); // (transmitter, receiver)

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```
Note that receiving can be done with `recv()` (will block the main thread's execution and wait for a value) and `try_recv()` (non-blocking version of `recv()` that returns a `Result<T>)
```
let (tx, rx) = mpsc::channel();

let tx1 = mpsc::Sender::clone(&tx); // tx1 becomes another producer of messages alongside tx
thread::spawn(move || {
    let vals = vec![
        String::from("hi"),
        String::from("from"),
        String::from("the"),
        String::from("thread"),
    ];

    for val in vals {
        tx1.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

thread::spawn(move || {
    let vals = vec![
        String::from("more"),
        String::from("messages"),
        String::from("for"),
        String::from("you"),
    ];

    for val in vals {
        tx.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

for received in rx { // acts as blocking recv() calls
    println!("Got: {}", received);
}
```
The `for` from the main will wait for messages to come and will print both thread messages concurrently
! Once a variabe has been sent its ownership is transmitted, therefore it is no longer accessible in the current scope / thread

## Shared-state concurrency (16.3)
##### Mutexes: allow access to data from one thread at a time
Mutex is a shortcut for *mutual exclusion*: it only allows access (asking for the lock) to some data one thread at a time
Usage rules:
- Mutexes attempt to access the lock before using the data
- Mutexes unlock the data after using it so other threads can acquire the lock
Rust advantage: no risk to get un/locking wrong
Mutexes provide interior mutability like `RefCell<T>`: they are immutable but we can get a mutable ref of the data it contains
! Mutexes can lead to deadlocks (two threads waiting for each other forever bc 2 locks are needed for them)
##### `Mutex<T>` (smart pointer) API
```use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {:?}", m);
}
```
`.unwrap()` is important to make the thread panic if a lock fails
`.lock()` returns a smart pointer called *MutexGuard* with the `Deref` trait to point to inner data + `Drop` trait to release the lock automatically when a *MutexGuard* goes out of scope
##### Share `Mutex<T>` between multiple threads
```
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];

    for _ in 0..10 {
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```
Will not compile bc Rust prevents from moving *counter* (the mutex) ownership into multiple threads
Could solution be: `Rc<T>` to have multiple owners, replacing `let counter = Mutex::new(0);` by `let counter = Rc::new(Mutex::new(0));` and calling `let counter = Rc::clone(&counter);` on each iteration ?
No, it would not work bc `Rc<T>` is not safe to share across threads (reference count calls are done non-concurrently so they could be interrupted by another thread so Rust prevents it)
Real solution: `Arc<T>`
##### Atomic reference counting with `Arc<T>`
Atomic work just like primitives (here `Rc<T>`) but are cross-thread safe and come with a performance cost
```
use std::sync::{Mutex, Arc};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

## Extensible concurrency: `Sync` and `Send` traits (16.4)
Rust has very few built-in concurrency features so look at crates / standard library / yourself
Built-in are `std::marker` traits `Sync` and `Send`
##### Allowing transference of ownership between threads with `Send`
Indicates that the ownership of the type implementing `Send` can be transferred across threads.
Almost every type *is* `Send` (ie implements `Send`). Exceptions: raw pointers; `Rc<T>` to avoid 2 threads updating the reference count at the same time. So Rust prevents using `Rc<T>` across threads, erroring: `the trait Send is not implemented for Rc<Mutex<i32>>`
##### Allowing access from multiple threads with `Sync`
Indicates that it is safe for a type that is `Sync` to be referenced from multiple threads
If a `&T` is `Send` then it is `Sync` too
Almost every type is `Sync` too (every primitives). Exceptions: `Cell<T>` related types (bc of runtime borrowing impl is not thread-safe), `Rc<T>` (bc of the same reason it is not `Send`)
##### Implementing `Send` and `Sync` manually is unsafe
Bc if a type does not come with it means something, these types are not made up to be manually implemented


# OBJECT ORIENTED PROGRAMMING FEATURES OF RUST (17)
It is a design pattern in which objects pass messages to each other
Rust is influenced by many programming paradigms including OOP and functional

## What does object-oriented mean ? (17.1)
Objects, encapsulation, inheritance are features OOP / Rust share but depending on definitions Rust is/not an OOP language
##### Objects contain data and behavior
OOP definition (one of many) that tends to define Rust as OOP (objects = `struct` & `enum`, since `impl` block provides methods):
> "Object-oriented programs are made up of objects. An object packages both data and the procedures that operate on that data. The procedures are typically called methods or operations."
##### Encapsulation that hides implementation details
Encapsulation means only the public API of an object can be reached, its internals are not and can be changed transparently.
Rust allows encapsulation with the `pub` keyword, by default modules / types / functions / methods are private
##### Inheritance as a type system and as code sharing
Polymorphism != Inheritance. Polymorphism means code that works with data of multiple types. There are the:
- *ad hoc polymorphism*:  operator overloading. Rust features it
- *parametric polymorphism*: generic parameter with a trait bound. Rust features it
- *subtype polymorphism* = inheritance = reuse of code like parent object behavior without needing to redefine it ; child type can be used in place of parent type
    There is no proper inheritance feature in Rust but traits allow to share methods implementation and generics abstract over possible types (*bounded parametric polymorphism*)

## Using trait objects that allow for values of different types (17.2)
Example: a `gui` crate that allows to draw components such as `Button`, `Image`, `SelectBox` or whatever the user would want to integrate
An OOP implementation would be to define a parent class named `component` that implements a `draw` method. Its children would be `Image` etc and inherit, override `draw`. Rust equivalent:
##### Define a trait for common behavior
Parenthesis: `enum` and `struct` have methods and data separated but traits are more like objects as they combine data and behavior. But they have a specific purpose: allow abstraction across common behavior
Define a `Draw` trait that will have a `draw` method: 
```
pub trait Draw {
    fn draw(&self);
}

pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}

impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```
`Vec<Box<dyn Draw>>>` stands-in for any type inside a Box that has to implement the `Draw` trait. Generic types would not work here bc only 1 type could be called (Buttons for example) whereas at runtime traits allow for multiple concrete types
To add something to draw, simply declare a struct:
```
use gui::{Draw, Screen, Button};

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // code to actually draw a select box
    }
}

pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // code to actually draw a button
    }
}


fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No")
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
```
If a `Box::new()` call a type that does not implement the `Draw` trait it will not compile

## Object-oriented design pattern implementation: the *state pattern*
*state object* means a value has both an internal state and a private value that can only be updated through its public methods. Each method updates the state value which is `Some(<Box>)`
Example: a `Post` crate that allows to create a Post draft, wait for its review and publish it once it has been reviewed. The post cannot be published before, and its content will be empty until publication
```
use blog::Post;

fn main() {
    let mut post = Post::new();
    post.add_text("I ate a salad for lunch today");
    post.request_review();
    assert_eq!("", post.content()); // post content is still empty
    post.approve();
    assert_eq!("I ate a salad for lunch today", post.content()); // post has been published
}
```
Here we can only access the `Post` type, that has an internal value and three mathods that are the only way to update the instance state and content changes
definition of public struct Post:
```
pub struct Post {
    state: Option<Box<dyn State>>,
    content: String,
}

impl Post {
    pub fn new() -> Post {
        Post {
            state: Some(Box::new(Draft {})),
            content: String::new(),
        }
    }

    pub fn content(&self) -> &str {
        ""
    }
    
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }

    pub fn request_review(&mut self) {
        if let Some(s) = self.state.take() { // if state != none: take the value out of the option, put it in `s` and consume the current state ...
            self.state = Some(s.request_review()) // ... update and return a new state
        }
        // cannot do it in a one line operation as we need to get ownership of the `state` value
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
    // the method is only valid when cvalled on a `Box` holding the type
    // the method takes ownership of the `Box<Self>` (state value) to update the state of the `Post`
}

struct Draft {}
impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        Box::new(PendingReview {})
    }
}

struct PendingReview {}
impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }
}
```

# PATTERNS AND MATCHING (18)
Patterns are a syntax for matching against values. If the value does not fit the shape of the pattern, the code associated with the pattern will not run
## Use cases (18.1)
##### `match` arms
! must be exhaustive (all possible expressions must be accounted for. Catchall patterns like `_` help)
```
match VALUE { // match x {
    PATTERN => EXPRESSION, // 1 => println!("one"),
    PATTERN => EXPRESSION,
}
```
`_` will match anything but will never bind to a variable
##### Conditional `if let` expressions
Mainly is a shortcut for a `match` of only one case
Compatible with `else`, `else if` or even `else if let`
```
let favorite_color: Option<&str> = None;
if let Some(color) = favorite_color {
    println!("Using your favorite color, {}, as the background", color);
}
```
Shows the correct syntax but condition is not met.
Note that `if let` creates a scope, so be careful to shadowing
`if let` downside: exhaustiveness is not checked (ie `else` is not mandatory) which is a source of logical bugs
##### `while let` conditional loops
```
while let Some(top) = stack.pop() {
    println!("{}", top);
}
```
When `pop()` returns `None` the loop stops (pop is a `Vec` method that deletes last elem and returns `Some(new_last_elem)` or None` if empty).
##### `for` loops
```
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() { // enumerate produces a tuple from an iterator
    println!("{} is at index {}", value, index);
}
```
##### `let` statements
It is actually a pattern: `let PATTERN = EXPRESSION;`
Destructuring (here a tuple) is possible: `let (x, y, z) = (1, 2, 3);`. Note that `let (x, y) = (1, 2, 3);` will not compile
##### function parameters
`fn foo(x: i32) {` is a pattern
Destructuring is also possible:
```
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

## Refutability: whether a pattern might fail to match (18.2)
Patterns can either be:
- Irrefutable: ie they match for *any* possible value. `let x = 5;` -> x can hold every value possible. Function params, `let` statements, `for` loops can only accept irrefutable patterns bc otherwise programs could not work safely
- Refutable: ie some values are not handled. Ex: `if let Some(x) = val` if `val` equals `None` pattern will not match. `if let` and `while let` can only accept refutable patterns bc they are made to handle possible failure
Consequence: `let Some(x) = some_option_value;` cannot compile and would error `refutable pattern in local binding: 'None' not covered`. `if let` is nice when  we have a refutable pattern where an irrefutable pattern is needed
The other way other does not make sense and will not compile: `if let x = 5;` will error: `irrefutable if-let pattern`

## All the pattern syntax (18.3)
##### Matching literals
`match x { 1 => println!("one"), }`
##### Matching named variables
It is actually an issue bc match starts a new scope so var names will be shadowed
```
let x = Some(5);
let y = 10;

match x {
    Some(y) => println!("Matched, y = {:?}", y),
    // y is a new variable that can take any value, will match wil x (Some(5)) and will print y = 5
}

println!("at the end: x = {:?}, y = {:?}", x, y); // will output x = Some(5), y = 10
```
Solution: use *match guards* conditional (later section)
##### Multiple patterns and ranges of values
`|` syntax means *or*.
`...` allows to match an inclusive range of values. Only allowed for numeric values and `char` values.
```
match x {
    1 | 2 => println!("one or two"),
    2 ... 5 => println!("2 | 3 | 4 | 5"),
    _ => println!("anything else"),
}
```
##### Destructuring to break apart values
Patterns can be used to destructure structs, enums, tuples and references, even nested
complex example: `let ((feet, inches), Point {x, y}) = ((3, 10), Point { x: 3, y: -10 });`
###### Destructuring structs and enums
```
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };
    let Point { x, y } = p; // shorthand for let Point { x: a, y: b } = p; 
}
```
Can be combined with `match`:
```
match p {
    Point { x, y: 0 } => println!("On the x axis"), // matches for all x, when y: 0
    Point { x, y } => println!("Coucou"), // matches everything
}
```
###### Destructuring enums
```
enum Color { // tuple
   Rgb(i32, i32, i32),
   Hsv(i32, i32, i32)
}
enum Message {
    Quit,
    Move { x: i32, y: i32 }, // struct
    ChangeColor(Color), // enum
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::Quit => println!("Quit variant: no data to destructure."),
        Message::Move { x, y } => println!("move struct destructured"),
        Message::ChangeColor(Color::Hsv(h, s, v)) => println!("colors enum(tuple) destructured"),
    }
}
```
###### Destructuring references
When the pattern's value contains a reference, specify a `&` in the pattern to get the pointed value rather than the variable that hosts the ref
```
let points = vec![
    Point { x: 0, y: 0 },
    Point { x: 1, y: 5 },
    Point { x: 10, y: -3 },
];

let sum_of_squares: i32 = points
    .iter()
    .map(|&Point { x, y }| x * x + y * y)
    .sum();
```
Without the `&` in `|&Point { x, y }|`: compile error, cannot access Point values directly

##### Ignoring values in a pattern
`_` replaces 1 value and `..` replaces every values left (but listed ones)
Can be used in any pattern
###### Ignoring an entire value with `_`
- functions params: `fn foo(_: i32, y: i32) {` can be useful when implementing a trait if a param is not needed (clearer and removes the compiler warning)
- `match`: 
```
match (setting_value, new_setting_value) {
    (Some(_), Some(_), Some(_)) => () // useful to check to values are not `None`
    (first, _, third) => (),
}
```
###### Ignoring an unused variable by starting its name with `_`
Useful when prototyping: `let _x = 5;` prevents from a compiler warning.
Note that `_s` binds, `_` does not. Important:
```
let s = Some(String::from("Hello!"));
if let Some(_s) = s {
    println!("found a string");
}
println!("{:?}", s);
```
here the value has been moved into `_s` so compiler error when trying to print it. Would not have been moved using simply `_`
###### Ignoring remaining parts of a value pair with `..`
```
match numbers {
    (first, .., last) => {
```
However `(.., second, ..)` this pattern would result in an error as it is unclear.
##### Extra conditionals with *match guards*
Additional `if` in a `match` case, which is actually not a pattern
```
match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
```
Note: if `if` condition is not matched code will keep itering cases
Note2: a shadowing workaround as you can use outer variables in match guards
Match guard can be used with `|`:
```
let y = false;
match x {
    4 | 5 | 6 if y => println!("yes"),
```
that acts as `(4 | 5 | 6) if y` which means the `if` always applies to the result of the entire arm pattern
###### `@` bindings
Allows to create a variable that at the same time holds a value and tests it
```
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello { id: id_variable @ 3...7 } => println!("Found an id in range: {}", id_variable),
    // id value is both saved into id_variable AND checked on the 3..7 range
    Message::Hello { id: 10...12 } => println!("Found an id in another range"), // cannot access id value here
```
##### Legacy patterns: `ref` and `ref mut`
Mostly useless as of today bc Rust has been updated with abstraction but was useful when borrowing a variable, prevented the variable to be moved
`ref` creates a reference, it is the opposite of `&` in patterns: it binds to a `&` so that Rust does not try to moove a variable.
Old version working code:
```
let robot_name = &Some(String::from("Bors"));
match robot_name {
    &Some(ref name) => println!("Found a name: {}", name),
    None => (),
}
```
Today's changes a lot easier, in match first arm: `Some(name)`


# ADVANCED FEATURES (19)
## Unsafe Rust (19.1)
Allow it with `unsafe` before a block
The borrow checker etc still stands but enforces Rust to deal with memory at runtime. It gives 4 superpowers:
- Deref a raw pointer
- Call an unsafe fun / method
- Access / modify a mut static variable
- Implement an unsafe trait
Beware, it leaves the door open to problems due to memory unsafety, such as null pointer dereferencing
Made possible bc Rust prefer to reject valid programs rather than accept invalid ones so it has to allow a workaround ; to allow low-level programming such as directly interacting with the operating system or even writing your own operating system
Wrapping unsafe code in a safe abstraction prevents uses of `unsafe` from yourself or other users

##### Dereferencing a raw pointer
Raw pointers: `*const T` and `*mut T`. The `*` is not the dereference operator, it is part of the type name
In the context of raw pointers, *immutable* means that the pointer can’t be directly assigned to after being dereferenced.
Different from references and smart pointers, raw pointers:
- Are allowed to ignore the borrowing rules by having both immutable and mutable pointers or multiple mutable pointers to the same location (be careful to data races)
- Are allowed to be null
- Are not guaranteed to point to valid memory
- Do not implement automatic cleaning
Creating a raw pointer (safe code):
```
let mut num = 5;
let address = 0x012345usize;

let r1 = &num as *const i32; // created from a ref so guaranteed to be valid
let r2 = &mut num as *mut i32; // created from a ref so guaranteed to be valid 
let r3 = adress as *const i32; // created from an arbitrary memory location, unsecure

unsafe {
    println!("r1 is: {}", *r1);
    println!("r2 is: {}", *r2);
}
```
Creating a pointer does no harm, only accessing to its value can. In safe code it is not possible to read the data pointed at by a dereferenced raw pointer
Major raw pointer use case: interfacing with C code or building up safe abstractions that the borrow checker doesn’t understand
##### Calling an unsafe function or method
definition of a fun: `unsafe fn dangerous() { ... }` which must be called wrapped in an unsafe block
To create a safe abstraction over unsafe code just wrap the unsafe part in a fun
##### Using `extern` functions to call external code
extern facilitates the creation and use of a *Foreign Function Interface (FFI)* to interact with other languages
Extern implies the `unsafe` keyword
```
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```
The `"C"` calls the language corresponding *application binary interface (ABI)* (assembly)
Work the other way too, the following function is accessible in C code:
```
#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```
##### Access or modify a mutable static variable
*global variables* exist in Rust as *static* variables but can cause data races when mutable
```
static HELLO_WORLD: &str = "Hello, world!"; // type is &'static str

fn main() {
    println!("name is: {}", HELLO_WORLD);
}
```
Variable's type must be annotated. All static references have the `'static` lifetime so it can be implicit
Static variables have a fixed adress in memory and are mutable (!= constants).
Any code that reads or write from a static variable must be in an `unsafe` block bc it is difficult to ensure there is no data race in a globally accessible variable
Avoid mutable static variables whenever possible
##### Implementing an unsafe trait
A trait is `unsafe` when any of its methods has some invariant Rust compiler cannot verify. 
```
unsafe trait Foo { // Declaration
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}
```

## Advanced Lifetimes (19.2)
Every reference has a lifetime but most of the time Rust infers it. Advanced forms: *lifetime subtyping, lifetime bounds, trait object lifetimes*
##### Lifetime subtyping
Example of a parser:
```
struct Context<'a>(&'a str); // holds a string slice

struct Parser<'a> { // holds a reference to a Context instance
    context: &'a Context<'a>,
}

impl<'a> Parser<'a> {
    fn parse(&self) -> Result<(), &str> {
        Err(&self.context.0[1..]) // the string from its first byte
    }
}
```
For simplicity sakes Parser::parser always returns a dumb Error: the part of the slice that failed.
This example would not compile without these lifetime parameters. Even now it would not compile if this function was called:
```
fn parse_context(context: Context) -> Result<(), &str> {
    Parser { context: &context }.parse()
}
```
...as both the `Parser` instance and the `context` parameter would die at the end of `parse_context()` since the function took ownership of `context`.
Solution could be for the lifetime (lets say `<'s>` of the string slice and the return value of `parse_context()` to be the same... but it would not compile bc if `'s` lives shorter than `'c` (the lifetime of `Parse` and `Context` reference) then the reference to `Context` would not be valid. But *lifetime subtyping* point is to guarantee `'s` will live as least as long as `'c`! It is made by writing: `'s: 'c`. Full and valid result:
```
struct Context<'s>(&'s str);

struct Parser<'c, 's: 'c> {
    context: &'c Context<'s>,
}

impl<'c, 's> Parser<'c, 's> {
    fn parse(&self) -> Result<(), &'s str> {
        Err(&self.context.0[1..])
    }
}

fn parse_context(context: Context) -> Result<(), &str> {
    Parser { context: &context }.parse()
}
```
##### Lifetime bounds
Allows to add lifetime params as contraints on generic types.
Example: `struct Ref<'a, T>(&'a T);` would not compile as it is unsure `T` will live at least as long as `'a` so the right definition will be: `struct Ref<'a, T: 'a>(&'a T);` which is a *lifetime bound*
Other solution would be: `struct StaticRef<T: 'static>(&'static T);` since `'static` means the ref must live as long as the entire program (ie longer than `'a`). IE a type without any ref counts as `T: 'static` since it has no ref... so `T` will be constrained to types that have only `'static` references or no references
##### Trait object lifetimes
Trait objects consist of referencing a trait in order to use dynamic dispatch. But if the type implementing this trait has a lifetime, it can be specified with: `Box<Foo + 'a>`. Consequence: any implementer of the `Foo` trait that holds a reference inside must have its (the implementer) lifetime specified (pas compris)

## Advanced traits (19.3)
##### Associated types
For example `Iterator` has an associated type named `Item` which is the type of the iterated values
```
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```
Implementers of the `Iterator` trait will specify the concrete `Item` type:
```
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
```
###### Associated types versus generics
Hypothetical `Iterator` implementation using generics:
```
pub trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
```
Difference is generics would allow for multiple impl of `Iterator` for `Counter` (one for each type). Downside is for every `next()` call, type annotation is mandatory. With associated types, `Counter` is locked to only 1 concrete type implementation but `next()` knows its type. Other advantage, clarity:
```
trait GGraph<Node, Edge> { // definition of a trait using generics
    fn distance<N, E, G: GGraph<N, E>>(graph: &G, start: &N, end: &N) -> u32;
}

trait AGraph {  // definition of a trait using associated types
    type Node;
    type Edge;
    
    fn distance<G: AGraph>(graph: &G, start: &G::Node, end: &G::Node) -> u32;
}
```
###### Trait objects with associated types
Last advantage: the type can be defined in the trait objects when the associated types are used in a single argument (`fn traverse(graph: &AGraph<Node=usize, Edge=(usize, usize)>) {}`). (not sure about my understanding of this one)
##### Operator overloading and default type parameters
`<placeholderType=ContreteType>` is useful to specify the default type that can be overwritten (just by specifying another type in the implementation) for a generic type. Rust does not allow to create our own operators or to overload operators but the operation and corresponding traits listed in `std::ops` can be overloaded. Ex with operator overloading:
```
use std::ops::Add;

#[derive(Debug,PartialEq)]
struct Millimeters(u32);
#[derive(Debug,PartialEq)]
struct Meters(u32);

impl Add for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Millimeters) -> Millimeters {
        Millimeters(self.0 + other.0)
    }
}

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}

fn main() {
    assert_eq!(Millimeters(1) + Millimeters(4), Millimeters(5));
}
```
`Add` definition uses a default type:
```
trait Add<RHS=Self> {
    type Output;
    fn add(self, rhs: RHS) -> Self::Output;
}
```
So if `RHS` is not defined it will have its default concrete type (the same as `Self` = the type `Add` is implemented on)
##### Fully qualified syntax for disambiguation
Several traits can share the same method's name. If they are implemented on one type (which can directly have a method with this same name), by default the direct type method will be called. To prevent this, rather than doing the standard `receiver.method(args);` do: `<Type as Trait>::method(receiver, args);` or shortened `Trait::method(receiver)`
##### Supertraits to use one trait's functionality within another trait
A trait can use another trait's functionality like: `trait OutlinePrint: Display {`. If OutlinePrint trait is implemented on a type like `Point`, this type will have to implement `Display` to compile:
```
use std::fmt;

impl OutlinePrint for Point {}
impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```
##### The newtype Pattern to implement external traits on external types
The orpahn rule says traits can be implemented on a type only if either the type / trait is local to the crate. Exception: the *newtype pattern* involves creating a wrapper around a type (a one field tuple struct: `struct Meters(u32)`). A non-local trait can be implemented on the wrapper of a non-local type.
```
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```
Explanation: print the wrapper by accessing to the inner `Vec` via `self.0`.
Downside: the newtype does not have the methods of its concrete type, they would have to be implemented manually or implement the Deref trait to return the inner type (would have ALL methods which can be an issue too).

## Advanced types (19.4)
*newtypes*, *aliases*, `!` type, dynamically sized types
##### Newtype encapsulation pattern for type safety and abstraction
- Reduce confusion and enforce type safety by wrapping types, ex: `Meters` wrapping `u32` values is nice and it prevents compilation when calling plain `u32` or another `u32` wrapper
- Abstract away or hide some internal implementation details of a type ; restrict an available functionality of a public API. Ex: public `People` type could wrap a private `HarshMap<i32, String>` so API users would not know an id is assigned to each `People`.
##### Type synonyms with type aliases
`type Kilometers = i32;` they become synonym, ie they are interchangable (vs newtypes)
Main use case is to reduce repetition: `type Thunk = Box<dyn Fn() + Send + 'static>;` (to store a closure) -> `let f: Thunk = Box::new(|| println!("hi"));`
`type Result<T> = Result<T, std::io::Error>;` (!= `Result<T, E>`) is a Result shortcut that can be called afterwards as: `Result<usize>` rathan than `Result<usize, std::io::Error>`
##### Never type that never returns
`!` is the *empty type*, use cases:
- For a (*diverging*) function that never returns: `fn bar() -> ! {`
- For `match` patterns as all arms must return the same type but `!` works:
    ```
    let guess: u32 = match guess.trim().parse() {
        Ok(num) => num,
        Err(_) => continue, // continue has a `!` value, `!` coerces in any other type
    };
    ```
    When matching `continue`, no value is returned, the control is given back to the top of the loop
    Works with `panic!` too as it has a `!` type and ends the program (does not return a value)
- For loops: if a loop never ends (!= `break`), it has a `!` value
##### Dynamically sized types and the `sized` trait
Ex: `str` is a *DST*. `let s1: str = "DO NOT COMPILE";` bc its size is unknown, this is why a slice string `&str` (that stores both the position and len). Particular case as a `&T` usually only stores the adress of the `T`.
This is how Rust usually deals with *DST*: storing the variable adress and a `usize` of the elem size
Traits are DST: to use traits as objects, you must use a pointer: `&dyn Trait` or `Box<dyn Trait>` or `Rc<dyn Trait>`
`Sized` is a trait the is automatically implemented for everything whose size is known at compile time and for generic functions:
`fn generic<T>(t: T) {}` equals `fn generic<T: Sized>(t: T) {}` work only on compile time known sized elements unless `fn generic<T: ?Sized>(t: T) {}` is used (`&T` as the size may not be known)

## Advanced functions & closures (19.5)
##### Function pointers
Pass functions as args using `fn` (vs closures using `Fn`)
```
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);
    println!("The answer is: {}", answer);
}
```
Unlike closures, `fn` is a type, not a trait.
Fonction pointers implement `Fn, FnMut, FnOnce` traits so a function pointer can be passed as an argument. If the calling function uses a generic type and one of the closure traits, it will accept both `fn` and `Fn` (Exception: only `fn` accepted when interfacing with a language that does not have closures) as it is the case for `map()` (`map(ToString::to_string)` and `map(|i| i.to_string())` both work)

Side note: 
```
let list_of_numbers = vec![1, 2, 3];
let list_of_strings: Vec<String> = list_of_numbers
    .iter()
    .map(ToString::to_string)
    .collect();
```
Same results as:
```
enum Status {
    Value(u32),
    Stop,
}

let list_of_statuses: Vec<Status> =
    (0u32..20)
    .map(Status::Value)
    .collect();
```
##### Returning closers
Closures cannot be returned (`-> Fn(i32)` is forbidden) as it is not possible to return traits directly. That said, traits can have an implementation that returns the concrete type. This is forbidden:
```
fn returns_closure() -> Fn(i32) -> i32 {
    |x| x + 1 // returns a closure, not the concrete type
}
```
Would not compile, erroing: `the trait `std::marker::Sized` is not implemented` bc compiler does not know the closure's size. Solution is to store the closure in the heap:
```
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```

## Macros (19.6)
Macros refer to a family of features:
- *Declarative* macros with `macro_rules!`
- *Procedural* macros with 1+ of the following attributes: custom `#[derive]` macros ; attribute-like macros ; function-like macros
##### The difference between macros and functions
Macros are *metaprogramming* = write code that writes other code
Downside: macro definitions are more complex, less readable and maintainable than fun as we write Rust code that writes code
Macros can take a variable number of params
Fun are called at runtime but macros are expanded before. So for example traits can be implemented in macros, not functions
Macros must be defined/brought into scope before being called. Fun can be defined and called anywhere
##### Declarative macros with `macro_rules!` for general metaprogramming AKA *macros by example*
Most common. Similar to pattern matching: at compilation type, macros compare a value (associated to code) to patterns (structures of that source code) then run the code associated with the pattern
`let v: Vec<u32> = vec![1, 2, 3];`
Simplified definition of the macro `vec!`:
```
#[macro_export] // allows to bring the macro into scope
macro_rules! vec { // macro_rules! name_of_the_macro
    ($($x:expr), *) => { // pattern
        let myt temp_vec = Vec::new();
        $(
            temp_vec.push($x);
        )*
        temp_vec
    }
}
```
Here there is a one-armed pattern (`($($x:expr), *)`). if it matches: associated code is emitted, if not: error
`$($x:expr)` matches any expression separated by `,`; `*` means the pattern matches 0+ whatever precedes `*`
Body part: code contained into `$()*` is generated for each part that matches `$()` in the pattern, `$x` being replaced by the expr matched
So the code generated by the macro `vec![1, 2, 3];` is actually:
```
let mut temp_vec = Vec::new();
temp_vec.push(1);
temp_vec.push(2);
temp_vec.push(3);
temp_vec
```
##### Procedural macros for generating code from attributes
They take Rust code as an input, operate on it and outputs the resulted code. No pattern here. More like functions (*procedure* is a synonym of *function*)
Definition:
```
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```
Let's see  the possible attributes (all similar):
###### How to write a custom `derive` macro
Works on structs and enums
Point of these is to define a default trait implementation for a given function to avoid having to define it for each type the fun will be called from
Any `derive` macro name has the `_derive` suffix
Tutorial:
First, in *src/main.rs*:
```
use hello_macro::HelloMacro;
use hello_macro_derive::HelloMacro;

#[derive(HelloMacro)]
struct Pancakes;

fn main() {
    Pancakes::hello_macro();
}
```
and add to the *Cargo.toml* file:
```
[dependencies]
hello_macro = { path = "../hello_macro" }
hello_macro_derive = { path = "../hello_macro/hello_macro_derive" }
```
Create the first called crate: `cargo new hello_macro --lib`
Filename: *src/lib.rs*
```
pub trait HelloMacro {
    fn hello_macro();
}
```
Create the second called crate: `cargo new hello_macro_derive --lib`
File: *hello_macro_derive/Cargo.toml*:
```
[lib]
proc-macro = true // optional as it is the standard API to read abd manipulate Rust code

[dependencies]
syn = "0.14.4" // parses Rust code from a string into a data structure we can operate on
quote = "0.6.3" // reverse syn struct into Rust code
```
Define the macro in the *hello_macro_derive/src/lib.rs* file:
```
extern crate proc_macro;

use crate::proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream { // name convention is to match the trait name
    let ast = syn::parse(input).unwrap(); // Construct the struct from code string (returns a DeriveInput)
    impl_hello_macro(&ast) // Build the trait implementation
}
```
Note that `unwrap()` is necessary (to panic if needed) as procedural macro return `TokenStream` and not `Result`
Implement the `HelloMacro` trait in the *hello_macro_derive/src/lib.rs* file:
```
fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}", stringify!(#name));
            }
        }
    };
    gen.into()
}
```
See `quote!` doc for enhanced possiblilities (https://docs.rs/quote/0.6.11/quote/)
And it is done: `HelloMacro` trait has been included and implemented from `#[derive(HelloMacro)]`, `pancakes` does not have to implement it
###### Attribute-like macros
Works the same way custom `derive` macros work. Works on functions too.
Allows to create attributes. Ex `#[route(GET, "/")]` that would be defined like:
```
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```
first `TokenStream`: `GET, "/"` ; second one: the function name and body (here `fn route...`) 
###### Function-like macros
Define macros that work like function calls: `let sql = sql!(SELECT * FROM posts WHERE id=1);` and its signature would be:
```
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

# APPENDIX (21)
## Operators and symbols (21.2)
##### Operators
- `ident!` followed by `()/{}/[]` - macro expansion
- `&expr, &mut expr` - Borrow
- `&type, &mut type, &'a type, &'a mut type` - Borrowed pointer type
- `!expr` - Bitwise or logical complement
- `expr & expr` - Bitwise AND 
- `var &= expr` - Bitwise AND and assignment
- `expr ^ expr` - Bitwise exclusive OR
- `expr | expr` - Bitwise OR
- `expr << expr` - Left-shift
- `expr >> expr` - Right-shift
- `*expr` - Dereference
- `*const type, *mut type` - Raw pointer 
- `.., expr.., ..expr, expr..expr` - Right-exclusive range
- `..=expr, expr..=expr` - Right-inclusive range
- `ident @ pat` - Pattern binding
- `pat | pat` - Pattern alternatives
- `expr?` - Error propagation
##### Symbols
- `...u8, ...i32, ...f64, ...usize, etc.` - Numeric literal of specific type
- `"..."` - String literal
- `r"...", r#"..."#, r##"..."##, etc.` - Raw string literal, escape characters not processed
- `b"..."` - Byte string literal; constructs a [u8] instead of a string
- `br"...", br#"..."#, br##"..."##, etc.` - Raw byte string literal, combination of raw and byte string literal
- `'...'` - Character literal
- `b'...'` - ASCII byte literal
- `|...|expr` - Closure
- `!` - Always empty bottom type for diverging functions 

## Derivable traits (21.3)
the `derive` attrb appliable to a struct / enum definition implements traits. List of `derive` traits available: 
##### `Debug` for programmer output
- What it does: enable to print instances of a type for debug
- Operators / methods made available: `:?` within `{}` operator
- Trait required for: `assert_eq!` macro
##### `PartialEq` and `Eq` for equality comparisons
- What it does: compare instances of a type. `Eq` signals that for every value of a type the value is equal to itself.
- Operators / methods made available: `==` `!=` operators, `eq()` method
- Implementation requirments: `Eq` requires `PartialEq` trait and even then, some types cannot implement `Eq` (floating point types)
- Trait required for: `Eq` required for keys in a `HashMap<K, V>`
- Notes: `PartialEq` implements `Eq` ; `PartialEq` on structs: equal only if all fields are. On Enums: only equal to the compared variant
##### `PartialOr` and `Ord` for ordering comparisons
- What it does: compare instances of a type for sorting purposes. `Ord` allows to know that a valid ordering exists between 2 values
- Operators / methods made available: `>` `<` `<=` `>=` operators, `partial_cmp()` method which returns an `Option<Ordering>` ; `cmp()` method which returns an `Ordering`
- Implementation requirments: `Ord` requires both `PartialOrd` and `Eq`.
- Trait required for: `gen_range` method from `rand` crate. When storing values in a `BTreeSet<T>` (`Ord`)
- Notes: on structs, compares each field ; on enums, earlier variants are considered less.
##### `Clone` and `Copy` for duplicating values
- What it does: `Clone` to create a deep copy of heap data ; `Copy` to duplicate only the bits on the stack.
- Operators / methods made available: `clone()` method that calls clone on each parts of the type, so each part must implement `Clone` 
- Implementation requirments: `to_vec()` method on a slice
- Trait required: `Clone` is required to implement `Copy`
##### `Hash` for mapping a value to a value of fixed size
- What it does: takes an instance of a type of arbitrary size and map the instance to a value of fixed 
- Operators / methods made available: `hash()` method, called on each part of the `hash()` calling type so each part must implement it
- Trait required for: storing keys in a `HashMap<K, V>` to store data efficiently
##### `Default` for default values
- What it does: creates a default value for a type
- Operators / methods made available: `default()` fun, on each part so each part must implement the trait
- Trait required for: `unwrap_or_default` method on `Option<T>` instances bc if `None` it will return the result of `Default::default()`

## Useful development tools (21.4)
- `$ cargo fmt` of the `rustfmt` tool: auto-formats to standard Rust style (beta, install it via `rustup component add rustfmt` which installs `cargo-fmt` too)
- `cargo fix` of the `rustfix` tool: fixes your code by applying compiler correction suggestions
- `cargo clippy` of the `clippy` linter (beta, install: `rustup component add clippy`): checks code to suggest corrections of common mistakes as `std::f64::consts::PI` instead of `3.1415`
- IDE Integration (VS:Code) using the Rust Language Server: `rustup component add rls rust-analysis rust-src` to gain autocompletion, definition jumps, inline errors etc

## Editions (21.5)
Edition is found in the *Cargo.toml* file
Rust language and compiler are updated every 6 weeks but new *editions* are pushed (in a 6-week update) every 2/3 years into a clear package, with fully updated documentation and tooling. Good and clean and fully updated rally point.
`cargo fix --edition` upgrades code to a new edition

## Newest features (21.7)
##### Field init shorthand
`struct, enum, union` named fields can now be compacted:
```
struct Person {
    name: String,
    age: u8,
}

let peter = Person { name: name, age: age }; // full initialisation
let portia = Person { name, age }; // new sugar initialisation
```
##### Return from loops
`break` can now be used to return from loops
```
let result = loop {
    counter += 1;

    if counter == 10 {
        break counter * 2; // break returns 20
    }
};
```