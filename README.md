# Simple Webserver

The webserver is a minimal implementation of a webserver (by the HTTP specification which can be found [here](https://datatracker.ietf.org/doc/html/rfc2616)) and uses only standard C libraries. The webserver has only minimal support for http features. The webservers main features is routes with individual responses and dynamically scaling client request handling(threads scale dynamically according to the amount of requests with a cleanup routine and are not buffered(compromise between performance and memory efficiency). The only supported request method is GET with no parsing of query parameters. It's a fun project of mine and although I tried to write a usable and safe application due to the complex nature of C I cannot guarantee for anything especially security.

The goal of the project is to write a minimal C webserver which maximizes on performance, security and memory safety.

Since part of the idea was to write something with a small footprint the whole project source code is contained in the `webserver.c` file.
The project is not meant to be used as a library but can be easily reused as one since the complete project is strictly statically written and everything is isolated from the execution context (all the functions are usable in a static library manner).

### Performance
The response and request buffer are both statically buffered. I thought about making the response buffer dynamically allocated in oder to be flexible with potential greater response payloads but decided that if this application would be used, this would happen in specific context in which the range of response sizes is not too great which would outweigh the performance decrease of a dynamically managed response.

Since I don't have any experience with writing software which has to handle huge bandwidths of requests and I still wanted to keep this project able to handle request spikes I the threads scale dynamically and are not limited by a statically sized buffer of max client threads.
All the memory allocated (by a client thread) is freed on socket close, also the actual payload is only referenced and only copied for the actual buffer send.

The parsing is implemented in a very basic manner, not leveraging any library functions. This makes maintenance more difficult and is generally challenging to read but (possibly)more performant and (possibly)more secure since it reduces operations on a few very simple procedures instead of implementing complex std lib functions.

### Security

As declared at the beginning of this projects readme it's not meant to be used in any kind of professional or production environment. I'm neither a professional nor do I have sufficient experience in order to claim this project to be secure.
That said I (as always) tried to considered all the good practices and possible attack vectors. Since this webserver is not complex and only support very few features the only superficial vector would be the http request string.

Apart from buffer overflows, 0 character escape the parsing is probably the most crucial and worrying part. In order to build something simple that does not open up too many eventualities I went with a character iterating loop which only checks for the 3 (SP,CR,LF) separating characters and copies/ parses the memory of the mem space in between. The only deciding information on which basis the parsing happens is the length (index difference)between the separation characters thus this is the only exploitable "interface" and is limited by the request buffer size. Every anomaly from the request protocol will result in an immediate abort. The introduction of malicious information in the parsed memory should be irrelevant since this memory is not interpreted in anyway afterwards(except for the versions strtok and character removal wichs common vulnerabilities were considered).

Feature support:
- dynamic route creation
- dynamic client request threads
- multi-client support
- buffer & file read response
- static header response
- GET method
