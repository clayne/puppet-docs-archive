# Rules endpoint

Use the rules endpoint to translate a group’s rule condition into PuppetDB query syntax.

## POST /v1/rules/translate

Translate a group's rule condition into PuppetDB query syntax.

### Request format

The request's body should contain a rule condition as it would appear in the `rule` field of a group object. See the documentation on rule grammar for a complete description of how these conditions can be structured.

The endpoint supports an optional query parameter `format`, which defaults to `nodes`. If specified as `?format=inventory`, it allows you to get the classifier rules in a compatible [dot notation](https://puppet.com/docs/puppetdb/latest/api/query/v4/ast.html#dot-notation) format instead of the standard [PuppetDB AST](https://puppet.com/docs/puppetdb/latest/api/query/v4/ast.html).

### Response format

The response is a PuppetDB query string that can be used with PuppetDB nodes endpoint in order to see which nodes would satisfy the rule condition \(that is, which nodes would be classified into a group with that rule\).

### Error responses

Rules that use structured or trusted facts cannot be converted into PuppetDB queries, because PuppetDB does not yet support structured or trusted facts. If the rule cannot be translated into a PuppetDB query, the server will return a 422 Unprocessable Entity response containing the usual JSON error object. The error object will have a `kind` of "untranslatable-rule", a `msg` that describes why the rule cannot be translated, and contain the received rule in `details`.

If the request does not contain a valid rule, the server will return a 400 Bad Request response with the usual JSON error object. If the rule was not valid JSON, the error's `kind` will be "malformed-request", the `msg` will state that the request's body could not be parsed as JSON, and the `details` will contain the request's body as received by the server.

If the rule does not conform to the rule grammar, the `kind` key will be "schema-violation, and the `details` key will be an object with `submitted`, `schema`, and `error` keys which respectively describe the submitted object, the schema that object should conform to, and how the submitted object failed to conform to the schema.

**Related information**  


[GET /v1/groups](groups_endpoint.md#)

