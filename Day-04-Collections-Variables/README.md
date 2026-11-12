# Day 04 – Collections, Collection Configuration & Collection Runner

## Overview

Day 04 focuses on organizing API requests using Postman Collections, configuring reusable settings, and executing multiple requests using Collection Runner.

## 1. Collections

A **Collection** is a group of related API requests stored together in Postman.

Instead of creating and managing API requests separately, we can organize them inside a collection.

### Why do we use Collections?

- Organize related API requests.
- Save and reuse requests.
- Share requests with team members.
- Execute multiple requests together.
- Maintain and test APIs efficiently.

### Collection vs Folder vs Request

| Component | Description |
|---|---|
| Collection | Groups related API requests. |
| Folder | Organizes requests inside a collection. |
| Request | Represents an individual API call. |

### Example: Wanderlust API Collection

```text
air_bnb (Collection)
│
├── Listings (Folder)
│   ├── Get All Listings
│   ├── Get Listing By ID
│   ├── Create Listing
│   ├── Update Listing
│   └── Delete Listing
│
├── Users (Folder)
│   ├── Signup
│   ├── Login
│   └── Logout
│
└── Reviews (Folder)
    ├── Create Review
    └── Delete Review
```

This structure organizes requests according to application features.

---

## 2. Collection-level Configuration

Postman allows us to configure common settings at the collection level.

These settings can be reused by requests inside the collection.

### A. Authorization

Authorization can be configured once at the collection level.

Requests inside the collection can select:

`Auth → Inherit auth from parent`

This allows them to inherit the collection's authorization configuration instead of configuring it individually.

**Note:** A request can override the inherited authorization configuration with its own settings.

### B. Variables

Collection variables store reusable values that can be accessed by requests inside the collection.

Example:

```text
baseUrl = http://localhost:8080
```

We can use the variable in request URLs:

```text
{{baseUrl}}/listings
{{baseUrl}}/login
{{baseUrl}}/signup
```

This avoids repeating the same base URL in every request.

### C. Scripts

Postman supports JavaScript scripts that execute during the request lifecycle.

There are two important types:

**1. Pre-request Script**

Executes before sending the request.

Example:

```javascript
const timestamp = Date.now();

pm.variables.set("requestTime", timestamp);
```

**2. Post-response Script**

Executes after receiving the response.

Example:

```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});
```

Scripts can be used to prepare request data, extract response values, and validate API responses.

### D. Settings

Collection settings control collection-specific behavior and configuration.

For example, they include options related to request behavior, such as following HTTP redirects.

---

## 3. Collection Runner

**Collection Runner** is a Postman feature that executes multiple requests from a collection or folder in a specified sequence.

Instead of manually clicking Send for every request, we can execute them together.

### Example

Suppose we want to test the listing functionality in Wanderlust.

```text
1. Login
2. Create Listing
3. Get Listing
4. Update Listing
5. Delete Listing
```

Collection Runner can execute these requests in sequence.

### How Collection Runner works

1. Select a collection or folder.
2. Configure the run.
3. Select an environment if required.
4. Configure the number of iterations.
5. Start the run.
6. Inspect the execution results.

### Request Dependencies

Requests may depend on the output of previous requests.

For example:

- Creating a listing requires authentication.
- Fetching a listing requires its listing ID.
- Updating a listing requires its ID and appropriate authorization.
- Deleting a listing also requires its ID and appropriate authorization.

Postman scripts and variables can help transfer data between requests.

Example: Extracting the newly created listing ID.

```javascript
const response = pm.response.json();

pm.collectionVariables.set(
    "listingId",
    response.listing._id
);
```

The next request can use:

```text
{{baseUrl}}/listings/{{listingId}}
```

**Important:** Collection Runner executes requests in sequence, but authentication and request dependencies must be configured correctly.

### Iterations

An iteration represents one complete execution of the selected request sequence.

For example:

```text
Iterations = 3
```

The selected sequence executes three times.

Iterations are useful for testing APIs with different sets of input data.

Postman also supports CSV and JSON data files for data-driven testing.

---

## 4. Regression Testing

**Regression testing** means testing existing functionality again after making code changes to ensure that previously working features have not broken.

For example, after modifying the listing creation functionality, we can test:

- Login
- Listing creation
- Listing retrieval
- Listing updating
- Listing deletion

Collection Runner can help execute these API tests together.

---

## 5. Key Takeaways

| Feature | Purpose |
|---|---|
| Collection | Organizes related API requests. |
| Folder | Groups requests by functionality. |
| Collection Authorization | Reuses authorization configuration. |
| Collection Variables | Stores reusable values. |
| Pre-request Script | Executes before sending a request. |
| Post-response Script | Executes after receiving a response. |
| Collection Settings | Controls collection behavior. |
| Collection Runner | Executes multiple requests in sequence. |
| Iterations | Repeats the selected request sequence. |
| Regression Testing | Checks whether existing functionality still works after changes. |
