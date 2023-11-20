# WireMock with JSON

## Basic Stubbing

- Reload changes to mappings:

```console
$ curl --verbose --request POST http://localhost:8080/__admin/mappings/reset
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> POST /__admin/mappings/reset HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Vary: Origin
< Transfer-Encoding: chunked
<
* Connection #0 to host localhost left intact
```

- Simple stubbing:
  - [mappings/hello-sun.json](mappings/hello-sun.json)
  - [mappings/hello-moon.json](mappings/hello-moon.json)

```console
curl --verbose http://localhost:8080/hello/moon
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /hello/moon HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: text/plain
< Matched-Stub-Id: 83ccb0cc-062d-49d3-b762-a53407f7124c
< Transfer-Encoding: chunked
<
* Connection #0 to host localhost left intact
Hello, moon!

$ curl --verbose http://localhost:8080/hello/sun
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /hello/sun HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: text/plain
< Matched-Stub-Id: 26242366-a15c-4552-9ff1-ce973f6118a7
< Transfer-Encoding: chunked
<
* Connection #0 to host localhost left intact
Hello, sun!
```

- JSON body response:
  - [mappings/hello-sun-json.json](mappings/hello-sun-json.json)

```console
$ curl --verbose http://localhost:8080/hello/sun/json
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /hello/sun/json HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: application/json
< Matched-Stub-Id: 16e3834f-7af0-4104-9677-581579563b54
< Transfer-Encoding: chunked
<
* Connection #0 to host localhost left intact
{"hello":"sun!"}
```

- Read body content from a file:
  - [mappings/hello-moon-file.json](mappings/hello-moon-file.json)
  - [__files/moon-file.json](__files/moon-file.json)

```console
$ curl --verbose http://localhost:8080/hello/moon/file
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /hello/moon/file HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: application/json
< Matched-Stub-Id: a3fd4b77-92c9-429d-85e1-d4349ddd11ad
< Transfer-Encoding: chunked
<
{
  "hello": "moon!"
}
* Connection #0 to host localhost left intact
```

- Default response for unmapped request:
  - [mappings/default-unmapped.json](mappings/default-unmapped.json)
  - `"priority": 10`: Default priority is `5`. Higher values have lower priority.

```console
$ curl --verbose http://localhost:8080/unmapped
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /unmapped HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< Content-Type: application/json
< Matched-Stub-Id: 314b220c-d77d-47ec-aa12-63941a1c4264
< Transfer-Encoding: chunked
<
* Connection #0 to host localhost left intact
{"status":"Error","message":"Endpoint not found"}

$ curl --verbose --request POST http://localhost:8080/hello/sun
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> POST /hello/sun HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< Content-Type: application/json
< Matched-Stub-Id: 314b220c-d77d-47ec-aa12-63941a1c4264
< Transfer-Encoding: chunked
<
* Connection #0 to host localhost left intact
{"status":"Error","message":"Endpoint not found"}
```

## References

- "Stubbing." _WireMock_, 17 Nov. 2023, [wiremock.org/docs/stubbing/](https://wiremock.org/docs/stubbing/). Accessed 20 Nov. 2023.
