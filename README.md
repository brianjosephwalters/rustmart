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

### Building a Home Page
Implements a `Component` for a `Home` struct.
*  Component has Messages and Properties
*  Boilerplate for create(), update(), change().
    * What's the difference between update and change?
*  view() returns HTML.  Note the inline html tags.

`html!` is similar to JSX
*  `create()` sets the model state when the component is created.
*  `view()` is called when componet is rendered.


### Adding a Cart (interactions)
*  We `clone` the Product when it is added to the cart.
*  We create a `link` that allows us to register callbacks, which can trigger the `update()` lifecycle.
*  `update()` is where the `State` is changed and side-effects (i.e., network requests) are called.
    *  All of the possible actions are defined in the `Message`.
    *  Returning `true` re-renders the component.

* I'm not sure what `move` is doing in this context:
```
<button onclick=self.link.callback(move |_| Msg::AddToCart(product_id))>
```
