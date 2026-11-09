# Day 02 — URL Parameters & HTTP Request Headers

## Overview

On Day 2, I learned how URL parameters and HTTP request headers work, how Express.js accesses them, and how to configure them using Postman.

The focus was on understanding how clients send additional information to servers through HTTP requests.

## 1. Path Parameters

### What are Path Parameters?

Path parameters are dynamic values passed directly inside the URL path to identify a particular resource.

**Example:**

```http
GET /listings/123
```

Here:
- `/listings` identifies the resource type.
- `123` identifies a particular listing.
- `123` is the path parameter.

### Defining Path Parameters in Express

```javascript
app.get('/listings/:id', (req, res) => {
    console.log(req.params.id);
});
```

Explanation:
- `:id` defines a dynamic path parameter.
- `req.params` contains all path parameters.
- `req.params.id` retrieves the value of the ID parameter.

**Example request:**

```http
GET /listings/123
```

**Output:**

```javascript
{
    id: "123"
}
```

### Multiple Path Parameters

Express supports multiple path parameters in a single route.

```javascript
app.get('/users/:userId/posts/:postId', (req, res) => {
    console.log(req.params);
});
```

Request:

```http
GET /users/101/posts/505
```

Output:

```javascript
{
    userId: "101",
    postId: "505"
}
```

### Important Points

- Path parameters are defined using `:` in Express routes.
- They are accessed using `req.params`.
- Path parameter values are received as strings.
- If numerical operations are required, convert them using `Number()` or `parseInt()`.

---

## 2. Query Parameters

### What are Query Parameters?

Query parameters are key-value pairs added to the URL after the `?` symbol.

They are commonly used for:
- Filtering
- Searching
- Sorting
- Pagination

**Example:**

```http
GET /listings?location=Mumbai&price=5000
```

Here:

| Component | Meaning |
|---|---|
| `?` | Starts the query string |
| `location=Mumbai` | First query parameter |
| `price=5000` | Second query parameter |
| `&` | Separates multiple parameters |

### Accessing Query Parameters in Express

```javascript
app.get('/listings', (req, res) => {
    console.log(req.query);
});
```

For the request:

```http
GET /listings?location=Mumbai&price=5000
```

Output:

```javascript
{
    location: "Mumbai",
    price: "5000"
}
```

### Multiple Query Parameters

Example:

```http
GET /listings?location=Mumbai&minPrice=1000&maxPrice=5000
```

Output:

```javascript
{
    location: "Mumbai",
    minPrice: "1000",
    maxPrice: "5000"
}
```

### Practical Example: Price Filtering

Suppose a user wants to find listings between ₹1,000 and ₹5,000.

Request:

```http
GET /listings?minPrice=1000&maxPrice=5000
```

Express:

```javascript
const { minPrice, maxPrice } = req.query;
```

MongoDB filtering:

```javascript
const listings = await Listing.find({
    price: {
        $gte: Number(minPrice),
        $lte: Number(maxPrice)
    }
});
```

Explanation:

- `$gte` — Greater than or equal to.
- `$lte` — Less than or equal to.

**Important:** Sending query parameters does not automatically implement filtering. The backend must read those parameters and apply the appropriate filtering logic.

### Important Points

- Query parameters begin after `?`.
- Multiple parameters are separated using `&`.
- They follow the `key=value` format.
- They are accessed using `req.query`.
- Query parameter values are generally received as strings.

---

## 3. Difference Between `req.params` and `req.query`

Both are used to retrieve values from the URL, but they serve different purposes.

### Example

```http
GET /listings/123?location=Mumbai
```

Express route:

```javascript
router.get('/:id', (req, res) => {
    const { id } = req.params;
    const { location } = req.query;

    console.log(id);
    console.log(location);
});
```

Output:

```javascript
req.params = {
    id: "123"
};

req.query = {
    location: "Mumbai"
};
```

### Comparison

| `req.params` | `req.query` |
|---|---|
| Retrieves path parameters | Retrieves query parameters |
| Defined using `:name` in the route | Passed after `?` in the URL |
| Accessed using `req.params` | Accessed using `req.query` |
| Commonly identifies a resource | Commonly used for filtering and searching |

### Quick Revision

```http
GET /listings/123?location=Mumbai
```

- `123` → `req.params.id`
- `Mumbai` → `req.query.location`

---

## 4. HTTP Request Headers

### What are Request Headers?

Request headers are key-value pairs sent by the client to the server along with an HTTP request.

They provide additional information about the request, such as:

- Expected response format
- Request body format
- Authentication credentials
- Cookies
- Client information

### Example

```http
GET /listings HTTP/1.1
Host: localhost:8080
Accept: application/json
Authorization: Bearer abc123
```

### Important HTTP Headers

| Header | Purpose |
|---|---|
| `Host` | Specifies the target host |
| `Accept` | Specifies acceptable response formats |
| `Content-Type` | Specifies the format of the request body |
| `Authorization` | Carries authentication credentials |
| `Cookie` | Sends stored cookies to the server |
| `User-Agent` | Identifies the client software |
| `Content-Length` | Specifies the request body size in bytes |
| `Accept-Encoding` | Specifies supported response compression formats |
| `Connection` | Controls connection behavior |

