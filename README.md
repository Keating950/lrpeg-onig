
![crates.io](https://img.shields.io/crates/v/lrpeg.svg)
[![CI](https://github.com/seanyoung/lrpeg/workflows/test/badge.svg)](https://github.com/seanyoung/lrpeg/actions)
[![license](https://img.shields.io/github/license/seanyoung/lrpeg.svg)](LICENSE)

# Left Recursive Parsing Expression Grammar (PEG)

lrpeg allows left recursive rules, and uses ratpack parsing for speed.

The existing PEG parser generators for rust do not allow left recursion,
which makes it very awkward to write grammars. It is possible to write
a PEG parser generator which allows for
[left recursion](http://www.vpri.org/pdf/tr2007002_packrat.pdf),
just as [python now uses](https://medium.com/@gvanrossum_83706/left-recursive-peg-grammars-65dab3c580e1).

See [IRP Grammar](https://github.com/seanyoung/lrpeg/blob/main/lrpeg-test/src/irp.peg) for a complete lrpeg grammar for
[IRP](http://hifi-remote.com/wiki/index.php?title=IRP_Notation).

## How to use lrpeg

Add lrpeg to your Cargo.toml in build-dependencies:

```
[build-dependencies]
lrpeg = "0"

[dependencies]
regex = "1"
unicode-xid = "0.2"
```

Now add a `build.rs` to the root of your project, containing:

```
use std::path::PathBuf;

fn main() {
    lrpeg::process_files(&PathBuf::from("src"));
}
```
Write your peg grammar, and put it in a file which ends with `.peg`, for example `src/calculator.peg`:

```
expr <- term "*" term
    / term "/" term
    / term;

term <- term "+" term
    / term "-" term
    / "(" expr ")"
    / num;

num <- r"[0-9]+";
```
When your run `cargo build`, `src/calculator.rs` will be generated. You need to include this module in
your project, and then you can instantiate the PEG like so:

``` rust
mod calculator;

fn main() {
   let mut parser = calculator::PEG::new();

   match parser.parse("10 + (100 * 9)") {
      Ok(s) => println!("parse tree: {}", s.print_to_string()),
      Err(pos) => println!("parse error at offset {}", pos);
   }
}
```

## How lrpeg is bootstrapped

First of all, we need to bootstrap the PEG. We do this with a very simple
lalrpop LR(1) PEG grammar. This grammar has some limitations, but it is
good enough.

## TODO

- More tests
- Better parse error information (now only the error offset is returned)
- Better documentation
- Make generator into rust macro
- Detect unused rules
- Detect unreachable alternatives
