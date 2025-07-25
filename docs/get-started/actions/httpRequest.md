---
title: httpRequest
layout: default
nav_order: 1
parent: Actions
grand_parent: Tests
description: Perform a generic HTTP request, for example to an API.
---

# httpRequest

The `httpRequest` action makes arbitrary HTTP calls, allowing you to interact with and validate APIs or other web services directly within your tests.

You can specify a simple GET request using a string shorthand or use an object for more complex requests and validation:

- **String Shorthand:** Provide the full URL directly as the value for the `httpRequest` key. This performs a simple GET request to that URL.
- **Object Format:** Use an object with the following properties:
  - `url`: (Required unless using `openApi`) The target URL for the request.
  - `method`: (Optional) The HTTP method (e.g., `GET`, `POST`, `PUT`, `DELETE`). Defaults to `GET`.
  - `timeout`: (Optional) Maximum duration in milliseconds to wait for the request to complete.
  - `request`: (Optional) An object defining the request details:
    - `headers`: (Optional) Key-value pairs for request headers.
    - `parameters`: (Optional) Key-value pairs for query string parameters.
    - `body`: (Optional) The request body. Can be a string or JSON object.
  - `response`: (Optional) An object defining expected response validation:
    - `headers`: (Optional) Key-value pairs for expected response headers. Values must be strings.
    - `body`: (Optional) Expected response body. Can be a string or JSON object.
  - `statusCodes`: (Optional) An array of acceptable HTTP status codes. If the response code is not in this list, the step fails (default: `[200]`).
  - `openApi`: (Optional) Define the request based on an OpenAPI definition. Can be a string (operation ID) or an object:
    - `name`: (Optional) Name of the registered OpenAPI definition (if multiple are loaded).
    - `descriptionPath`: (Optional) Path or URL to the OpenAPI description file.
    - `operationId`: (Required) The ID of the operation to use.
    - `useExample`: (Optional) Use example data from the OpenAPI spec (`request`, `response`, or `both`).
    - `exampleKey`: (Optional) Key of the specific example to use if multiple exist.
    *Note: Properties like `request.headers`, `request.parameters`, `request.body` can override values from the OpenAPI definition or example.*
  - *Output Saving:* You can also save the response body using `path`, `directory`, `maxVariation`, and `overwrite` properties. See the [`httpRequest`](/docs/references/schemas/httprequest) reference for details.

**Setting Variables:** To capture parts of the response for later steps, use the step-level `variables` object. You can assign values based on the response using expressions like `$$response.body`, `$$response.headers`, `$$response.status`, etc. You can use dot notation for nested JSON fields (e.g., `$$response.body.user.id`).

> For comprehensive options, see the [`httpRequest`](/docs/references/schemas/httprequest) reference.

## Examples

Here are a few ways you might use the `httpRequest` action:

### Simple GET request (string shorthand)

```json
{
  "tests": [
    {
      "steps": [
        {
          "description": "Perform a simple GET request.",
          "httpRequest": "https://reqres.in/api/users?page=2"
        }
      ]
    }
  ]
}
```

### Simple GET request (object format)

```json
{
  "tests": [
    {
      "steps": [
        {
          "description": "Perform a simple GET request using object format.",
          "httpRequest": {
            "url": "https://reqres.in/api/users?page=2"
          }
        }
      ]
    }
  ]
}
```

### POST request with JSON body

```json
{
  "tests": [
    {
      "steps": [
        {
          "description": "Create a user via POST request.",
          "httpRequest": {
            "url": "https://reqres.in/api/users",
            "method": "POST",
            "request": {
              "body": {
                "name": "morpheus",
                "job": "leader"
              }
            }
          }
        }
      ]
    }
  ]
}
```

### PUT request with headers and query parameters

```json
{
  "tests": [
    {
      "steps": [
        {
          "description": "Update a user via PUT request.",
          "httpRequest": {
            "url": "https://reqres.in/api/users/2",
            "method": "PUT",
            "request": {
              "headers": {
                "Content-Type": "application/json"
              },
              "parameters": {
                "source": "test"
              },
              "body": {
                "name": "morpheus",
                "job": "zion resident"
              }
            }
          }
        }
      ]
    }
  ]
}
```

### Validate response status code and body

```json
{
  "tests": [
    {
      "steps": [
        {
          "description": "Create user and validate response.",
          "httpRequest": {
            "url": "https://reqres.in/api/users",
            "method": "POST",
            "request": {
              "body": {
                "name": "morpheus",
                "job": "leader"
              }
            },
            "response": {
              "body": {
                "name": "morpheus",
                "job": "leader",
              }
            },
            "statusCodes": [201] // Expect HTTP 201 Created
          }
        }
      ]
    }
  ]
}
```

### Use OpenAPI definition (by operation ID)

Assumes an OpenAPI definition with operation ID `getUserById` is loaded.

```json
{
  "tests": [
    {
      "steps": [
        {
          "description": "Get user by ID using OpenAPI definition.",
          "httpRequest": {
            "openApi": "getUserById",
            "request": {
              "parameters": { // Provide required path parameter
                "id": 2
              }
            }
          }
        }
      ]
    }
  ]
}
```

### Use OpenAPI definition (detailed object)

```json
{
  "tests": [
    {
      "steps": [
        {
          "description": "Get user by ID using specific OpenAPI file.",
          "httpRequest": {
            "openApi": {
              "descriptionPath": "./reqres.openapi.json",
              "operationId": "getUserById"
            },
            "request": {
              "parameters": { 
                "id": 3
              }
            }
          }
        }
      ]
    }
  ]
}
```

### Set a variable from response body

```json
{
  "tests": [
    {
      "steps": [
        {
          "description": "Create user and capture the ID.",
          "httpRequest": {
            "url": "https://reqres.in/api/users",
            "method": "POST",
            "request": {
              "body": {"name": "neo", "job": "the one"}
            },
            "statusCodes": [201]
          },
          "variables": {
            "USER_ID": "$$response.body.id"
          }
        },
        {
          "description": "Use the captured ID in the next request.",
          "httpRequest": {
            "url": "https://reqres.in/api/users/$USER_ID", // Use variable in URL
            "method": "GET"
          }
        }
      ]
    }
  ]
}
```

### Save response body to a file

```json
{
  "tests": [
    {
      "steps": [
        {
          "description": "Get user data and save response to file.",
          "httpRequest": {
            "url": "https://reqres.in/api/users/2",
            "method": "GET",
            "path": "user_2_response.json", // File name
            "directory": "./output/api_responses" // Output directory
          }
        }
      ]
    }
  ]
}