### Request Headers vs Response Headers

| Request Headers | Response Headers |
|---|---|
| Sent by the client | Sent by the server |
| Provide information about the request | Provide information about the response |
| `Accept` | `Content-Type` |
| `Authorization` | `Set-Cookie` |
| `Cookie` | `Cache-Control` |

### Content-Type vs Accept

These two headers are frequently confused.

**Content-Type**

Specifies the format of the data being sent in the request body.

```http
Content-Type: application/json
```

Meaning: The client is sending JSON data to the server.

**Accept**

Specifies the format of the response the client wants to receive.

```http
Accept: application/json
```

Meaning: The client expects a JSON response.

### Easy Way to Remember

- `Content-Type` → What format am I sending?
- `Accept` → What format do I expect in return?

### Example

```http
POST /users
Content-Type: application/json
Accept: application/json
```

Here:
- The client sends JSON data.
- The client expects a JSON response.

---

## 5. Custom Request Headers

Apart from automatically generated headers, developers can also define their own custom headers.

**Example:**

```http
X-Client-Name: Wanderlust
X-API-Version: v1
```

These are custom headers whose names and values are defined by the developer.

### Accessing Custom Headers in Express

```javascript
app.get('/test', (req, res) => {
    console.log(req.headers['x-client-name']);
});
```

Output:

```text
Wanderlust
```

### Important Points

- Custom headers can be added through Postman.
- Express accesses request headers using `req.headers`.
- Header names are case-insensitive.
- Express normalizes header names to lowercase in `req.headers`.
- Adding a custom header does not automatically make the backend process it. The backend must explicitly read and use it.

---

## 6. Practical Work in Postman

### A. Sending Query Parameters

1. Open Postman.
2. Select the GET method.
3. Enter the API endpoint.

```http
http://localhost:8080/listings
```

4. Open the Params tab.
5. Add the following parameters:

| Key | Value |
|---|---|
| `minPrice` | `1000` |
| `maxPrice` | `5000` |

Postman automatically generates:

```http
http://localhost:8080/listings?minPrice=1000&maxPrice=5000
```

**Observation:** Postman sends the query parameters, but actual filtering depends on backend implementation.

### B. Inspecting Request Headers

1. Open an existing Postman request.
2. Navigate to the Headers tab.
3. Inspect the available request headers.

Headers observed during practical work included:

| Header | Purpose |
|---|---|
| `Cookie` | Sends the session cookie |
| `Postman-Token` | Postman-generated request identifier |
| `Content-Type` | Indicates request body format |
| `Content-Length` | Request body size |
| `Host` | Target server host |
| `User-Agent` | Identifies Postman's runtime |
| `Accept` | Acceptable response formats |
| `Accept-Encoding` | Supported compression formats |
| `Connection` | Connection behavior |

### C. Understanding Session Cookies

During practical work, the following header was observed:

```http
Cookie: connect.sid=...
```

This is the session cookie used by the Wanderlust application.

The client sends this cookie to the server so that the server can identify the associated session.

---

## 7. Important Interview and ESE Questions

**Q1. What are path parameters?**

Path parameters are dynamic values passed directly in the URL path to identify a particular resource.

**Q2. What are query parameters?**

Query parameters are key-value pairs passed after the `?` symbol in a URL. They are commonly used for filtering, searching and pagination.

**Q3. What is the difference between `req.params` and `req.query`?**

`req.params` retrieves dynamic values from the URL path, whereas `req.query` retrieves key-value pairs from the URL query string.

**Q4. What are HTTP request headers?**

Request headers are key-value pairs sent by the client to provide additional information about an HTTP request.

**Q5. What is the difference between Content-Type and Accept?**

Content-Type specifies the format of the request body, whereas Accept specifies the response formats acceptable to the client.

**Q6. How do you access request headers in Express?**

Using `req.headers`.

**Q7. Can an Express route contain multiple path parameters?**

Yes. For example:

```javascript
app.get('/users/:userId/posts/:postId', handler);
```

**Q8. Does adding query parameters automatically filter database records?**

No. The backend must read the query parameters and implement the filtering logic.

---

## 8. Quick Revision Cheat Sheet

| Concept | Express property / Syntax |
|---|---|
| Path parameters | `req.params` |
| Query parameters | `req.query` |
| Request headers | `req.headers` |
| Path parameter definition | `:id` |
| Query string begins with | `?` |
| Multiple query parameters separated by | `&` |
| Request body format | `Content-Type` |
| Expected response format | `Accept` |

### Final Takeaways

By completing Day 2, I learned:

- How path parameters identify specific resources.
- How query parameters pass additional information through URLs.
- The difference between `req.params` and `req.query`.
- How HTTP request headers work.
- The difference between request and response headers.
- The difference between `Content-Type` and `Accept`.
- How to inspect query parameters and request headers using Postman.
- How Express accesses custom request headers.

**Day 02 Completed — Postman Learning Journey.**
