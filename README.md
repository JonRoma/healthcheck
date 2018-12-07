Docker Healthcheck for HTTP and AJP endpoints
----------------------------------------------

The `healthcheck` command line utility is designed to be used as a
Docker health check. It can be used to check the status code returned
by a GET or HEAD method to an HTTP or [AJP](https://tomcat.apache.org/connectors-doc/ajp/ajpv13a.html) endpoint.

The `healthcheck` program can be installed using the following command:
```
    $ go get github.com/techservicesillinois/healthcheck
```
The utility is also available as a Docker image on [Docker Hub](https://hub.docker.com/r/techservicesillinois/healthcheck/).

Usage
-----

`healthcheck` supports the following options:

```
Usage: healthcheck URL1 URL2 ... URLN
  -b, --body                               Print full body of response. By default only headers are shown
  -c, --expected-status-code stringArray   Check expected status code is returned. (default [200])
  -g, --get                                Use GET instead of HEAD
  -L, --location                           Follow redirects in Location header field.
  -q, --quiet                              Quiet mode (don't output anything)
```

Examples
--------

1) The following is a simple example to ensure that an HTTP endpoint
returns a 200. We see that it does and that the `healthcheck` utility
returns a status code of 0.

```
$ healthcheck http://www.google.com
HTTP/1.1 200 OK
Transfer-Encoding: chunked
Accept-Ranges: none
Cache-Control: private, max-age=0
Content-Type: text/html; charset=ISO-8859-1
Date: Wed, 05 Dec 2018 23:02:40 GMT
Expires: -1
P3p: CP="This is not a P3P policy! See g.co/p3phelp for more info."
Server: gws
Set-Cookie: 1P_JAR=2018-12-05-23; expires=Fri, 04-Jan-2019 23:02:40 GMT; path=/; domain=.google.com
Set-Cookie: NID=150=NtnnCkxXlvY1Psv848mYJnu66NiJ9P2-ACezvKtQ218UqT9-A_DAsjiqF8lIT_5QUjlxHoVM52jbrtstSVTn324rJM0vQtFUUDGDTjHqE-04dqqYLXgwn08M_rIIyJGUQ6euc3WYRSuF2L1QV3GwJqH2RIIxph6qB0qUipj4vXQ; expires=Thu, 06-Jun-2019 23:02:40 GMT; path=/; domain=.google.com; HttpOnly
Vary: Accept-Encoding
X-Frame-Options: SAMEORIGIN
X-Xss-Protection: 1; mode=block

$ echo $?
0
```

2) The next example is similar but is a failing test that checks
for a 303. We can see that the status code returned is 1:
```
$ healthcheck -c 303 http://www.google.com
HTTP/1.1 200 OK
Transfer-Encoding: chunked
Accept-Ranges: none
Cache-Control: private, max-age=0
Content-Type: text/html; charset=ISO-8859-1
Date: Wed, 05 Dec 2018 23:04:43 GMT
Expires: -1
P3p: CP="This is not a P3P policy! See g.co/p3phelp for more info."
Server: gws
Set-Cookie: 1P_JAR=2018-12-05-23; expires=Fri, 04-Jan-2019 23:04:43 GMT; path=/; domain=.google.com
Set-Cookie: NID=150=uiAhei5hEmh2unTDSwvau0JCqAaQOxj7pXTP8pMfb-xuEnGhN7zR6PQ6KN1fEzoaHNYx_AoZ0aWKz_AW6wL1GE6Yu0lllD8FeJ3qr_WQB4kyfz_jjUg87a9butktDK5NH3iTfxo7fTWB5LvmOi4L-wk5kXJtm-6wPxg2ZSGIPMI; expires=Thu, 06-Jun-2019 23:04:43 GMT; path=/; domain=.google.com; HttpOnly
Vary: Accept-Encoding
X-Frame-Options: SAMEORIGIN
X-Xss-Protection: 1; mode=block

Bad Response code: 200
Expected Response code: 303

$ echo $?
1
```

3) Here we see how `healthcheck` can be used in a Dockerfile as a
Docker health check for multiple endpoints:
```
HEALTHCHECK CMD /healthcheck \
    -c 200 http://localhost:8080/auth/Shibboleth.sso/Metadata \
    -c 200 http://localhost:8080/auth/Shibboleth.sso/Status \
    -c 204 http://localhost:8080/auth/elmr/status
```

4) The `healthcheck` utility also supports the AJP protocol. Here
is an example from a Dockerfile:
```
HEALTHCHECK CMD /bin/healthcheck -c 200 http://127.0.0.1:8009/auth/elmr/config -c 302 http://127.0.0.1:8009/auth/elmr/attributes
```
