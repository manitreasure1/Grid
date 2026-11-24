# GRID
A modern php framework (json in, json out)
```php
Grid::get("/", function(){
});
```
supports  `GET`, `POST`, `PATCH`, `PUT`, `DELETE`
```txt
Grid::method(url, callback);
```
> each request would contain traceparent(header) and trace_id
## Json Request
```php
class User{
    public string $name;
    puplic ?string $email;
}
Grid::post("/", function(User $user){
});
```
**Grid** would make the request expecting `name` with optional `email`
Fields would be casted into specified data type is failed execption would be raise
## Path and Query
```php
Grid::get("/{userId}", function(string $userId, string $level){
});
```
Paramter name in the url `{}` are use for path request, if name exist in function parameter and not included in the url, grid treat it as query parameter
### HTTP VERBS Constraint
`DELETE` and `GET` forbids body
## Response and Execption
```txt
Response::STATUS_CODE([BODY], DISPATCHER);
```
The response `class` with `status code` and optional `dispatcher`
```php
class UserResponse{
    public string $userId;
    public string $level;
}
Grid::get("/{userId}", function(string $userId, string $level){
    return Response::OK(["userId" => $userId, "level"=> $level], UserResponse::class);
});
```
response supports chaining header
```php
->header("Location", route("get-user", ["userId" => $userId]));
```
`/{userId}` is expected to return object of the shape of `UserResponse`
if fields specified in `UserResponse` are not given and exception would be thrown
`Response` ignore extra fields when not specified in the dispatcher

### Response constraints
`NO_CONTENT` accepts no body
`HTTPException` class
```txt
HTTPException::STATUS_CODE(optional problem detail error)
```
### problem detail
```txt
string $title;
string $detail;
int $status;
?string $instance;
?array $extension;
```
```php
Grid::get("/{userId}", function(string $userId, string $level){
    if($level === "5"){
        throw HTTPExecption::BadRequest(new ProblemDetail(...));
    }
});
```
## blueprint
For mudules
```php
class AuthRoutes extends ApiGroup
{
    public static array $routes = [];
}
AuthRoutes::get("/me", #[Auth] function () {
    return Response::OK();
});
```
### Register blueprint in Grid
```txt
Grid::register_group(prefix, blueprint)
```
```php
Grid::register_group("/auth", AuthRoutes::class);
```
## Attri as Middleware
a dev can write a custom attri and use on route
```php
AuthRoutes::get("/me", #[Auth] #[Role] function () {
    return Response::OK();
});
```
## Attributes for request
```php
#[Header("X-Tenant-ID")] string $tenantId,
#[Cookie("session_token")] ?string $token = null
#[Path(class)]
```

## Middleware

```php
Grid::register_group("/v1/auth", AuthRoutes::class)
    ->middleware([RateLimitMiddleware::class, AuditLogMiddleware::class]);
```

## Exception Handler
### Local `for business logic`
```php
UserRoutes::post("/transfer", function (TransferRequest $req) {

    account_transfer($req);
})->catch(InsufficientFundsException::class, function (InsufficientFundsException $e) {
    throw HTTPException::UnprocessableEntity(
        new ProblemDetail(
            title: "Transfer Failed",
            detail: $e->getMessage(),
            status: 422,
            extension: ["balance" => $e->getCurrentBalance()]
        )
    );
});
```

### Global
```php
Grid::mapException(DatabaseNotFoundException::class, function($e) {
    return ProblemDetail::notFound($e->getMessage());
});
```

## Logging

```php
Logger->info(....);
Logger->debug(...);
```

## Testing

```php
$response = GridTest::post('/auth/login', ['email' => 'test@example.com'])
    ->assertStatus(200)
    ->assertJsonStructure(UserResponse::class);
```
