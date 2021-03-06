# Response

When a GraphQL server receives a request, it must return a well-formed
response. The server's response describes the result of executing the requested
operation if successful, and describes any errors encountered during the
request.

A response may contain both a partial response as well as encountered errors in
the case that an error occurred on a field which was replaced with null.


## Serialization Format

GraphQL does not require a specific serialization format. However, clients
should use a serialization format that supports the major primitives in the
GraphQL response. In particular, the serialization format must support
representations of the following four primitives:

 * Map
 * List
 * String
 * Null

Serialization formats which can represent an ordered map should preserve the
order of requested fields as defined by {CollectFields()} in the Execution
section. Serialization formats which can only represent unordered maps should
retain this order grammatically (such as JSON).

Producing a response where fields are represented in the same order in which
they appear in the request improves human readability during debugging and
enables more efficient parsing of responses if the order of properties can
be anticipated.

A serialization format may support the following primitives, however, strings
may be used as a substitute for those primitives.

 * Boolean
 * Int
 * Float
 * Enum Value


### JSON Serialization

JSON is the preferred serialization format for GraphQL, though as noted above,
GraphQL does not require a specific serialization format. For consistency and
ease of notation, examples of the response are given in JSON throughout the
spec. In particular, in our JSON examples, we will represent primitives using
the following JSON concepts:

| GraphQL Value | JSON Value        |
| ------------- | ----------------- |
| Map           | Object            |
| List          | Array             |
| Null          | {null}            |
| String        | String            |
| Boolean       | {true} or {false} |
| Int           | Number            |
| Float         | Number            |
| Enum Value    | String            |

**Object Property Ordering**

While JSON Objects are specified as an
[unordered collection of key-value pairs](https://tools.ietf.org/html/rfc7159#section-4)
the pairs are represented in an ordered manner. In other words, while the JSON
strings `{ "name": "Mark", "age": 30 }` and `{ "age": 30, "name": "Mark" }`
encode the same value, they also have observably different property orderings.

Since the result of evaluating a selection set is ordered, the JSON object
serialized should preserve this order by writing the object properties in the
same order as those fields were requested as defined by query execution.

For example, if the query was `{ name, age }`, a GraphQL server responding in
JSON should respond with `{ "name": "Mark", "age": 30 }` and should not respond
with `{ "age": 30, "name": "Mark" }`.

NOTE: This does not violate the JSON spec, as clients may still interpret
objects in the response as unordered Maps and arrive at a valid value.


## Response Format

A response to a GraphQL operation must be a map.

If the operation included execution, the response map must contain a first entry
with key `data`. The value of this entry is described in the "Data" section. If
the operation failed before execution, due to a syntax error, missing
information, or validation error, this entry must not be present.

If the operation encountered any errors, the response map must contain a next
entry with key `errors`. The value of this entry is described in the "Errors"
section. If the operation completed without encountering any errors, this entry
must not be present.

The response map may also contain an entry with key `extensions`. This entry,
if set, must have a map as its value. This entry is reserved for implementors
to extend the protocol however they see fit, and hence there are no additional
restrictions on its contents.

To ensure future changes to the protocol do not break existing servers and
clients, the top level response map must not contain any entries other than the
three described above.


### Data

The `data` entry in the response will be the result of the execution of the
requested operation. If the operation was a query, this output will be an
object of the schema's query root type; if the operation was a mutation, this
output will be an object of the schema's mutation root type.

If an error was encountered before execution begins, the `data` entry should
not be present in the result.

If an error was encountered during the execution that prevented a valid
response, the `data` entry in the response should be `null`.


### Errors

The `errors` entry in the response is a non-empty list of errors, where each
error is a map.

If no errors were encountered during the requested operation, the `errors`
entry should not be present in the result.

Every error must contain an entry with the key `message` with a string
description of the error intended for the developer as a guide to understand
and correct the error.

If an error can be associated to a particular point in the requested GraphQL
document, it should contain an entry with the key `locations` with a list of
locations, where each location is a map with the keys `line` and `column`, both
positive numbers starting from `1` which describe the beginning of an
associated syntax element.

GraphQL servers may provide additional entries to error as they choose to
produce more helpful or machine-readable errors, however future versions of the
spec may describe additional entries to errors.

If the `data` entry in the response is `null` or not present, the `errors`
entry in the response must not be empty. It must contain at least one error.
The errors it contains should indicate why no data was able to be returned.

If the `data` entry in the response is not `null`, the `errors` entry in the
response may contain any errors that occurred during execution. If errors
occurred during execution, it should contain those errors.
