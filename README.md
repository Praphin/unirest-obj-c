# Unirest for Objective-C [![Build Status](https://api.travis-ci.org/Mashape/unirest-obj-c.png)](https://travis-ci.org/Mashape/unirest-obj-c)

Unirest is a set of lightweight HTTP libraries available in multiple languages.

## Installing
<a href="https://github.com/Mashape/unirest-obj-c/archive/master.zip">Download</a> the Objective-C Unirest Library from <a href="https://github.com/Mashape/unirest-obj-c">GitHub</a> (or clone the repo) and import the folder into your project.

The Unirest-Obj-C client library requires ARC (Automatic Reference Counting) to be enabled in your Xcode project. To enable ARC select your project or target and then go to Build Settings and under the section Apple LLVM compiler 3.0 - Language you will see the option Objective-C Automatic Reference Counting:

<img src="http://unirest.io/img/arc-enable.png" alt="Enable ARC in Xcode"/>

For existing projects, fortunately Xcode offers a tool to convert existing code to ARC, which is available at Edit -> Refactor  -> Convert to Objective-C ARC

## Creating Request
So you're probably wondering how using Unirest makes creating requests in Objective-C easier, let's look at a working example:

```objective-c
NSDictionary* headers = [NSDictionary dictionaryWithObjectsAndKeys:@"application/json", @"accept", nil];
NSDictionary* parameters = [NSDictionary dictionaryWithObjectsAndKeys:@"value", @"parameter", @"bar", @"foo", nil];

HttpJsonResponse* response = [[Unirest post:^(MultipartRequest* request) {
  [request setUrl:@"http://httpbin.org/post"];
  [request setHeaders:headers];
  [request setParameters:parameters];
}] asJson];
```
    
Just like in the Unirest Java library the Objective-C library supports multiple response types given as the last parameter. In the example above we use `asJson` to get a JSON response, likewise there are `asBinary` and `asString` for responses of other nature such as file data and hypermedia responses.

## Asynchronous Requests
For non-blocking requests you will want to make an asychronous request to keep your application going while data is fetched or updated in the background, doing so with unirest is extremely easy with barely any code change from the previous example:

```objective-c
NSDictionary* headers = [NSDictionary dictionaryWithObjectsAndKeys:@"application/json", @"accept", nil];
NSDictionary* parameters = [NSDictionary dictionaryWithObjectsAndKeys:@"value", @"parameter", @"bar", @"foo", nil];

[[Unirest post:^(MultipartRequest* request) {
  [request setUrl:@"http://httpbin.org/post"];
  [request setHeaders:headers];
  [request setParameters:parameters];
}] asJsonAsync:^(HttpJsonResponse* response) {
  // This is the asyncronous callback block
  int code = [response code];
  NSDictionary* responseHeaders = [response headers];
  JsonNode* body = [response body];
  NSData* rawBody = [response rawBody];
}];
```

## File Uploads
Transferring files through request with unirest in Objective-C can be done by creating a `NSURL` object and passing it along as a parameter value with a `MultipartRequest` like so:

```objective-c
NSDictionary* headers = [NSDictionary dictionaryWithObjectsAndKeys:@"application/json", @"accept", nil];
NSURL file = nil;
NSDictionary* parameters = [NSDictionary dictionaryWithObjectsAndKeys:@"value", @"parameter", file, @"file", nil];

HttpJsonResponse* response = [[Unirest post:^(MultipartRequest* request) {
  [request setUrl:@"http://httpbin.org/post"];
  [request setHeaders:headers];
  [request setParameters:parameters];
}] asJson];
```
 
## Custom Entity Body
To send a custom body such as JSON simply serialize your data utilizing the `NSJSONSerialization` with a `BodyRequest` and `[method]Entity` instead of just `[method]` block like so:

```objective-c
NSDictionary* headers = [NSDictionary dictionaryWithObjectsAndKeys:@"application/json", @"accept", nil];
NSDictionary* parameters = [NSDictionary dictionaryWithObjectsAndKeys:@"value", @"parameter", @"bar", @"foo", nil];

HttpJsonResponse* response = [[Unirest postEntity:^(BodyRequest* request) {
  [request setUrl:@"http://httpbin.org/post"];
  [request setHeaders:headers];
  // Converting NSDictionary to JSON:
  [request setBody:[NSJSONSerialization dataWithJSONObject:headers options:0 error:nil]];
}] asJson];
```

# Request
The Objective-C unirest library uses configuration blocks of type SimpleRequest, MultipartRequest and BodyRequest to configure the URL, Headers, and Parameters / Body of the request.

```objective-c
+(HttpRequest*) get:(void (^)(SimpleRequest*)) config;

+(HttpRequestWithBody*) post:(void (^)(MultipartRequest*)) config;
+(HttpRequestWithBody*) postEntity:(void (^)(BodyRequest*)) config;

+(HttpRequestWithBody*) put:(void (^)(MultipartRequest*)) config;
+(HttpRequestWithBody*) putEntity:(void (^)(BodyRequest*)) config;

+(HttpRequestWithBody*) patch:(void (^)(MultipartRequest*)) config;
+(HttpRequestWithBody*) patchEntity:(void (^)(BodyRequest*)) config;

+(HttpRequestWithBody*) delete:(void (^)(MultipartRequest*)) config;
+(HttpRequestWithBody*) deleteEntity:(void (^)(BodyRequest*)) config;
```

- `HttpRequest` `[Unirest get:` `(void (^)(SimpleRequest*))] config;`  
  
  Sends equivalent request with method type to given URL
- `HttpRequestWithBody` `[Unirest (post|postEntity):` `(void (^)(MultipartRequest|BodyRequest)(*))] config;`  
  
  Sends equivalent request with method type to given URL
- `HttpRequestWithBody` `[Unirest (put|putEntity):` `(void (^)(MultipartRequest|BodyRequest)(*))] config;`  
  
  Sends equivalent request with method type to given URL
- `HttpRequestWithBody` `[Unirest (patch|patchEntity):` `(void (^)(MultipartRequest|BodyRequest)(*))] config;`  
  
  Sends equivalent request with method type to given URL
- `HttpRequestWithBody` `[Unirest (delete|deleteEntity):` `(void (^)(MultipartRequest|BodyRequest)(*))] config;`
  
  Sends equivalent request with method type to given URL

# Response
The `HttpRequest` and `HttpRequestWithBody` can then be executed by calling one of:

```objective-c
-(HttpStringResponse*) asString;
-(void) asStringAsync:(void (^)(HttpStringResponse*)) response;

-(HttpBinaryResponse*) asBinary;
-(void) asBinaryAsync:(void (^)(HttpBinaryResponse*)) response;

-(HttpJsonResponse*) asJson;
-(void) asJsonAsync:(void (^)(HttpJsonResponse*)) response;
```

- `-(HttpStringResponse*)` `asString;`  
  
  Blocking request call with response returned as string for Hypermedia APIs or other.
- `-(void)` `asStringAsync:` `(void (^)(HttpBinaryResponse*)) response;`  
  
  Asynchronous request call with response returned as string for Hypermedia APIs or other.
- `-(HttpStringResponse*)` `asBinary;`  
  
  Blocking request call with response returned as binary output for files and other media.
- `-(void)` `asBinaryAsync:` `(void (^)(HttpBinaryResponse*)) response;`  
  
  Asynchronous request call with response returned as binary output for files and other media.
- `-(HttpStringResponse*)` `asJson;`  
  
  Blocking request call with response returned as JSON.
- `-(void)` `asJsonAsync:` `(void (^)(HttpBinaryResponse*)) response;`  
  
  Asynchronous request call with response returned as JSON.
