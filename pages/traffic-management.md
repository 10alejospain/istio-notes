| Property | Description |
| -------- | ----------- |
| uri | Matches the request URI to the specified value |
| scheme | Match the request schema (HTTP, HTTPS, …) |
| method | Match the request method (GET, POST, …) |
| authority | Match the request authority headers |
| headers | Match the request headers. Headers must be lower-case and separated by hyphens (e.g., x-my-request-id). Note, if we use headers for matching, other properties are ignored (uri, scheme, method, authority) |