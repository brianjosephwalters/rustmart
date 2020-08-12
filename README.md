# Building a Web Application using Rust with WASM
[Single Page Applications using Rust](http://www.sheshbabu.com/posts/rust-wasm-yew-single-page-application/)

*  Compile Rust code to WASM (Web Assembly), which runs on the browser.
*  Relies on the [Yew](https://yew.rs/docs/) framework


### To Run:
Using `cargo-make` there are two tasks:
*  `cargo make build` - builds the wasm files to the ./static/wasm folder.
*  `cargo make serve` - serves the ./static folder, in particular the ./static/index.html file.

### Components
*  Uni-directional data passing
*  Composable into larger components
*  `Props` - pass data and callbacks into child components.
*  `State` - Manipulate the state of the local component.
*  `AppState` - Manipulate the global state.

Component re-rendering occurs on:
*  Parent component is re-rendered
*  `Props`, `State`, or `AppState` changes.