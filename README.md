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

### Styling
Yew allows class attributes and inline styles.
Pretty straight forward.
*  Also learned that there can only be one root html element in `html!` macro.

### Routing
SPAs mimic multi-page websites using routers.  We'll use the yew-router.  It handles query parameters, path variables, etc.  And we'll build a new homepage, since the top-level index.html page will now need a router outlet, into which other components can be loaded.  In our case, it's really just a "routing layer" above the Home component.

*  Build a Route stuct containing Components.  Annotate the Component with its route `#[to = "/"]`
*  In the App component, create initial renderings for the Routes configured in `route.rs`. Map each of these routes to the html! macro that will render it.
```
let render = Router::render(|switch: Route| match switch {
    Route::HomePage => html! {<Home/>},
});
```
*  In the html! macro for the App, use the Router component to build out the component.  Normally, the Router component would be inside the portion of the page that changes, while elements like headers etcs, are around it.

### Detail pages
We'll want to navigate to a details page based on the product's id.  We use yew_router's RouterAnchor tag to supply the routing configuration element for the html! macro.  
*  Note the use of `classes` instead of `class`.

We need to create a data route for particular products in the API.  
*  For now, this will just pull json out of static files.

Building out the Product pages is similar to Home:
*  The Props is the product id, corresponding to the route.
*  The State is similar to before: GetProduct, error, onload
*  The Task will store the api call.
*  Link is used to make the api call. 


### State Management
We need to move the State held in the Home component to a parent location so that it can be updated by the Details component.  We'll also create a top-level nav bar that can hold the current cart total.  This involves a bit of refactoring.

State sharing stragies:
*  Hoist the state to a parent component shared by both
    *  Results in "prop drilling" and prevents components being used in contexts outside of that heirarchy - why are they separate components, then?  Seems like bad design.

*  Move state to a global app state.
    *  Recommend using Agents for pub/sub.
    *  No good solution from Yew right now.

*  Use of the `change()` lifecycle method in navbar
    -  When the props sent from the parent changes, it needs to be changed here as well.
*  Use of both `update()` and `change()` in atc_button
    -  The button needs to receive both the callback function and the product to pass to the callback
    -  Really just a component to create button, but we can use it in multiple places.
    -  Product Card usage is an example of a component using a component (both dumb components), passing the properties that the parent component received from *its* parent.
        *  Note that both the data *and* the callback are clones.
    -  Product Details usage is a smart component using the dumb component.


