[![npm](https://img.shields.io/npm/v/nativescript-apiclient.svg)](https://www.npmjs.com/package/nativescript-apiclient)
[![npm](https://img.shields.io/npm/dt/nativescript-apiclient.svg?label=npm%20downloads)](https://www.npmjs.com/package/nativescript-apiclient)

# NativeScript API Client

A [NativeScript](https://nativescript.org/) module module for simply calling HTTP based APIs.

[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=G88PA3Q7FFSGN)

## License

[MIT license](https://raw.githubusercontent.com/mkloubert/nativescript-apiclient/master/LICENSE)

## Platforms

* Android
* iOS

## Installation

Run

```bash
tns plugin add nativescript-apiclient
```

inside your app project to install the module.

## Demo

For quick start have a look at the [plugin/index.ts](https://github.com/mkloubert/nativescript-apiclient/blob/master/plugin/index.ts) or use the "IntelliSense" of your IDE to learn how it works.

Otherwise...

## Usage

### Import

```typescript
import ApiClient = require("nativescript-apiclient");
```

### Example

```typescript
import ApiClient = require("nativescript-apiclient");
import HTTP = require("http");

interface IUser {
    displayName: string;
    id?: number;
    name: string;
}

var client = ApiClient.newClient({
    baseUrl: "https://api.example.com/users",
    route: "{id}",  
});

client.beforeSend(function(opts: HTTP.HttpRequestOptions) {
                      // prepare the request here
                  })
      .clientError(function(result: ApiClient.IApiClientResult) {
                       // handle all responses with status code 400 to 499
                   })
      .serverError(function(result: ApiClient.IApiClientResult) {
                       // handle all responses with status code 500 to 599
                   })
      .success(function(result: ApiClient.IApiClientResult) {
                    // handle all responses with status codes less than 400
                    
                    var user = result.getJSON<IUser>();
               })
      .error(function(err: ApiClient.IApiClientError) {
                 // handle API client errors
             })
      .completed(function(ctx: ApiClient.IApiClientCompleteContext) {
                     // invoked after "result" and "error" actions
                 });

var credentials = new ApiClient.BasicAuth("Marcel", "p@ssword!");

for (var userId = 1; userId <= 100; userId++) {
    // start a GET request
    // 
    // [GET]  https://api.example.com/users/{id}?ver=1.6.6.6
    client.get({
        authorizer: credentials,
    
        // request headers
        headers: {
            'X-MyHeader-TM': 5979,
            'X-MyHeader-MK': 23979
        },
        
        // URL parameters
        params: {
            ver: 1.6.6.6
        },
    
        // route parameters
        routeParams: {
            id: userId  // {id}
        }
    });
}
```

## Routes

Routes are suffixes for a base URL.

You can define one or parameters inside that route, which are replaced when you start a request.

If you create a client like this

```typescript
var client = ApiClient.newClient({
    baseUrl: "https://api.example.com/users",
    route: "{id}/{resource}",  
});
```

and start a request like this

```typescript
client.get({
    routeParams: {
        id: 5979  // {id},
        resource: "profile"  // {resource}
    }
});
```

The client will call the URL

```
[GET]  https://api.example.com/users/5979/profile
```

## Authorization

You can submit an optional `IAuthorizer` object when you start a request:

```typescript
interface IAuthorizer {
    /**
     * Prepares a HTTP request for authorization.
     * 
     * @param {HTTP.HttpRequestOptions} reqOpts The request options.
     */
    prepare(reqOpts: HTTP.HttpRequestOptions);
}
```

The plugin provides the following implementations:

### AggregateAuthorizer

```typescript
var authorizer = new ApiClient.AggregateAuthorizer();
authorizer.addAuthorizers(new ApiClient.BasicAuth("Username", "Password"),
                          new ApiClient.BearerAuth("MySecretToken"));
```

### BasicAuth

```typescript
var authorizer = new ApiClient.BasicAuth("Username", "Password");
```

### BearerAuth

```typescript
var authorizer = new ApiClient.BearerAuth("MySecretToken");
```

## Requests

### GET

```typescript
// ?TM=5979&MK=23979
client.get({
    params: {
        TM: 5979,
        MK: 23979
    }
});
```

### POST

```typescript
client.post({
    content: {
        id: 5979,
        name: "Tanja"
    },
    
    type: ApiClient.HttpRequestType.JSON
});
```

### PUT

```typescript
client.put({
    content: '<user><id>23979</id><name>Marcel</name></user>',
    
    type: ApiClient.HttpRequestType.XML
});
```

### PATCH

```typescript
client.patch({
    content: '<user id="241279"><name>Julia</name></user>',
    
    type: ApiClient.HttpRequestType.XML
});
```

### DELETE

```typescript
client.post({
    content: {
        id: 221286
    },
    
    type: ApiClient.HttpRequestType.JSON
});
```

### Custom

```typescript
client.request("FOO", {
    content: {
        TM: 5979,
        MK: 23979
    },
    
    type: ApiClient.HttpRequestType.JSON
});
```

## Logging

If you want to log inside your result / error callbacks, you must define one or not logger actions in a client:

```typescript
var client = ApiClient.newClient({
    baseUrl: "https://example.com/users",
    route: "{id}",  
});

client.addLogger(function(msg : ApiClient.ILogMessage) {
    console.log("[" + ApiClient.LogSource[msg.source] + "]: " + msg.message);
});
```

Each action receives an object of the following type:

```typescript
interface ILogMessage {
    /**
     * Gets the category.
     */
    category: LogCategory;
    
    /**
     * Gets the message value.
     */
    message: any;
    
    /**
     * Gets the priority.
     */
    priority: LogPriority;
    
    /**
     * Gets the source.
     */
    source: LogSource;
    
    /**
     * Gets the tag.
     */
    tag: string;
    
    /**
     * Gets the timestamp.
     */
    time: Date;
}
```

Now you can starts logging in your callbacks:

```typescript
client.clientError(function(result : ApiClient.IApiClientResult) {
                       result.warn("Client error: " + result.code);
                   })
      .serverError(function(result : ApiClient.IApiClientResult) {
                       result.err("Server error: " + result.code);
                   })
      .success(function(result : ApiClient.IApiClientResult) {
                    result.info("Success: " + result.code);
               })
      .error(function(err : ApiClient.IApiClientError) {
                 result.crit("API CLIENT ERROR!: " + err.error);
             })
      .completed(function(ctx : ApiClient.IApiClientCompleteContext) {
                     result.dbg("Completed action invoked.");
                 });
```

The `IApiClientResult`, `IApiClientError` and `IApiClientCompleteContext` objects using the `ILogger` interface:

```typescript
export interface ILogger {
    /**
     * Logs an alert message.
     * 
     * @param any msg The message value.
     * @param {String} [tag] The optional tag value.
     * @param {LogPriority} [priority] The optional log priority.
     */
    alert(msg : any, tag?: string,
          priority?: LogPriority) : ILogger;
    
    /**
     * Logs a critical message.
     * 
     * @param any msg The message value.
     * @param {String} [tag] The optional tag value.
     * @param {LogPriority} [priority] The optional log priority.
     */
    crit(msg : any, tag?: string,
         priority?: LogPriority) : ILogger;
    
    /**
     * Logs a debug message.
     * 
     * @param any msg The message value.
     * @param {String} [tag] The optional tag value.
     * @param {LogPriority} [priority] The optional log priority.
     */
    dbg(msg : any, tag?: string,
        priority?: LogPriority) : ILogger;
    
    /**
     * Logs an emergency message.
     * 
     * @param any msg The message value.
     * @param {String} [tag] The optional tag value.
     * @param {LogPriority} [priority] The optional log priority.
     */
    emerg(msg : any, tag?: string,
          priority?: LogPriority) : ILogger;
    
    /**
     * Logs an error message.
     * 
     * @param any msg The message value.
     * @param {String} [tag] The optional tag value.
     * @param {LogPriority} [priority] The optional log priority.
     */
    err(msg : any, tag?: string,
        priority?: LogPriority) : ILogger;
    
    /**
     * Logs an info message.
     * 
     * @param any msg The message value.
     * @param {String} [tag] The optional tag value.
     * @param {LogPriority} [priority] The optional log priority.
     */
    info(msg : any, tag?: string,
         priority?: LogPriority) : ILogger;
    
    /**
     * Logs a message.
     * 
     * @param any msg The message value.
     * @param {String} [tag] The optional tag value.
     * @param {LogCategory} [category] The optional log category. Default: LogCategory.Debug
     * @param {LogPriority} [priority] The optional log priority.
     */
    log(msg : any, tag?: string,
        category?: LogCategory, priority?: LogPriority) : ILogger;
    
    /**
     * Logs a notice message.
     * 
     * @param any msg The message value.
     * @param {String} [tag] The optional tag value.
     * @param {LogPriority} [priority] The optional log priority.
     */
    note(msg : any, tag?: string,
         priority?: LogPriority) : ILogger;
     
    /**
     * Logs a trace message.
     * 
     * @param any msg The message value.
     * @param {String} [tag] The optional tag value.
     * @param {LogPriority} [priority] The optional log priority.
     */
    trace(msg : any, tag?: string,
          priority?: LogPriority) : ILogger;
        
    /**
     * Logs a warning message.
     * 
     * @param any msg The message value.
     * @param {String} [tag] The optional tag value.
     * @param {LogPriority} [priority] The optional log priority.
     */
    warn(msg : any, tag?: string,
         priority?: LogPriority) : ILogger;
}
```

## URL parameters

You can befine additional parameters for the URL.

If you create a client like this

```typescript
var client = ApiClient.newClient({
    baseUrl: "https://api.example.com/users"
});
```

and start a request like this

```typescript
client.get({
    params: {
        id: 23979,
        resource: "profile"
    }
});
```

The client will call the URL

```
[GET]  https://api.example.com/users?id=23979&resource=profile
```

## Responses

### Callbacks

#### Simple

```typescript
client.success(function(result : ApiClient.IApiClientResult) {
                    // handle any response
               });
```

The `result` object has the following structure:

```typescript
export interface IApiClientResult extends ILogger {
    /**
     * Gets the underlying API client.
     */
    client: IApiClient;

    /**
     * Gets the HTTP response code.
     */
    code: number;
    
    /**
     * Gets the raw content.
     */
    content: any;
    
    /**
     * Gets the underlying (execution) context.
     */
    context: ApiClientResultContext;
    
    /**
     * Gets the response headers.
     */
    headers: HTTP.Headers;
    
    /**
     * Returns the content as wrapped AJAX result object.
     * 
     * @return {IAjaxResult<TData>} The ajax result object.
     */
    getAjaxResult<TData>() : IAjaxResult<TData>;
    
    /**
     * Returns the content as file.
     * 
     * @param {String} [destFile] The custom path of the destination file.
     * 
     * @return {FileSystem.File} The file.
     */
    getFile(destFile?: string) : FileSystem.File;
    
    /**
     * Tries result the content as image source.
     */
    getImage(): Promise<Image.ImageSource>;
    
    /**
     * Returns the content as JSON object.
     */
    getJSON<T>() : T;
    
    /**
     * Returns the content as string.
     */
    getString() : string;
    
    /**
     * Gets the information about the request.
     */
    request: IHttpRequest;
    
    /**
     * Gets the raw response.
     */
    response: HTTP.HttpResponse;
}
```

### Errors

```typescript
client.error(function(err : ApiClient.IApiClientError) {
                 // handle an HTTP client error here
             });
```

The `err` object has the following structure:

```typescript
export interface IApiClientError extends ILogger {
    /**
     * Gets the underlying client.
     */
    client: IApiClient;
    
    /**
     * Gets the context.
     */
    context: ApiClientErrorContext;
    
    /**
     * Gets the error data.
     */
    error: any;
    
    /**
     * Gets or sets if error has been handled or not.
     */
    handled: boolean;
    
    /**
     * Gets the information about the request.
     */
    request: IHttpRequest;
}
```

#### Short hand callbacks

```typescript
client.clientError(function(result : ApiClient.IApiClientResult) {
                       // handle status codes between 400 and 499
                   });
                   
client.ok(function(result : ApiClient.IApiClientResult) {
                       // handle status codes with 200, 204 or 205
                   });
                   
client.serverError(function(result : ApiClient.IApiClientResult) {
                       // handle status codes between 500 and 599
                   });
```

The following methods are also supported:

| Name | Description |
| ---- | --------- |
| badGateway | Handles a request with status code `502`.  |
| badRequest | Handles a request with status code `400`. |
| forbidden | Handles a request with status code `403`. |
| gone | Handles a request with status code `410`. |
| internalServerError | Handles a request with status code `500`. |
| locked | Handles a request with status code `423`. |
| methodNotAllowed | Handles a request with status code `405`. |
| notFound | Handles a request with status code `404`. |
| notImplemented | Handles a request with status code `501`. |
| redirection | Handles a request with a status code between `300` and `399`. |
| serviceUnavailable | Handles a request with status code `503`. |
| succeededRequest | Handles a request with a status code between `200` and `299`. |
| tooManyRequests | Handles a request with status code `429`. |
| unauthorized | Handles a request with status code `401`. |
| unsupportedMediaType | Handles a request with status code `415`. |
| uriTooLong | Handles a request with status code `414`. |

##### Conditional callbacks

You can define callbacks for any kind of conditions.

A generic way to do this is to use the `if()` method:

```javascript
client.if(function(result : IApiClientResult) : boolean {
              // invoke if 'X-My-Custom-Header' is defined
              return undefined !== result.headers["X-My-Custom-Header"];
          },
          function(result : IApiClientResult) {
              // handle the response
          });
```

If no condition matches, the callback defined by `success()` method is used.

For specific status codes you can use the `ifStatus()` method:

```javascript
client.ifStatus(500,
                function(result : IApiClientResult) {
                    // handle the internal server error
                });
```
