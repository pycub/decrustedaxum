# Axum
axum is super light and is really a router. Let you extract wanted information from the handlers.

Axum really lean a lot on the rest of the ecosystem:
- heavily on top of hyper
- uses tower for middleware system
- uses tracing
- matchit for route matching
- tokio runtime


# Routing and Handling
- Create a new router (primary entrypoint of the application)
- handlers are asynchronous functions
- `post, get, patch, put, ...` http method matcher (you can create yourself)
- `Json(payload): Json<CreateUser>` a pattern matching to destruct payload from the Json tuple struct.
-  Having tuples in the return of the handlers, (status_code, body) is considred. However, it is not mandatory! There are many ways you can return in the handler.
- For querystrings you can do `Query(params): Query<HashMap<String, String>>` and then `params.get("foo")` to get the param.

# Starting server
- create socker address
- and bind the socket with the axum server
- write `.serve` to actually listen for requests and respond them
- Each connection gets owned service
- serve basically never returns
- create connection for service, route each request to its own service
- What is service? Tower service


# Tower crate

## Tower service trait
Tower service trait is a trait that takes a request and processes asynchronously and return the result. These are not tied to HTTP.

```rs
pub trait Service<Request> {
    type Response;
    type Error;
    type Future; Future
    where
        <Self::Future as Future>::Output == Result<Self::Response, Self::Error>
    
    fn poll_ready(
        &mut self,
        cx: &mut Contxt<'_>
    ) -> Poll<Resut<(), Self::Error>>;
    fn call(&mut self, req: Request) -> Self::Future;
}
```

- axum does this effort of converting axum handler; a very simple function to Twoer service that takes request and returns response.
- There is a rule for what can be a axum handler:
    - It must return a `Fut`
    - `Fut::Output` must be a `IntoResponse`
    - Every argument except the last one must implement `FromRequestParts` (`FromRequestParts` has not access to the request body)
    - The last argument must implement `FromRequest` (Only `FromRequest` has access to the request body)
- Using `debug_handler` can help you debug handlers
- Although your handler return type is clearly `impl IntoResponse`, Rust only let you to have one single return type.
    - Below code CAN NOT be compiled as the return type of the function is not the same.
    ```rs
    async fn foo(...) -> impl IntoResponse {
        let failed = true;

        if failed == true {
            return StatusCode::200_OK
        }

        let user = User {
            id: 2,
            name: "GholamReza"
        }

        return (StatusCode::200_OK, Json(user))
    }
    ```
    - Solution, using the `Response` type and manually call `.into_response()` for the return values
    ```rs
    async fn foo(...) -> Response {
        let failed = true;

        if failed == true {
            return StatusCode::200_OK.into_response();
        }

        let user = User {
            id: 2,
            name: "GholamReza"
        }

        return (StatusCode::200_OK, Json(user)).into_response();
    }
    ```

