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

### Fetching Data
Yew exposes common browser APIs such as fetch, localstorage, etc through services.
We need the `anyhow` and `serde` crates to use these services.
    * `serde` provides Serialization classes.

*better design:*
First, we are moving the data into a json file to simulate a backend service.
Then, we move the shared data types to a types.rs file.
Next, we build an API class to handle requesting data.
Finally, we clean up.

*  FetchService API requires us to build a request out of the request data and a callback function.
    * This returns a FetchTask.  If the FetchTask is dropped (gc'ed, del), the connection is closed.  So store the response on the component that will call it.
*  We create `Message`s for the Outgoing updates and the sucess/error incoming updates.
*  We use the "get" message when the component is created.
    *  This uses the `link`, which is still a bit unclear to me.
    ```
        link.send_message(Msg::GetProducts);
    ```
*  We can track the success/error paths on the request and create separate Messages for each.  The component would need to have state that can be updated, which in turn gets applied in the `view()` lifecycle.
*  From some debugging around the `view()` method, it seems like only one `html!{}` macro can be evaluated per path.

### Reusable Components

*  In Components, often use _ if there is a feature the component won't implement, like messages or links.
*  Properties (`Props`) are the input to the Component.  This is both the data and any callback.
*  Dumb components don't hold their own state, but call back to the parent component to update it's state.
    * We use either the `emit` or `reform` methods on the callback.
    * `emit` just calls the callback function.
    * `reform`  seems to change the type.


