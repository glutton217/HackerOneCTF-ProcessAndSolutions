## Http basics
### Requests
Basic format:
	```
	VERB/resource/locator HTTP/1.1
	Header1: Value1
	Header2: Value2
	...
	<body of request>
	```
### Request Headers
- Host: indicates desired host handling request
- Accept: Indicates what MIME type(s) are accepted by the client; often used to specify JSON or XML output for web-services
- Cookie: Passes cookie data to the server
- Referer: Page leading to this request (not passed to other servers when using HTTPS on the origin)
- Authorization: used for 'basic auth' pages. takes the form "Basic <base64'd username:password>"
### Cookies
- key-value fairs of data that are sent from server and reside on client for fixed period of time
- each cookie has a domain pattern it applies to and they're passed with each request the client makes to matching hosts
- cookies going to a place they shouldn't be can be dangerous. You also shouldn't be able to set cookies on domains you shouldn't be able to
## Cookie security
- cookies added for .example.com can be read by any subdomain of example.com
- cookies added for a subdomain can only read in that subdomain and its subdomains
- a subdomain can set cookies for its own subdomains and parent, but it can't set cookies for sibling domains
	- test.example.com can't set cookies on test2.example.com, but can set them on example.com and foo.test.example.com
- There are two important flags to know:
	- Secure: the cookie only accessible to HTTPS pages
	- HTTPOnly: The cookie cannot be read by Javascript
		- document.cookie javascript will not work here, only accessible through the http request itself
- The server indicates these flags in the Set-Cookie header that passes them in the first place
## HTML Parsing
HTML should be parsed according to relevant spec (HTML5), however when thinking about security HTML isn't just parsed by the browser but also by web-application firewalls and other filters. Where there's a discrepancy in how different technology parses things there is most likely a vulnerability.
## Mime sniffing

## Encoding sniffing

## Same-Origin policy

## CSRF (Cross-Site Request Forgery)