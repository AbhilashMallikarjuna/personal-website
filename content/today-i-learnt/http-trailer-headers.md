+++
title = 'HTTP Trailer Headers'
date = 2024-11-09T19:08:17+05:30
draft = false
hideSummary= true
+++

While returning a HTTP response, you cannot write headers once you complete writing the body.

So what if some headers needs to generated while message body is sent. Trailer Headers comes to our rescue here.

**Trailer Headers are HTTP response headers sent at the end of response message.** 
They work specifically with chunked transfer encoding i.e. where response sent in chunks.

You have to specify the header names which will be sent in the end with the header key `Trailer`

Example Usage of Trailer Header:
Computing a checksum to verify integrity
1. Initial Headers
```http
HTTP/1.1 200 OK
Transfer-Encoding: chunked
Trailer: Content-MD5
```

2. Response Body

```text
5\r\n 
Hello\r\n
5\r\n
World\r\n
0\r\n
Content-MD5: abcdef1234567890\r\n
\r\n
```


