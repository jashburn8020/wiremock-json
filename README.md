# WireMock with JSON

- [WireMock with JSON](#wiremock-with-json)
  - [Basic Stubbing](#basic-stubbing)
  - [Response Templating](#response-templating)
    - [Enabling/disabling response templating](#enablingdisabling-response-templating)
    - [Templated body file](#templated-body-file)
    - [The request model](#the-request-model)
    - [Transformer parameters](#transformer-parameters)
    - [Handlebar helpers](#handlebar-helpers)
      - [String helpers](#string-helpers)
      - [Conditional helpers](#conditional-helpers)
    - [Date and time helpers](#date-and-time-helpers)
    - [Random helpers](#random-helpers)
    - [Base64 helper](#base64-helper)
    - [Contains helper](#contains-helper)
    - [Matches helper](#matches-helper)
    - [Regular expression extract helper](#regular-expression-extract-helper)
  - [References](#references)

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

- Multi-stub JSON files:
  - [mappings/multi-stub.json](mappings/multi-stub.json)

```console
$ curl --verbose http://localhost:8080/multistub/suns
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /multistub/suns HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: application/json
< Matched-Stub-Id: 1614d01b-97af-4f03-a861-5c0ffb060c8a
< Transfer-Encoding: chunked
<
* Connection #0 to host localhost left intact
[{"id":1},{"id":2}]

$ curl --verbose --request POST http://localhost:8080/multistub/suns
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> POST /multistub/suns HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 201 Created
< Location: /multistub/suns/3
< Matched-Stub-Id: c238f161-a021-4545-8a66-3342aae1bde9
< Transfer-Encoding: chunked
<
* Connection #0 to host localhost left intact
```

## Response Templating

- Response headers and bodies, as well as proxy URLs, can optionally be rendered using [Handlebars templates](https://handlebarsjs.com/)
  - enables attributes of the request to be used in generating the response

### Enabling/disabling response templating

- Response templating is enabled by default in local mode.
- Command line options when running WireMock server as a standalone process:
  - `--global-response-templating`: Render all response definitions using Handlebars templates.
  - `--local-response-templating`: Enable rendering of response definitions using Handlebars templates for specific stub mappings.
- [mappings/templating.json](mappings/templating.json)
  - Note the use of `"transformers": ["response-template"]` in the 2nd mapping.
- Without using any of the command line options or with the `--local-response-templating` option:

```console
$ curl --verbose http://localhost:8080/templating/notransformer
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /templating/notransformer HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: text/plain
< Matched-Stub-Id: ba139dcf-72ef-487a-897c-59b0500d2ef2
< Transfer-Encoding: chunked
<
* Connection #0 to host localhost left intact
{{request.path.[1]}}

$ curl --verbose http://localhost:8080/templating/withtransformer
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /templating/withtransformer HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: text/plain
< Matched-Stub-Id: 48cf8518-9eae-4472-bdf0-5ad63d72d6c4
< Transfer-Encoding: chunked
<
* Connection #0 to host localhost left intact
withtransformer
```

- Using the `--global-response-templating` option:

```console
$ java -jar wiremock-standalone-3.3.1.jar --global-response-templating

$ curl --verbose http://localhost:8080/templating/notransformer
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /templating/notransformer HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: text/plain
< Matched-Stub-Id: 538e3fbb-5efd-4df7-a5d8-678c533ff434
< Transfer-Encoding: chunked
<
* Connection #0 to host localhost left intact
notransformer

$ curl --verbose http://localhost:8080/templating/withtransformer
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /templating/withtransformer HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: text/plain
< Matched-Stub-Id: bdc477c2-e7d4-49e5-a9a1-834fc5e6e933
< Transfer-Encoding: chunked
<
* Connection #0 to host localhost left intact
withtransformer
```

### Templated body file

- `"bodyFileName": "..."`
- The body file for a response can be selected dynamically by templating the file path:
  - [mappings/templated-body-file.json](mappings/templated-body-file.json)
  - [__files/moon-file.json](__files/moon-file.json)

```console
$ curl --verbose http://localhost:8080/templated-body-file/moon-file.json
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /templated-body-file/moon-file.json HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Matched-Stub-Id: a2ae7eeb-1b48-42ea-82c7-5b99059127fa
< Transfer-Encoding: chunked
<
{
  "hello": "moon!"
}
* Connection #0 to host localhost left intact
```

### The request model

- `{{request...}}`
- The model of the request is supplied to the header and body templates:
  - [mappings/request-model.json](mappings/request-model.json)
- See <https://wiremock.org/docs/response-templating/#the-request-model>

```console
$ curl http://localhost:8080/request-model/one/two?three=3 | python3 -m json.tool
{
    "request.url": "/request-model/one/two?three=3",
    "request.path": "/request-model/one/two",
    "request.path.0": "request-model",
    "request.pathSegments.1": "one",
    "request.pathSegments.2": "two",
    "request.query.three": "3",
    "request.method": "GET",
    "request.headers.User-Agent": "curl/7.81.0"
}
```

```console
$ curl --verbose --header "Content-Type: application/json" http://localhost:8080/request-model --data-binary @- <<EOF
{
  "request": "body"
}
EOF
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> POST /request-model HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
> Content-Type: application/json
> Content-Length: 24
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 201 Created
< Location: /request-model/0
< Content-Type: application/json
< Matched-Stub-Id: f55f78be-b1ee-4d58-ba99-d08f7790d512
< Transfer-Encoding: chunked
<
{
  "request": "body"
}
* Connection #0 to host localhost left intact
```

- `"urlPathTemplate": "/request-model-path-template/{contactId}/addresses/{addressId}"`
- `{{request.path.contactId}}`
- With URL path template ([RFC 6570](https://www.rfc-editor.org/rfc/rfc6570)), you can additionally reference path variables by name:
  - [mappings/request-model-path-template.json](mappings/request-model-path-template.json)

```console
$ curl --verbose  http://localhost:8080/request-model-path-template/001/addresses/addr_100
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /request-model-path-template/001/addresses/addr_100 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Matched-Stub-Id: c5c9a11f-9e6f-42f9-8931-c321f04188f6
< Transfer-Encoding: chunked
<
* Connection #0 to host localhost left intact
{"request.path.contactId":"001","request.path.addressId":"addr_100"}
```

### Transformer parameters

- Parameter values can be passed to the transformer, referenced in template body content using the `parameters` prefix.

```json
    "transformerParameters": {
      "idToName": {
        "1": "lorem",
        "2": "ipsum",
        "3": "dolor"
      }
    }
```

- `{{lookup parameters.idToName id}}`
  - `id` is a variable whose value is set using the `assign` helper.
- [mappings/transformer-parameter.json](mappings/transformer-parameter.json)
  - (inspired by [https://github.com/francocm/wiremock-template-value-mappings/blob/main/stubs/mappings/example-get-user.json](https://github.com/francocm/wiremock-template-value-mappings/blob/main/stubs/mappings/example-get-user.json))
- Helpers:
  - [`lookup`](https://github.com/jknack/handlebars.java/blob/master/handlebars/src/main/java/com/github/jknack/handlebars/helper/LookupHelper.java): Dynamic parameter resolution using Handlebars variables.

```console
$ curl --verbose  http://localhost:8080/transformer-parameter/?id=3
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /transformer-parameter/?id=3 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: application/json
< Matched-Stub-Id: 4ccea15f-89b1-4bd0-b964-23ff7daa03f4
< Transfer-Encoding: chunked
<
* Connection #0 to host localhost left intact
{"username":"dolor"}
```

```json
    "transformerParameters": {
      "idToName": {
        "1": {
          "given": "given_1",
          "family": "family_1"
        },
        "2": {
          "given": "given_2",
          "family": "family_2"
        },
        "3": {
          "given": "given_3",
          "family": "family_3"
        }
      },
      "idToPostcode": {
        "1": "AA1 1AA",
        "2": "BB2 2BB",
        "3": "CC3 3CC"
      }
    }
```

- `{{#assign 'id'}}{{request.query.id}}{{/assign}}{{id}}`
- `{{#with (lookup parameters.idToName id)}}{{given}}{{/with}}`
- `{{#with (lookup parameters.idToName id)}}{{family}}{{/with}}`
- `{{lookup parameters.idToPostcode id}}`
- Multiple parameters with a templated body file.
  - [mappings/transformer-parameter-multiple.json](mappings/transformer-parameter-multiple.json)
  - [__files/transformer-parameter-file.json](__files/transformer-parameter-file.json)
- Helpers:
  - [`assign`](https://github.com/jknack/handlebars.java/blob/master/handlebars-helpers/src/main/java/com/github/jknack/handlebars/helper/AssignHelper.java): Create auxiliary/temporary variables.
  - [`with`](https://github.com/jknack/handlebars.java/blob/master/handlebars/src/main/java/com/github/jknack/handlebars/helper/WithHelper.java): Change evaluation context of the template part.
- [Subexpressions](https://handlebarsjs.com/guide/expressions.html#subexpressions): Delimited by parentheses, invoke multiple helpers within a single mustache, and pass in the results of inner helper invocations as arguments to outer helpers.

```console
$ curl --verbose  http://localhost:8080/transformer-parameter-multiple/?id=2
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /transformer-parameter-multiple/?id=2 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: application/json
< Matched-Stub-Id: 407d8ac8-99c3-48be-9bc8-7989527977f5
< Transfer-Encoding: chunked
<
{
  "id": "2",
  "given": "given_2",
  "family": "family_2",
  "postcode": "BB2 2BB"
}
* Connection #0 to host localhost left intact
```

### Handlebar helpers

- All of the [standard helpers](https://github.com/jknack/handlebars.java/tree/master/handlebars/src/main/java/com/github/jknack/handlebars/helper) (template functions) provided by the [Java Handlebars implementation](https://github.com/jknack/handlebars.java) plus all of the string and conditional helpers are available.
- A separate set of [WireMock helpers](https://github.com/wiremock/wiremock/tree/master/src/main/java/com/github/tomakehurst/wiremock/extension/responsetemplating/helpers) is also available.

#### String helpers

- [mappings/helpers-string.json](mappings/helpers-string.json)
- Helpers:
  - [string helpers](https://github.com/jknack/handlebars.java/blob/master/handlebars/src/main/java/com/github/jknack/handlebars/helper/StringHelpers.java)

```console
$ curl http://localhost:8080/helpers/string/ | json_pp
{
   "abbreviate" : "Truncate long sentence up...",
   "capitalize" : {
      "capitalize first letter of all words" : "ONLY First Letter Capitalized",
      "capitalize first letter of all words AND lower case other characters" : "Fully First Letter Capitalized"
   },
   "capitalizeFirst" : "Only first character",
   "center" : {
      "center a string" : "      centerAStringWithEmptySpaces      ",
      "center string with padding" : "****centerAStringWithAsteriskPadding****"
   },
   "cut" : {
      "remove number 7 from a string" : "string  with  number s",
      "remove spaces from a string" : "stringwithspaces"
   },
   "dateFormat" : {
      "display current date and time in custom format" : "2023-12-05 01:09:13",
      "display current date in full format" : "Tuesday, 5 December 2023",
      "display current date in medium format" : "5 Dec 2023",
      "display current date in short format" : "05/12/2023",
      "parse timestamp from value in specific format, and display in full format" : "Monday, 21 June 2021"
   },
   "defaultIfEmpty" : {
      "set NOTHING as value provided value is empty " : "NOTHING",
      "set empty string if value provided is empty" : ""
   },
   "join" : {
      "join a list of items with custom joiner (last item in list is considered the joiner)" : "a-b-c",
      "join a list of items with custom joiner and prefix" : "[a-b",
      "join a list of items with custom joiner and suffix" : "a-b]",
      "join a list of items with custom joiner, prefix and suffix" : "[a-b]"
   },
   "ljust" : {
      "left align a given string in a 30 width space" : "left aligned                  ",
      "left align a given string in a 30 width space with padding" : "left aligned******************"
   },
   "lower" : "change value to lower case",
   "now" : {
      "display current date time" : "2023-12-05T01:09:13Z",
      "display current date time in custom format" : "2023-12-05 01:09:13.000067"
   },
   "numberFormat" : {
      "format number in currency format" : "£30.00",
      "format number in currency format with locale" : "30,00 ¤",
      "format number in custom decimal format" : "3,000,000.000",
      "format number in integer format" : "30",
      "format number in percent format" : "3,000%",
      "format number with defined maximum integer and fraction digits" : "42.373",
      "format number with defined minimum integer and fraction digits" : "00.370"
   },
   "replace" : "Replaces placeholder with provided replacement",
   "rjust" : {
      "right align a given string in a 30 width space" : "                 right aligned",
      "right align a given string in a 30 width space with padding" : "*****************right aligned"
   },
   "slugify" : "creates-a-slug-useful-for-blog-post-urls",
   "stringFormat" : {
      "boolean" : "isSet: true isNotSet: false",
      "string" : "applies string formatting capabilities. All Java formatting options supported."
   },
   "stripTags" : "Removes all (X)HTML tags",
   "substring" : {
      "substring from 3rd (exclusive) to 7th character" : "3456",
      "substring from 5th character (exclusive)" : "56789"
   },
   "upper" : "CHANGE VALUE TO UPPER CASE",
   "yesno" : {
      "set true/false/null to ndio/hapana/labda (Swahili)" : "ndio | hapana | labda",
      "set true/false/null to sí/no/quizás (Spanish)" : "sí | no | quizás",
      "set true/false/null to yes/no/maybe" : "yes | no | maybe"
   }
}
```

#### Conditional helpers

- `eq`:
  - `{{#assign 'id'}}{{request.query.id}}{{/assign}}{{#eq id '18'}}id is 18{{else}}id is not 18{{/eq}}`
  - `{{#eq id '18'}}id is 18{{else eq id '19'}}id is 19{{else}}id is not 18 or 19{{/eq}}`
  - `{{eq id '18'}}`
  - `{{eq id '18' yes='id is 18' no='id is not 18'}}`
- `or`:
  - `{{#assign 'id'}}{{request.query.id}}{{/assign}}{{#or (eq id '18') (eq id '19')}}id is 18 or 19{{else}}id is not 18 or 19{{/or}}`
  - `{{or (eq id '18') (eq id '19') yes='id is 18 or 19' no='id is not 18 or 19'}}`
- [mappings/helpers-conditional.json](mappings/helpers-conditional.json)
- Helpers:
  - [conditional helpers](https://github.com/jknack/handlebars.java/blob/master/handlebars/src/main/java/com/github/jknack/handlebars/helper/ConditionalHelpers.java)

```console
$ curl http://localhost:8080/helpers/conditional/eq/?id=18 | python3 -m json.tool
{
    "eq with else": "id is 18",
    "eq with multiple else": "id is 18",
    "eq rendering 'true' or 'false'": "true",
    "eq rendering inlined values": "id is 18"
}

$ curl http://localhost:8080/helpers/conditional/eq/?id=19 | python3 -m json.tool
{
    "eq with else": "id is not 18",
    "eq with multiple else": "id is 19",
    "eq rendering 'true' or 'false'": "false",
    "eq rendering inlined values": "id is not 18"
}

$ curl http://localhost:8080/helpers/conditional/eq/?id=20 | python3 -m json.tool
{
    "eq with else": "id is not 18",
    "eq with multiple else": "id is not 18 or 19",
    "eq rendering 'true' or 'false'": "false",
    "eq rendering inlined values": "id is not 18"
}

$ curl http://localhost:8080/helpers/conditional/or/?id=18 | python3 -m json.tool
{
    "or with else": "id is 18 or 19",
    "or rendering inlined values": "id is 18 or 19"
}

$ curl http://localhost:8080/helpers/conditional/or/?id=19 | python3 -m json.tool
{
    "or with else": "id is 18 or 19",
    "or rendering inlined values": "id is 18 or 19"
}

$ curl http://localhost:8080/helpers/conditional/or/?id=20 | python3 -m json.tool
{
    "or with else": "id is not 18 or 19",
    "or rendering inlined values": "id is not 18 or 19"
}
```

### Date and time helpers

- `{{now}}`
- `{{now format='HH:mm:ss'}}`
- `{{now offset='5 minutes' format='HH:mm:ss'}}`
- `{{now offset='5 minutes' format='unix'}}`
- `{{date}}`
- `{{date (parseDate request.query.date format='dd/MM/yyyy HH:mm:ss') format='HH:mm:ss' offset='5 minutes'}}`
- [mappings/helpers-datetime.json](mappings/helpers-datetime.json)
- Helpers:
  - [`now` and `date`](https://github.com/wiremock/wiremock/blob/master/src/main/java/com/github/tomakehurst/wiremock/extension/responsetemplating/helpers/HandlebarsCurrentDateHelper.java) (same implementation code): Render date/time.
  - [`parseDate`](https://github.com/wiremock/wiremock/blob/master/src/main/java/com/github/tomakehurst/wiremock/extension/responsetemplating/helpers/ParseDateHelper.java): Parse dates.

```console
$ curl http://localhost:8080/helpers/datetime/?date=01/02/2003+14:15:16 | python3 -m json.tool
{
    "now": "2023-12-29T02:30:38Z",
    "now with format": "02:30:38",
    "now + 5 mins": "02:35:38",
    "now + 5 mins, UNIX epoch": "1703817338",
    "date": "2023-12-29T02:30:38Z",
    "date parsed from query string": "2003-02-01T14:15:16Z",
    "date with format": "14:15:16",
    "date + 5 mins": "14:20:16"
}
```

### Random helpers

- `{{pickRandom 'lorem' 'ipsum' 'dolor' 'sit' 'amet'}}`
- `{{pickRandom parameters.values}}`
- `{{randomValue length=9 type='ALPHABETIC'}}`
- `{{randomValue length=9 type='NUMERIC'}}`
- `{{randomValue type='UUID'}}`
- `{{randomInt lower=1 upper=10}}`
- `{{randomDecimal lower=-1.0 upper=-0.1}}`
- [mappings/helpers-random.json](mappings/helpers-random.json)
- Helpers:
  - [`pickRandom`](https://github.com/wiremock/wiremock/blob/master/src/main/java/com/github/tomakehurst/wiremock/extension/responsetemplating/helpers/PickRandomHelper.java): Randomly select a value from a list.
  - [`randomValue`](https://github.com/wiremock/wiremock/blob/master/src/main/java/com/github/tomakehurst/wiremock/extension/responsetemplating/helpers/HandlebarsRandomValuesHelper.java): Generate random strings of various kinds.
  - [`randomInt`](https://github.com/wiremock/wiremock/blob/master/src/main/java/com/github/tomakehurst/wiremock/extension/responsetemplating/helpers/RandomIntHelper.java): Generate random integers.
  - [`randomDecimal`](https://github.com/wiremock/wiremock/blob/master/src/main/java/com/github/tomakehurst/wiremock/extension/responsetemplating/helpers/RandomDecimalHelper.java): Generate random decimals.

```console
$ curl http://localhost:8080/helpers/random/ | python3 -m json.tool
{
    "pick random value from inlined values": "amet",
    "pick random value from transformer parameters": "elit",
    "alphabetic": "mxrsjpeql",
    "numeric": "174094805",
    "UUID": "7912a0ba-4233-4ba3-b3dc-42c492e759e8",
    "randomInt": "2",
    "randomDecimal": "-0.5129736337938928"
}

$ curl http://localhost:8080/helpers/random/ | python3 -m json.tool
{
    "pick random value from inlined values": "sit",
    "pick random value from transformer parameters": "consectetur",
    "alphabetic": "feibypvfq",
    "numeric": "638192223",
    "UUID": "93680621-4f50-4591-8029-79e78758f20d",
    "randomInt": "6",
    "randomDecimal": "-0.18499845699253625"
}
```

### Base64 helper

- `{{#assign 'encoded'}}{{base64 request.query.value}}{{/assign}}{{encoded}}`
- `{{base64 encoded decode=true}}`
- `{{#base64}}lorem{{/base64}}`
- `{{#base64 decode=true}}bG9yZW0={{/base64}}`
- [mappings/helpers-base64.json](mappings/helpers-base64.json)
- Helper:
  - [`base64`](https://github.com/wiremock/wiremock/blob/master/src/main/java/com/github/tomakehurst/wiremock/extension/responsetemplating/helpers/Base64Helper.java): Base64 encode and decode values.

```console
$ curl http://localhost:8080/helpers/base64/?value=lorem+ipsum | python3 -m json.tool
{
    "encoded": "bG9yZW0gaXBzdW0=",
    "decoded": "lorem ipsum",
    "encoded block": "bG9yZW0=",
    "decoded block": "lorem"
}
```

### Contains helper

- `{{#assign 'value'}}{{request.query.value}}{{/assign}}{{#contains 'lorem' value}}lorem contains {{value}}{{/contains}}`
- `{{#if (contains 'lorem' value)}}lorem contains {{value}}{{else}}lorem does not contain {{value}}{{/if}}`
- `{{#contains parameters.values value}}{{parameters.values}} contains {{value}}{{/contains}}`
- [mappings/helpers-contains.json](mappings/helpers-contains.json)
- Helpers:
  - [`contains`](https://github.com/wiremock/wiremock/blob/master/src/main/java/com/github/tomakehurst/wiremock/extension/responsetemplating/helpers/ContainsHelper.java)
    - Returns a boolean value indicating whether the string or array passed as the first parameter contains the string passed in the second.
    - Note: `"string contains": "{{#contains 'lorem' request.query.value}}lorem contains {{value}}{{/contains}}"` will _not_ work:
      - `java.lang.ClassCastException: class com.github.tomakehurst.wiremock.common.ListOrSingle cannot be cast to class java.lang.String`
  - [`if`](https://github.com/jknack/handlebars.java/blob/master/handlebars/src/main/java/com/github/jknack/handlebars/helper/IfHelper.java): Conditionally render a block. If its argument returns false, null or empty list/array (a "falsy" value), Handlebars will not render the block.
  - [`else`](https://github.com/jknack/handlebars.java/blob/master/handlebars/src/main/java/com/github/jknack/handlebars/internal/Block.java)

```console
$ curl http://localhost:8080/helpers/contains/?value=ore | python3 -m json.tool
{
    "string contains": "lorem contains ore",
    "contains with if and else": "lorem contains ore",
    "array contains": ""
}

$ curl http://localhost:8080/helpers/contains/?value=orb | python3 -m json.tool
{
    "string contains": "",
    "contains with if and else": "lorem does not contain orb",
    "array contains": ""
}

$ curl http://localhost:8080/helpers/contains/?value=lorem | python3 -m json.tool
{
    "string contains": "lorem contains lorem",
    "contains with if and else": "lorem contains lorem",
    "array contains": "[lorem, ipsum, dolor, sit, amet] contains lorem"
}
```

### Matches helper

- `{{#assign 'string'}}{{request.query.string}}{{/assign}}{{#matches string 'lor.+'}}lor.+ matches {{string}}{{/matches}}`
- `{{#if (matches string 'l[a-z]+m')}}l[a-z]+m matches {{string}}{{else}}l[a-z]+m does not match {{string}}{{/if}}`
- [mappings/helpers-matches.json](mappings/helpers-matches.json)
- [`matches`](https://github.com/wiremock/wiremock/blob/master/src/main/java/com/github/tomakehurst/wiremock/extension/responsetemplating/helpers/MatchesRegexHelper.java)
  - Returns a boolean value indicating whether the string passed as the first parameter matches the regular expression passed in the second.
    - Note: Regular expressions with backslash+character (e.g., `\w`) does not seem to work.
    - `l\\w+m` and `l\\\\w+m` do not match `lorem`. (Double backslashes due to escaping in JSON.)

```console
$ curl http://localhost:8080/helpers/matches/?string=lorem | python3 -m json.tool
{
    "string matches": "lor.+ matches lorem",
    "matches with if and else": "l[a-z]+m matches lorem"
}

$ curl http://localhost:8080/helpers/matches/?string=l1rem | python3 -m json.tool
{
    "string matches": "",
    "matches with if and else": "l[a-z]+m does not match l1rem"
}

```

### Regular expression extract helper

- `{{regexExtract request.query.string 're.+sum'}}`
- `{{regexExtract request.query.string 'pre.+sum' default='dolor sit'}}`
- `{{regexExtract request.query.string '([^ ]+) ([a-z]+)' 'lorem_parts'}}{{lorem_parts.1}} and {{lorem_parts.0}}`
- [mappings/helpers-regexextract.json](mappings/helpers-regexextract.json)
- [`regexExtract`](https://github.com/wiremock/wiremock/blob/master/src/main/java/com/github/tomakehurst/wiremock/extension/responsetemplating/helpers/RegexExtractHelper.java)
  - Extracts values matching a regular expression from a string:
    - single value
    - single value with default if no match
    - multiple values/parts

```console
$ curl http://localhost:8080/helpers/regexextract/?string=lorem+ipsum | python3 -m json.tool
{
    "single value": "rem ipsum",
    "single value with default": "dolor sit",
    "regex groups": "ipsum and lorem"
}

$ curl http://localhost:8080/helpers/regexextract/?string=lorem+ipsam | python3 -m json.tool
{
    "single value": "[ERROR: Nothing matched re.+sum]",
    "single value with default": "dolor sit",
    "regex groups": "ipsam and lorem"
}
```

## References

- "Stubbing." _WireMock_, 17 Nov. 2023, [wiremock.org/docs/stubbing/](https://wiremock.org/docs/stubbing/). Accessed 20 Nov. 2023.
- "Running as a Standalone Process." _WireMock_, 17 Nov. 2023, [wiremock.org/docs/standalone/java-jar/#command-line-options](https://wiremock.org/docs/standalone/java-jar/#command-line-options). Accessed 21 Nov. 2023.
- "Response Templating." _WireMock_, 17 Nov. 2023, [wiremock.org/docs/response-templating/](https://wiremock.org/docs/response-templating/). Accessed 21 Nov. 2023.
- Muya, Fred. "Using Wiremock to Mock API Responses: Part 3 - Response Templating Using Handlebars Helpers." _Muya's Blog_, 18 Oct. 2021, [blog.muya.co.ke/wiremock-response-templating-part-3/](https://blog.muya.co.ke/wiremock-response-templating-part-3/). Accessed 5 Dec. 2023.
