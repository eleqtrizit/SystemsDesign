# HTTP

## Common Http response codes  

200 OK The request has succeeded  
500 Internal Server Error (Server Error)  
403 Forbidden  
301 Permanent Redirect  
302 Temporary Redirect

Reference: [http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)

## What is a http cookie

Http cookie is a small piece of data that a server sends to a browser, which a browser usually stores in it’s cookie cache. Cookie can be used to maintain session information since HTTP is stateless, and also for user preferences at a given site. Cookies can also be used to store encrypted password. Browsers send cookies back to the server when they make a connection’  
Reference: [http://en.wikipedia.org/wiki/HTTP_cookie](http://en.wikipedia.org/wiki/HTTP_cookie)

## Http methods

Http methods are ways of communicating between server and client. 
GET, POST, PUT, PATCH, DELETE

HTTP is an application layer protocol relying on lower-level protocols such as **TCP** and **UDP**.


## Http Headers

Http header fields are common components of HTTP requests and responses. Headers are colon separated name-value pairs in clear text. Some common headers are: Cache-control which specifies where to cache or not the contents of a page, Accept, which can be text/plain, Content-length which specifies the size of the content, Host, which is the domain name of the server.  

## Remote procedure call (RPC)
![[Pasted image 20220417114522.png]]
In an RPC, a client causes a procedure to execute on a different address space, usually a remote server. The procedure is coded as if it were a local procedure call, abstracting away the details of how to communicate with the server from the client program. Remote calls are usually slower and less reliable than local calls so it is helpful to distinguish RPC calls from local calls. Popular RPC frameworks include [Protobuf](https://developers.google.com/protocol-buffers/), [Thrift](https://thrift.apache.org/), and [Avro](https://avro.apache.org/docs/current/).

#### Disadvantage(s): RPC
-   RPC clients become tightly coupled to the service implementation.
-   A new API must be defined for every new operation or use case.
-   It can be difficult to debug RPC.
-   You might not be able to leverage existing technologies out of the box. For example, it might require additional effort to ensure [RPC calls are properly cached](http://etherealbits.com/2012/12/debunking-the-myths-of-rpc-rest/) on caching servers such as [Squid](http://www.squid-cache.org/).

## Representational state transfer (REST)
REST is an architectural style enforcing a client/server model where the client acts on a set of resources managed by the server. The server provides a representation of resources and actions that can either manipulate or get a new representation of resources. All communication must be stateless and cacheable.

There are four qualities of a RESTful interface:
-   **Identify resources (URI in HTTP)** - use the same URI regardless of any operation.
-   **Change with representations (Verbs in HTTP)** - use verbs, headers, and body.
-   **Self-descriptive error message (status response in HTTP)** - Use status codes, don't reinvent the wheel.
-   **[HATEOAS](http://restcookbook.com/Basics/hateoas/) (HTML interface for HTTP)** - your web service should be fully accessible in a browser.

REST is focused on exposing data. It minimizes the coupling between client/server and is often used for public HTTP APIs. REST uses a more generic and uniform method of exposing resources through URIs, and actions through verbs such as GET, POST, PUT, DELETE, and PATCH. **Being stateless, REST is great for horizontal scaling and partitioning.**

#### Disadvantage(s): REST
-   With REST being focused on exposing data, it might not be a good fit if resources are not naturally organized or accessed in a simple hierarchy. For example, returning all updated records from the past hour matching a particular set of events is not easily expressed as a path. With REST, it is likely to be implemented with a combination of URI path, query parameters, and possibly the request body.
-   REST typically relies on a few verbs (GET, POST, PUT, DELETE, and PATCH) which sometimes doesn't fit your use case. For example, moving expired documents to the archive folder might not cleanly fit within these verbs.
-   Fetching complicated resources with nested hierarchies requires multiple round trips between the client and server to render single views, e.g. fetching content of a blog entry and the comments on that entry. For mobile applications operating in variable network conditions, these multiple roundtrips are highly undesirable.
-   Over time, more fields might be added to an API response and older clients will receive all new data fields, even those that they do not need, as a result, it bloats the payload size and leads to larger latencies.