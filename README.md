[![npm](https://img.shields.io/npm/v/nativescript-apiclient.svg)](https://www.npmjs.com/package/nativescript-apiclient)
[![npm](https://img.shields.io/npm/dt/nativescript-apiclient.svg?label=npm%20downloads)](https://www.npmjs.com/package/nativescript-apiclient)

# NativeScript API Client

A [NativeScript](https://nativescript.org/) module module for simply calling HTTP based APIs.

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
interface IUser {
    displayName: string;
    id?: number;
    name: string;
}

var client = ApiClient.newClient({
    baseUrl: "https://example.com/users",
    route: "{id}",  
});

client.clientError(function(result : ApiClient.IApiClientResult) {
                       // handle all responses with status code 400 to 499
                   })
      .serverError(function(result : ApiClient.IApiClientResult) {
                       // handle all responses with status code 500 to 599
                   })
      .success(function(result : ApiClient.IApiClientResult) {
                    // handle all responses with status codes less than 400
                    
                    var user = result.getJSON<IUser>();
               })
      .error(function(err : ApiClient.IApiClientError) {
                 // handle API client errors
             })
      .completed(function(ctx : ApiClient.IApiClientCompleteContext) {
                     // invoked after "result" and "error" actions
                 });

for (var userId = 1; userId <= 100; userId++) {
    // start a GET request
    client.get({
        routeParams: {
            id: userId
        }
    });
}
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
