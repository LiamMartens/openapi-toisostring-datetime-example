# Shows an issue with DateTime type for OpenAPI generator
## Scenario
We use the OpenAPI generator tooling to generate a Typescript Axios client for a swagger definition which has a `date-time` parameter. However for our purpose we type map the `date` object to `string` using `--type-mappings Date=string` (this subsequently also maps `date-time` to `string`).
```
"parameters": [
    {
        "name": "param",
        "in": "query",
        "required": false,
        "type": "string",
        "format": "date-time"
    }
],
```

## Issue description
The generated code arguments appear correctly as `string`, but a `toISOString()` call is still executed.
```
example(param?: string, options: any = {}): RequestArgs {
    ....
    if (param !== undefined) {
        localVarQueryParameter['param'] = (param as any).toISOString(); <-- toISOString() on string type
    }
    ...
},
```

## Solution
A fix was applied to date template where a `instanceof Date` check was added to prevent this. The same fix should be applied to the date-time type.
[https://github.com/OpenAPITools/openapi-generator/pull/3423](https://github.com/OpenAPITools/openapi-generator/pull/3423)
```
// this fix was applied to date type
localVarQueryParameter['{{baseName}}'] = ({{paramName}} as any instanceof Date) ?
    ({{paramName}} as any).toISOString().substr(0,10) :
    {{paramName}};

// this fix can be applied to date-time
localVarQueryParameter['{{baseName}}'] = ({{paramName}} as any instanceof Date) ?
    ({{paramName}} as any).toISOString() :
    {{paramName}};
```
I have opened a PR here for it [here](https://github.com/OpenAPITools/openapi-generator/pull/5057)