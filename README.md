# REST API STANDARDS

- [REST API STANDARDS](#rest-api-standards)
- [REST API Methods](#rest-api-methods)
  - [Examples](#examples)
  - [GET methods](#get-methods)
  - [POST methods](#post-methods)
  - [PUT methods](#put-methods)
  - [PATCH methods](#patch-methods)
  - [DELETE methods](#delete-methods)
- [Versioning](#versioning)
- [Resource Names](#resource-names)



# REST API Methods 

The HTTP protocol defines a number of methods that assign semantic meaning to a request. The common HTTP methods used by most RESTful web APIs are:

- GET retrieves a representation of the resource at the specified URI. The body of the response message contains the details of the requested resource.
- POST creates a new resource at the specified URI. The body of the request message provides the details of the new resource. Note that POST can also be used to trigger operations that don't actually create resources.
- PUT either creates or replaces the resource at the specified URI. The body of the request message specifies the resource to be created or updated.
- PATCH performs a partial update of a resource. The request body specifies the set of changes to apply to the resource.
- DELETE removes the resource at the specified URI.


## Examples


| Resource            | POST                              | GET                                 | PUT                                           | DELETE                           |
| ------------------- | --------------------------------- | ----------------------------------- | --------------------------------------------- | -------------------------------- |
| /customers          | Create a new customer             | Retrieve all customers              | Bulk update of customers                      | Remove all customers             |
| /customers/1        | Error                             | Retrieve the details for customer 1 | Update the details of customer 1 if it exists | Remove customer 1                |
| /customers/1/orders | Create a new order for customer 1 | Retrieve all orders for customer 1  | Bulk update of orders for customer 1          | Remove all orders for customer 1 |

## GET methods
A successful GET method typically returns HTTP status code 200 (OK). If the resource cannot be found, the method should return 404 (Not Found).

If the request was fulfilled but there is no response body included in the HTTP response, then it should return HTTP status code 204 (No Content); for example, a search operation yielding no matches might be implemented with this behavior.

Link returning one entry:
```
https://adventure-works.com/orders/1
```

Response:
```
{
    "orderId":1,
    "orderValue":99.90,
    "productId":1,
    "quantity":1
}
```

Link returning multiple entries:
```
https://adventure-works.com/orders
```

Response:
```
{
    "items": [
        {
            "orderId":1,
            "orderValue":99.90,
            "productId":1,
            "quantity":1
        },
        {
            "orderId":2,
            "orderValue":89.90,
            "productId":1,
            "quantity":1
        }
    ]
}
```

## POST methods
If a POST method creates a new resource, it returns HTTP status code 201 (Created). The URI of the new resource is included in the Location header of the response. The response body contains a representation of the resource.

If the method does some processing but does not create a new resource, the method can return HTTP status code 200 and include the result of the operation in the response body. Alternatively, if there is no result to return, the method can return HTTP status code 204 (No Content) with no response body.

If the client puts invalid data into the request, the server should return HTTP status code 400 (Bad Request). The response body can contain additional information about the error or a link to a URI that provides more details.

## PUT methods
If a PUT method creates a new resource, it returns HTTP status code 201 (Created), as with a POST method. If the method updates an existing resource, it returns either 200 (OK) or 204 (No Content). In some cases, it might not be possible to update an existing resource. In that case, consider returning HTTP status code 409 (Conflict).

Consider implementing bulk HTTP PUT operations that can batch updates to multiple resources in a collection. The PUT request should specify the URI of the collection, and the request body should specify the details of the resources to be modified. This approach can help to reduce chattiness and improve performance.

## PATCH methods
With a PATCH request, the client sends a set of updates to an existing resource, in the form of a patch document. The server processes the patch document to perform the update. The patch document doesn't describe the whole resource, only a set of changes to apply. The specification for the PATCH method (RFC 5789) doesn't define a particular format for patch documents. The format must be inferred from the media type in the request.

JSON is probably the most common data format for web APIs. There are two main JSON-based patch formats, called JSON patch and JSON merge patch.

JSON merge patch is somewhat simpler. The patch document has the same structure as the original JSON resource, but includes just the subset of fields that should be changed or added. In addition, a field can be deleted by specifying null for the field value in the patch document. (That means merge patch is not suitable if the original resource can have explicit null values.)

For example, suppose the original resource has the following JSON representation:

```

{
    "name":"gizmo",
    "category":"widgets",
    "color":"blue",
    "price":10
}
```
Here is a possible JSON merge patch for this resource:

```
{
    "price":12,
    "color":null,
    "size":"small"
}
```
This tells the server to update price, delete color, and add size, while name and category are not modified. For the exact details of JSON merge patch, see RFC 7396. The media type for JSON merge patch is application/merge-patch+json.

Merge patch is not suitable if the original resource can contain explicit null values, due to the special meaning of null in the patch document. Also, the patch document doesn't specify the order that the server should apply the updates. That may or may not matter, depending on the data and the domain. JSON patch, defined in RFC 6902, is more flexible. It specifies the changes as a sequence of operations to apply. Operations include add, remove, replace, copy, and test (to validate values). The media type for JSON patch is application/json-patch+json.

Here are some typical error conditions that might be encountered when processing a PATCH request, along with the appropriate HTTP status code.

Error condition	HTTP status code
The patch document format isn't supported.	415 (Unsupported Media Type)
Malformed patch document.	400 (Bad Request)
The patch document is valid, but the changes can't be applied to the resource in its current state.	409 (Conflict)

## DELETE methods

If the delete operation is successful, the web server should respond with HTTP status code 204 (No Content), indicating that the process has been successfully handled, but that the response body contains no further information. If the resource doesn't exist, the web server can return HTTP 404 (Not Found).

# Versioning

Each time you modify the web API or change the schema of resources, you add a version number to the URI for each resource. The previously existing URIs should continue to operate as before, returning resources that conform to their original schema.

Extending the previous example, if the address field is restructured into subfields containing each constituent part of the address (such as streetAddress, city, state, and zipCode), this version of the resource could be exposed through a URI containing a version number, such as https://adventure-works.com/v2/customers/3

# Resource Names

Resource names must be plural nouns when referring to a resource collection e.g. ‘/users’. A singleton, such as ‘/users/1234/cart’ must be singular.