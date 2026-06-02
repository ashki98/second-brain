# Status Code

Here is the **full text** of the "HTTP Status Codes" page from restfulapi.net, ready for you to copy in one place:[restfulapi](https://restfulapi.net/http-status-codes/)

---

HTTP specification defines these standard status codes divided into five categories that can be used to convey the results of a client’s request.

Written by: Lokesh Gupta

Last Updated: August 9, 2024

REST APIs use the **Status-Line** part of an HTTP response message to inform clients of their request’s overarching result. RFC 2616 defines the Status-Line syntax as shown below:

> Status-Line = HTTP-Version SP Status-Code SP Reason-Phrase CRLF
> 

HTTP defines these standard status codes that can be used to convey the results of a client’s request. The status codes are divided into five categories.

- **1xx: Informational** – Communicates transfer protocol-level information.
- **2xx: Success** – Indicates that the client’s request was accepted successfully.
- **3xx: Redirection** – Indicates that the client must take some additional action in order to complete their request.
- **4xx: Client Error** – This category of error status codes points the finger at clients.
- **5xx: Server Error** – The server takes responsibility for these error status codes.

---

## 1xx Status Codes [Informational]

| Status Code | Description |
| --- | --- |
| 100 Continue | An interim response. Initial part of the request has been received and has not yet been rejected by the server. |
| 101 Switching Protocol | Sent in response to an Upgrade request header, indicates protocol switching. |
| 102 Processing (WebDAV) | Server has received and is processing the request, but no response yet. |
| 103 Early Hints | Used with the `Link` header, suggests preloading resources while server prepares final response. |

---

## 2xx Status Codes [Success]

| Status Code | Description |
| --- | --- |
| 200 OK | Request succeeded. |
| 201 Created | Request succeeded and a new resource was created. |
| 202 Accepted | Request received but not yet acted upon. |
| 203 Non-Authoritative Information | Metadata from a local or third-party copy returned, maybe subset/superset of original. |
| 204 No Content | Server fulfilled request, no response body. |
| 205 Reset Content | Client should reset the document that sent this request. |
| 206 Partial Content | Used when `Range` header requests part of a resource. |
| 207 Multi-Status (WebDAV) | Multiple operations happened; status for each in response body. |
| 208 Already Reported (WebDAV) | Client indicated same resource earlier. Only appears in bodies. |
| 226 IM Used | GET request fulfilled with instance-manipulations applied. |

---

## 3xx Status Codes [Redirection]

| Status Code | Description |
| --- | --- |
| 300 Multiple Choices | More than one possible response for the request. |
| 301 Moved Permanently | Resource URL changed permanently; Location header provides new URL. |
| 302 Found | Resource URL changed temporarily; Location header provides new URL. |
| 303 See Other | Response can be found under a different URI via GET request. |
| 304 Not Modified | Resource unchanged since last request; client can use cached copy. |
| 305 Use Proxy (Deprecated) | Response must be accessed by a proxy. |
| 306 (Unused) | Reserved, not used. |
| 307 Temporary Redirect | Requested resource at another URI; same HTTP method as previous request. |
| 308 Permanent Redirect (experimental) | Resource permanently at another URI; same HTTP method as previous request. |

---

## 4xx Status Codes (Client Error)

| Status Code | Description |
| --- | --- |
| 400 Bad Request | Malformed syntax; client should not repeat request without modifications. |
| 401 Unauthorized | Request requires authentication; repeat request with proper Authorization header. |
| 402 Payment Required (Experimental) | Reserved for future use; digital payment systems. |
| 403 Forbidden | Unauthorized request; client’s identity known to server. |
| 404 Not Found | Requested resource not found. |
| 405 Method Not Allowed | Request method disabled for that resource. |
| 406 Not Acceptable | Server can't provide content in requested format. |
| 407 Proxy Authentication Required | Client must authenticate with proxy first. |
| 408 Request Timeout | Server didn’t receive complete request in allotted time. |
| 409 Conflict | Conflict with current state of resource. |
| 410 Gone | Resource no longer available. |
| 411 Length Required | Must provide Content-Length header field. |
| 412 Precondition Failed | Header preconditions not met. |
| 413 Request Entity Too Large | Request entity exceeds server limits. |
| 414 Request-URI Too Long | URI longer than server can interpret. |
| 415 Unsupported Media Type | Content-Type not supported. |
| 416 Requested Range Not Satisfiable | Range in `Range` header can't be fulfilled. |
| 417 Expectation Failed | Server unable to meet the expectations in request. |
| 418 I’m a teapot (RFC 2324) | April Fool joke, not expected to be implemented. Used by some providers. |
| 420 Enhance Your Calm (Twitter) | Client is rate-limited by Twitter API. |
| 422 Unprocessable Entity (WebDAV) | Content syntax understood, still unable to process. |
| 423 Locked (WebDAV) | Resource being accessed is locked. |
| 424 Failed Dependency (WebDAV) | Request failed due to previous request failure. |
| 425 Too Early (WebDAV) | Server unwilling to risk replayed request processing. |
| 426 Upgrade Required | Request refused; process after protocol upgrade. |
| 428 Precondition Required | Requires conditional request. |
| 429 Too Many Requests | User sent too many requests (rate limiting). |
| 431 Request Header Fields Too Large | Header fields too large for server to process. |
| 444 No Response (Nginx) | Returns no info, closes the connection. |
| 449 Retry With (Microsoft) | Retry after performing appropriate action. |
| 450 Blocked by Windows Parental Controls | Parental Controls blocking webpage. |
| 451 Unavailable For Legal Reasons | Resource cannot legally be provided. |
| 499 Client Closed Request (Nginx) | Client closed connection before server responded. |

---

## 5xx Status Codes (Server Error)

| Status Code | Description |
| --- | --- |
| 500 Internal Server Error | Unexpected condition, request not fulfilled. |
| 501 Not Implemented | HTTP method not supported. |
| 502 Bad Gateway | Invalid response as gateway. |
| 503 Service Unavailable | Server not ready to handle request. |
| 504 Gateway Timeout | Gateway cannot get response in time. |
| 505 HTTP Version Not Supported | HTTP version not supported (experimental). |
| 506 Variant Also Negotiates | Server internal configuration error. |
| 507 Insufficient Storage (WebDAV) | Server cannot store resource representation. |
| 508 Loop Detected (WebDAV) | Infinite loop during request processing. |
| 510 Not Extended | Requires request extensions. |
| 511 Network Authentication Required | Client needs network authentication. |

---

## REST Specific HTTP Status Codes

**200 (OK)**: REST API carried out requested action successfully, includes response body, dependent on method.

**201 (Created)**: Resource created inside a collection.

**202 (Accepted)**: Request accepted for processing, not completed yet (often used for long-running or batch processes).

**204 (No Content)**: Usually for PUT/POST/DELETE requests, API declines to send back any status message or representation.

**301 (Moved Permanently)**: Resource redesignated, new permanent URI assigned, specified in Location header.

**302 (Found)**: Common URL redirect, location provided in header.

**303 (See Other)**: Controller finished work, provides URI of resource (temporarily or permanently), not cached.

**304 (Not Modified)**: Resource unchanged since previous request, no retransmission of resource needed.

**307 (Temporary Redirect)**: Client should resubmit to URI given in Location header; future requests still use original URI.

**400 (Bad Request)**: Generic client-side error, malformed request syntax, invalid params, etc.

**401 (Unauthorized)**: Attempted operation on protected resource, wrong or no credentials, must include WWW-Authenticate header.

**403 (Forbidden)**: Well-formed request, API refuses to honor, user lacks permissions.

**404 (Not Found)**: Resource can’t be mapped but may be available in future.

**405 (Method Not Allowed)**: Tried using HTTP method not allowed for resource; Allow header lists allowed methods.

**406 (Not Acceptable)**: API can’t generate preferred media type per Accept header.

**412 (Precondition Failed)**: Precondition specified in headers not met, request not carried out.

**415 (Unsupported Media Type)**: Content-Type not supported by API.

**500 (Internal Server Error)**: Generic error message for unexpected condition, reasonable to retry.

**501 (Not Implemented)**: Server does not recognize method or can’t fulfill request.

References: [www.iana.org/assignments/http-status-codes/http-status-codes.xhtml](https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml)

---

For even more, the page includes discussions, comments, practical pointers, and extended references. Let me know if you want any **specific section or code explained, summarized, or formatted!**

1. [https://restfulapi.net/http-status-codes/](https://restfulapi.net/http-status-codes/)
