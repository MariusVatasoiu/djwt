# djwt

the absolute minimum to make JSON Web Tokens on deno. Based on
[JWT](https://tools.ietf.org/html/rfc7519) and
[JWS](https://www.rfc-editor.org/rfc/rfc7515.html) specifications.

## Features

After the **Compact Serialization** process where a web token is represented as
the concatenation of:

`'BASE64URL(UTF8(JWS Protected Header))' || '.' || 'BASE64URL(JWS Payload)' ||'.'|| 'BASE64URL(JWS Signature)'`

...the finalized **JWT** looks like this:

```
 eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9
 .
 eyJpc3MiOiJqb2UiLA0KICJleHAiOjEzMDA4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ
 .
 dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
```

### Critical Header Parameter (crit)

It supports the **Critical Header Parameter** like described in the
[JWS specification](https://www.rfc-editor.org/rfc/rfc7515.html#section-4.1.11).

You can find [here](https://github.com/timonson/djwt/tree/master/examples) an
example which includes the _crit_ header parameter.

## API

The API consists mostly of the two functions `makeJwt` and `validateJwt`.

You can omit the JWT payload and its claims if you only need the signing and
verification feature of the JWS.

#### makeJwt(headerObject: Jose, claims: Claims | string = "", key: string)

#### validateJwt(jwt: string, key: string, throwErrors: boolean = true, criticalHandlers: CritHandlers = {})

Additionally there is the helper function `setExpiration` which simplifies
setting an expiration date.

#### setExpiration(exp: number | Date): number

```javascript
// A specific date:
setExpiration(new Date("2020-07-01"))
// One hour from now:
setExpiration(new Date().getTime() + 60 * 60 * 1000)
```

## Example

Try it out with this simple server example. The server will respond to a **GET**
request with a newly created **JWT**. On the other hand, if you send a **JWT**
as data along with a **PUT** request to the server, it will check the JWT for
validity.

```javascript
import { serve } from "https://deno.land/std/http/server.ts"
import { encode, decode } from "https://deno.land/std/strings/mod.ts"
import makeJwt, {
  setExpiration,
} from "https://cdn.jsdelivr.net/gh/timonson/djwt@0.2.1/create.ts"
import validateJwt from "https://cdn.jsdelivr.net/gh/timonson/djwt@0.2.1/validate.ts"

const key = "abc"
const claims = {
  iss: "joe",
  exp: setExpiration(new Date().getTime() + 60_000),
}
const headerObject = {
  alg: "HS512",
  typ: "JWT",
}
;(async () => {
  for await (const req of serve("0.0.0.0:8000")) {
    if (req.method === "GET") {
      const jwt = makeJwt(headerObject, claims, key)
      req.respond({ body: encode(jwt + "\n") })
    } else {
      const requestBody = decode(await req.body())
      validateJwt(requestBody, key, false)
        ? req.respond({ body: encode("Valid JWT\n") })
        : req.respond({ status: 401, body: encode("Invalid JWT\n") })
    }
  }
})()
```

## Todo

1. Add more optional features from the
   [JWT](https://tools.ietf.org/html/rfc7519) and
   [JWS](https://www.rfc-editor.org/rfc/rfc7515.html) specifications
2. Improve documentation
3. Make more tests