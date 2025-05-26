# Neverlose.cc API docs

<!-- vim-markdown-toc GFM -->

* [Recent changes](#recent-changes)
* [Webhook callbacks](#webhook-callbacks)
    * [Balance transfer](#balance-transfer)
    * [Item purchase](#item-purchase)
* [API requests](#api-requests)
    * [Endpoint](#endpoint)
    * [Common parameters](#common-parameters)
    * [Request id](#request-id)
    * [Curl example](#curl-example)
    * [Responses](#responses)
        * [Successful response](#successful-response)
        * [Failure response](#failure-response)
    * [Product list](#product-list)
* [API methods](#api-methods)
    * [Give item to user](#give-item-to-user)
    * [Transfer balance](#transfer-balance)
    * [Gift product](#gift-product)
    * [Get product prices](#get-product-prices)
    * [Check if user invited to product](#check-if-user-invited-to-product)
    * [Check if user exists](#check-if-user-exists)
    * [Get NLE balance](#get-nle-balance)
* [Signature creation and validation](#signature-creation-and-validation)
    * [Python example](#python-example)
    * [Javascript example](#javascript-example)
    * [PHP library](#php-library)
    * [JavaScript & TypeScript SDK](#javascript--typescript-sdk)
* [Example webhook callback handlers](#example-webhook-callback-handlers)
    * [python + bottle](#python--bottle)
    * [nodejs + express](#nodejs--express)
* [OAuth + OpenID identity provider](#oauth--openid-identity-provider)
    * [Endpoints](#endpoints)
    * [OAuth behavior](#oauth-behavior)
    * [Supported OAuth scopes](#supported-oauth-scopes)
    * [Supported OpenID Userinfo claims](#supported-openid-userinfo-claims)
* [Reseller integration](#reseller-integration)
    * [How it works?](#how-it-works)
    * [Setting integration visibiliy using API](#setting-integration-visibiliy-using-api)
    * [Providing your prices](#providing-your-prices)
    * [JWT token contents](#jwt-token-contents)

<!-- vim-markdown-toc -->

## Recent changes

- 14 Apr 2025
  - Added integration-visibility endpoint
- 22 Dec 2024
  - Add native reseller integration docs
- 02 Dec 2024
  - New api domain / endpoint
- 28 Jan 2024
  - Added OAuth and OpenID info
- 08 Dec 2023
  - Added `is-user-invited` method
  - Updated product list
- 27 Dec 2022
  - It's now possible to use string unique request IDs
- 25 Apr 2022
  - Added `get-balance` method
  - Webhooks are now have `kind` parameter,   
    so they can be distinguished easily when sent to single endpoint
  - Updated docs
- 04 Feb 2022
  - Added `is-user-exists` method
  - Added `get-prices` method
  - Success/failure responses are now consistent between methods (without breaking changes)

## Webhook callbacks

Event notifications sent to your server as POST requests with json body.
You can set webhook urls in market api settings

For signature verification refer to [signatures section](#signature-creation-and-validation).
You should always check signature validity and ignore any events with invalid signature

You can also check example webhook callback handler [here](#example-callback-handler-using-python--bottle)

### Balance transfer

This event is sent when balance is transferred to your account

```json
{
  "kind": "transfer",
  "amount": 1,
  "username": "A49",
  "unique_id": 22,
  "signature": "51a6c1c281c4cd8c4e020f89cc3bdca6aa6c0747fc33f7112ef041064bd864a2"
}
```

| Parameter   | Description              |
| ----------- | ------------------------ |
| `kind`      | Event type               |
| `amount`    | Amount received          |
| `username`  | Sender username          |
| `unique_id` | Incrementing transfer id |
| `signature` | Event signature          |

### Item purchase

This event is sent when your item is purchased

```json
{
  "kind": "purchase",
  "amount": 0.9,
  "username": "A49",
  "unique_id": 89968,
  "item_id": "E3yugw",
  "signature": "dc20a4d73447ac51689d6e03115aa135a8d734e610352dda818e830e70a60560"
}
```

| Parameter   | Description              |
| ----------- | ------------------------ |
| `kind`      | Event type               |
| `username`  | Buyer                    |
| `unique_id` | Incrementing purchase id |
| `item_id`   | Bought item code         |
| `signature` | Event signature          |

## API requests

### Endpoint

API domain is:

`user-api.neverlose.cc`

Marketplace API endpoint is:

`user-api.neverlose.cc/api/market/<method>`

API requests should be sent with **POST** method and `Content-Type: application/json` 

> [!WARNING]
> Use of old domain/endpoint `neverlose.cc/api/market` is still supported but discouraged.  
> Users are advised to migrate to new endpoint as soon as possible

### Common parameters

Common parameters used in all types of actions (unless specified otherwise):

| Parameter   | Description       | Additional                                                                       |
| ----------- | ----------------- | -------------------------------------------------------------------------------- |
| `user_id`   | Your user id      | You can get it on market api settings page                                       |
| `signature` | Request signature | Refer to [signatures section](#signature-creation-and-validation)                |
| `id`        | Unique request id | Used to prevent erroneous repetitive requests. Not needed for read-only requests |

### Request id

`id` parameter should be unique between requests. If you haven't received a successful response
for your request it's safe to try it again with the same `id` parameter. If your action haven't
been completed it will be completed with second request, and if it was, and you failed to receive
the response previously for some unrelated reason you will receive `Invalid id.` error and you will not
spend funds twice.

**Unique id format**

| Parameter          | Limitation                                                                                                            |
| ------------------ | --------------------------------------------------------------------------------------------------------------------- |
| Type               | `integer` or `string` (`"id": 123` or `"id": "abcde123"`)                                                             |
| Max length         | 80 characters (for both numbers and strings)                                                                          |
| Allowed characters | **Lower** and **upper** case **latin** alphabet, **numbers**, and `.-_` are supported.<br>Whitespace is not supported |

### Curl example

```bash
curl 'https://user-api.neverlose.cc/api/market/give-for-free' \
  --data '{"user_id": 1, "signature": "..."}' \
  -X POST --header "Content-Type: application/json"
```

Replace data and url in example above depending on action you need to do

### Responses

#### Successful response

```json
{
  "succ": true,
  "success": true
}
```

Successful responses may contain additional fields, refer to methods documentation below.

`succ` field is left for backwards compatibility with older api revisions and always have the same value as `success`.

#### Failure response

```json
{
  "succ": false,
  "success": false,
  "error": "Error message"
}
```

### Product list

List of valid products you can specify in `product` request field of methods below:

| `product` name | Game name |
| -------------: | --------- |
|         `csgo` | CS:GO     |
|          `cs2` | CS2       |

## API methods

### Give item to user

URL: `/api/market/give-for-free`

```json
{
  "user_id": 1,
  "id": 1338,
  "username": "darth",
  "code": "E3yugw",
  "signature": "c0e8a7fa9c9fafe16d21ad0be087a6372bb7a9256fab212ff106666a152c6e0a"
}
```

This request gives item `E3yugw` to user `darth`

| Parameter  | Description                          |
| ---------- | ------------------------------------ |
| `username` | Username that will receive this item |
| `code`     | Market code of item you want to give |

### Transfer balance

**This method is available for official resellers only**

URL: `/api/market/transfer-money`

```json
{
  "user_id": 1,
  "id": 1337,
  "username": "a49",
  "amount": 2.00,
  "signature": "32208d45c593478eceb0e15aa0f8013a1259c1ef32f755edf2c49b9df2072aa2"
}
```

This request transfers `2 NLE` to user `a49`

| Parameter  | Description                        |
| ---------- | ---------------------------------- |
| `username` | Username that will receive NLE     |
| `amount`   | Amount of NLE you want to transfer |

### Gift product

**This method is available for official resellers only**

URL: `/api/market/gift-product`

```json
{
  "user_id": 1,
  "id": 2,
  "username": "darth",
  "product": "csgo",
  "cnt": 0,
  "signature": "32208d45c593478eceb0e15aa0f8013a1259c1ef32f755edf2c49b9df2072aa2"
}
```

This request gifts `30 days` for *CS:GO* to user `darth`

| Parameter  | Description                         |
| ---------- | ----------------------------------- |
| `username` | Username that will receive product  |
| `product`  | Product name                        |
| `cnt`      | Upgrade type (refer to table below) |

`cnt` parameter:

| `cnt` | Price     | Days |
| ----- | --------- | ---- |
| `0`   | 17.1 NLE  | 30   |
| `1`   | 44.1 NLE  | 90   |
| `2`   | 80.1 NLE  | 180  |
| `3`   | 134.1 NLE | 365  |

For converted RUB prices refer to `get-prices` method below

### Get product prices

**This method is available for official resellers only**

URL: `/api/market/get-prices`

```json
{
  "user_id": 1,
  "product": "csgo",
  "signature": "..."
}
```

| Parameter | Description  |
| --------- | ------------ |
| `product` | Product name |

*Read-only method, `id` parameter is not needed here*

Response:

```json
{
  "succ": true,
  "success": true,
  "prices": {
    "30": {
      "cnt": 0,
      "eur": 17.1,
      "rub": 1368
    },
    "90": {
      "cnt": 1,
      "eur": 44.1,
      "rub": 3529
    },
    "180": {
      "cnt": 2,
      "eur": 80.1,
      "rub": 6410
    },
    "365": {
      "cnt": 3,
      "eur": 134.1,
      "rub": 10731
    }
  }
}
```

This request will return current prices for selected product

### Check if user invited to product

**This method is available for official resellers only**

URL: `/api/market/is-user-invited`

```json
{
  "user_id": 1,
  "username": "target_user",
  "product": "cs2",
  "signature": "..."
}
```

| Parameter  | Description            |
| ---------- | ---------------------- |
| `product`  | Product name           |
| `username` | Login of user to check |

*Read-only method, `id` parameter is not needed here*

Response:

```json
{
  "succ": true,
  "success": true,
  "cheat_public": false,
  "user_invited": true
}
```

- will return `cheat_public=true` if cheat does not require invite
  (`user_invited` will always be true in this case)
- when `cheat_public=false`, `user_invited` determines whether
  this user is invited to this product or not.
- **You won't be able to gift subscription
  to non-invited user when cheat is not public!**

### Check if user exists

**This method is available for official resellers only**

URL: `/api/market/is-user-exists`

```json
{
  "user_id": 1,
  "username": "darth",
  "signature": "..."
}
```

| Parameter  | Description       |
| ---------- | ----------------- |
| `username` | Username to check |

*Read-only method, `id` parameter is not needed here*

Response:

```json
{
  "success": true,
  "succ": true,
  "user_exists": true
}
```

`user_exists` field will be `true` if user `darth` exists, `false` otherwise

### Get NLE balance

URL: `/api/market/get-balance`

```json
{
  "user_id": 1,
  "signature": "..."
}
```

*Read-only method, `id` parameter is not needed here*

Response:

```json
{
  "succ": true,
  "success": true,
  "balance": 62.6
}
```

## Signature creation and validation

### Python example

```python
#!/usr/bin/python3
from hashlib import sha256


def market_api_generate_signature(j, secret):
    str_to_hash = ("".join([i + str(j[i]) for i in sorted(j)]) + secret).encode()
    # print(str_to_hash)
    hashed = sha256(str_to_hash).hexdigest()
    return hashed


def market_api_validate_signature(j, secret):
    nl_sig = j["signature"]
    del j["signature"]
    our_sig = market_api_generate_signature(j, secret)
    return nl_sig == our_sig


# Validation

event_data = {
    "amount": 0.9,
    "username": "A49",
    "unique_id": 89968,
    "item_id": "E3yugw",
    "signature": "dc20a4d73447ac51689d6e03115aa135a8d734e610352dda818e830e70a60560"
}
assert (market_api_validate_signature(event_data, "key") == True)

# Generation

request_data = {
    "user_id": 1,
    "id": 1337,
    "username": "a49",
    "amount": 2.00,
}
request_data.update(signature=market_api_generate_signature(request_data, "key"))
```

### Javascript example

```javascript
"use strict";
let crypto = require("crypto");

const sort_obj = function(obj) {
  return Object.keys(obj).sort().reduce(function (result, key) {
    result[key] = obj[key];
    return result;
  }, {});
}

const obj_to_string = function (obj) {
  obj = sort_obj(obj);
  let str = '';
  for (let p in obj) {
      if (obj.hasOwnProperty(p)) {
          str += p + obj[p];
      }
  }
  return str;
}

const market_api_generate_signature = function(j, secret){
    const str_to_hash = obj_to_string(j) + secret
    const hashed = crypto.createHash('sha256').update(str_to_hash).digest('hex');
    return hashed
}
const market_api_validate_signature = function(j, secret){
  const nl_sig = j["signature"]
  delete j["signature"]
  const our_sig = market_api_generate_signature(j, secret)
  return nl_sig === our_sig
}

// Validation

let data = {
  "amount": 0.9,
  "username": "A49",
  "unique_id": 89968,
  "item_id": "E3yugw",
  "signature": "dc20a4d73447ac51689d6e03115aa135a8d734e610352dda818e830e70a60560"
}
let sign_valid = market_api_validate_signature(data, "key")
console.log(sign_valid)
```

### PHP library

```bash
composer require rainedot/php-nl-market
```

```php
require("vendor/autoload.php");
$api = new \Rainedot\PhpNlMarket\MarketAPI('YOUR_API_KEY', 1);
$api->validateRequest(array $request); // Returns true if request is valid
```


### JavaScript & TypeScript SDK

[🔗 nl-market-sdk on GitHub](https://github.com/Walkaisa/nl-market-sdk) • [📖 Full Documentation](https://github.com/Walkaisa/nl-market-sdk#readme)

```bash
# npm
npm install nl-market-sdk

# pnpm (recommended)
pnpm add nl-market-sdk
```

*…or yarn / bun – pick your package manager.*

```ts
import { MarketClient } from "nl-market-sdk";

const client = new MarketClient({
  userId: "12345",              // Your user ID from API settings.
  secret: "SUPER_SECRET_KEY",   // Your secret from API settings.
  // integrationId: 100         // Only needed if using checkout integration.
});

const res = await client.getBalance();
if (res.success) console.log(`💰 ${res.balance} NLE left`);
```

## Example webhook callback handlers

### python + bottle

```python
#!/usr/bin/python3

from bottle import run, request, post
import bottle
from hashlib import sha256

SECRET_KEY = "key"


def market_api_generate_signature(j, secret):
    str_to_hash = ("".join([i + str(j[i]) for i in sorted(j)]) + secret).encode()
    hashed = sha256(str_to_hash).hexdigest()
    return hashed


def market_api_validate_signature(j, secret):
    nl_sig = j["signature"]
    del j["signature"]
    our_sig = market_api_generate_signature(j, secret)
    return nl_sig == our_sig


@post("/on_purchase")
def on_purchase():
    data = request.json
    if not market_api_validate_signature(data, SECRET_KEY):
        return "invalid signature"
    res = data["username"] + " bought item https://neverlose.cc/market/item?id=" + data["item_id"]
    print(res, flush=True)
    return res


app = application = bottle.Bottle()
run(host='0.0.0.0', port=8080)
```

### nodejs + express

```javascript
const express = require('express')
const app = express()
const crypto = require("crypto")

const secret_key = "key"

const sort_obj = function(obj) {
  return Object.keys(obj).sort().reduce(function (result, key) {
    result[key] = obj[key]
    return result
  }, {})
}

const obj_to_string = function (obj) {
 obj = sort_obj(obj)
    let str = ''
    for (let p in obj) {
        if (obj.hasOwnProperty(p)) {
            str += p + obj[p]
        }
    }
    return str
}

const market_api_generate_signature = function(j, secret){
    const str_to_hash = obj_to_string(j) + secret
    const hashed = crypto.createHash('sha256').update(str_to_hash).digest('hex')
    return hashed
}

const market_api_validate_signature = function(j, secret){
  const nl_sig = j["signature"]
  delete j["signature"]
  const our_sig = market_api_generate_signature(j, secret)
  return nl_sig === our_sig
}

app.post('/on_purchase', (req, res) => {
  if (!market_api_validate_signature(req.body.data, secret_key))
    res.send("invalid signature")

  let response = req.body.data.username + " bought item https://neverlose.cc/market/item?id=" + req.body.data.item_id
  console.log(response)
  res.send(response)
})

app.listen(8080)
```

## OAuth + OpenID identity provider

Neverlose now supports OAuth authorization and being an OpenID idP.

Supported api versions and their docs:

- **OpenID Connect Core 1.0** (OIDC)
  - https://openid.net/specs/openid-connect-core-1_0.html
- **OAuth 2.0**
  - https://oauth.net/2

Client IDs are issued manually.  
If you need one, please open a new support ticket and describe your use-case,
we will gladly register your app if it meets our guidelines.

### Endpoints

| Endpoint      | URL                                              |
| ------------- | ------------------------------------------------ |
| Auth URI      | `https://auth2.neverlose.cc/oauth/authorize`     |
| Token URI     | `https://auth2.neverlose.cc/oauth/token`         |
| OIDC Userinfo | `https://auth2.neverlose.cc/oauth/oidc_userinfo` |

### OAuth behavior

- Supported **response types**: `code`
- Supported **grant types**: `authorization_code`
- **Refresh tokens** are **NEVER** issued, access_token expire time can be adjusted at request
- **`state` parameter** is required unless `no_state` is in scope
- **PKCE** (`code_challenge`) is required unless you have good reason to not implement it.
  `no_pkce` will not work unless you are permitted to use it
- **PKCE** **S256** is the only supported challenge method (`plain` is not supported)

### Supported OAuth scopes

| Scope      | Description                                                                 |
| ---------- | --------------------------------------------------------------------------- |
| `profile`  | Read access to user's login and profile picture                             |
| `email`    | Read access to user's email                                                 |
| `openid`   | Required to access OIDC endpoints                                           |
| `no_state` | Allows to omit state parameter during auth flow                             |
| `no_pkce`  | Allows to skip/omit PKCE during auth flow<br/>(requires special permission) |

### Supported OpenID Userinfo claims

| Field                | Content                                             | Scope needed |
| -------------------- | --------------------------------------------------- | ------------ |
| `sub`                | Numeric user id (presented as string per OIDC spec) | `openid`     |
| `preferred_username` | User's login                                        | `profile`    |
| `name`               | User's login                                        | `profile`    |
| `profile`            | URL to user's profile (forum)                       | `profile`    |
| `picture`            | URL to user's profile picture (PNG)                 | `profile`    |
| `email`              | User's email                                        | `email`      |

## Reseller integration

Reseller integration provides a way for you to integrate your service
into our payment/checkout UI.  
You can set up your integration at API settings page. If you don't see
"Reseller integration" settings section, and you're willing to use it,
please contact us through tickets section.

### How it works?

When user clicks on your payment method in our checkout UI, they will be
redirected to your website (provided in Redirect URL in market API settings),
and redirect URL will contain a query argument `nl_purchase` that hold JWT token with information 
about purchase. For example:

**Redirect URL in settings**: `https://test.local/checkout`  

**User will be redirected to**: `https://test.local/checkout?nl_purchase=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0ZXN0IjoicGF5bG9hZCIsImZvbyI6ImJhciIsImlzcyI6Ik5MUkkiLCJleHAiOjB9.yNvxOjAE0K8ns4YiMI7oV8XV3R8qGpf2my0HHQGDO_I`  

JWT payload format is explained in latter section 

After creating your integration and specifying valid Redirect URL,
your payment method will be present in method dropdown on product purchase page.

Please note that until you enable **Public** checkbox, your method is visible only
to you. You can use this private method to test your integration and set it to public as 
soon as integration is ready.

### Setting integration visibiliy using API

You can toggle "Public" checkbox using API.  
S

URL: `/api/market/integration-visibility`

**Request:**

```json
{
  "user_id": 1,
  "integration_id": 100,
  "is_public": true
}
```

| Parameter        | Description                                                                  |
| ---------------- | ---------------------------------------------------------------------------- |
| `integration_id` | ID of your reseller integration (check on api settings page)                 |
| `is_public`      | Visibility status: `true` for public (visible), `false` for hidden (testing) |

Response will be simple `success: true|false` response

### Providing your prices

User will also see your actual prices in our checkout UI if you provide them.  
You can provide your prices using this market api method:

URL: `/api/market/set-reseller-prices`

**Prices object:**

```json
{
  "cs2-30": {"EUR":  ["30.50", "31.30"], "USD":  ["31.82", "32.65"]},
  "csgo-180": {"EUR":  "14.10", "USD":  "14.71", "XMR": "0.162"},
  "marketplace": {"EUR": "1.1", "USD":  "1.15"}
}
```

**Request:**

```json
{
  "user_id": 1,
  "integration_id": 100,
  "prices": "<prices json>",
  "signature": "..."
}
```

| Parameter        | Description                                                    |
| ---------------- | -------------------------------------------------------------- |
| `integration_id` | ID of your reseller integration (check on api settings page)   |
| `prices`         | Product-Currency-Price object in JSON string (explained below) |

Response will be simple `success: true|false` response

Note that you need to **first compose prices object**, **serialize it to json**, then 
**put it as string** to `"prices"` key of request object. This is necessary to keep signature 
algorithm compatible. Example pseudo-Javascript code:

```js
const request = {
  user_id: 1,
  integration_id: 100,
  prices: JSON.stringify({
    "cs2-30": {...},
    "csgo-180": {...}
  })
}
request.signature = generateSignature(request)
const requestJson = JSON.stringify(request)
fetch("...", {method: "POST", body: requestJson, ...})
```

Price set using this method will automatically expire after 24 hours, so you should
update them periodically using automatic script, otherwise prices will disappear from checkout UI.  
You can check remaining time until expiration on API settings page. We recommend you to update 
prices **each hour** if they are changed frequently and/or dynamically. Otherwise, if your prices 
are set manually and mostly static, we recommend you to update them **each 12 hours.**

**Prices object:**

Example describes the following prices:

- You sell CS2 (30 days) for:
  - from 30.50 to 31.30 EUR
  - from 31.82 to 32.65 USD
  - *This means that you have multiple payment methods for this product that fit specified price range*
- You sell CS:GO (180 days) for:
  - 14.10 EUR
  - 14.71 USD
  - 0.162 XMR
  - *In this case there's exactly one price for all methods available for this product*
- You sell **1 NLE** for:
  - 1.1 EUR
  - 1.15 USD

Rules:

- First level key should be in format `<product_name>-<days>` 
  - Product names are same as in `gift-product` method
  - `days` should be existing plan presented as days count (30, 90, 180, etc.)
    - If you don't specify plan in key it will default to 30 days (i.e. `"cs2"` is same as `"cs2-30"`)
  - Special product `marketplace` sets price for **1 NLE** market topup
  - 2nd level key is currency name
    - Currency name should be exactly 3 uppercase latin letters. Any other names will result in invalid format error.
    - This object should contain up to 3 currencies. Specifying more currencies will result in an error
    - 2nd level value is price or price range for this product in this currency
      - Decimal values should be passed as strings to avoid rounding errors
      - To specify a *range* you should pass array with exactly 2 elements
        - First price in a range should be less than second one
      - To specify singular price you should pass number string as value without array


### JWT token contents

JWT token passed to your redirect URL (see "How it works?" section above) is constructed as following:

- Algorithm: `HS256`
- Secret key: your Neverlose API key
- Claims:
  - `iss` (Issuer) - always `NLRI`
  - `iat` (Issued at) - order creation timestamp
  - `exp` (Expire) - timestamp at which this token is not valid anymore (currently iat + 1 hour)

Mainstream JWT libraries should check these fields automatically (you'll only need to provide valid Issuer).

> [!CAUTION]
> We strongly encourage you to use proper JWT library for your language that properly checks token signature.  
> This is important to avoid phishing attacks and scam attempts.

**Payload fields:**

| Field                 | Description                                                                                          |
| --------------------- | ---------------------------------------------------------------------------------------------------- |
| `integration_id`      | ID of your reseller integration                                                                      |
| `login`               | Login of user that should receive the order                                                          |
| `email`               | Email of receiving user                                                                              |
| `product`             | Product name that user ordered (same names as in `gift-product`); `marketplace` if market top-up     |
| `market`              | `true` if NLE (market top-up) order, `false` if product order                                        |
| `cnt`                 | Plan number requested by user (same as in `gift-product`), `null` for market                         |
| `days` (product only) | Number of days to gift to user (provided for convenience, always in sync with `cnt` field)           |
| `nle` (market only)   | NLE amount requested by user (Decimal number **presented as string**, with `.` as decimal separator) |

After receiving this redirect you should direct user directly to payment method selection, 
or directly to order confirmation if you have only one method available for customers of this product.

After confirming user's payment, you should gift the product or transfer funds to login 
that was previously specified in JWT token using usual methods `gift-product` and `transfer-money` 

**Full payload example:**

For product:

```json
{
  "iss": "NLRI",
  "iat": 1734899417,
  "exp": 1734903017,
  "integration_id": 100,
  "product": "cs2",
  "cnt": 0,
  "login": "a47",
  "email": "foo@bar.baz",
  "market": false,
  "days": 30
}
```

For market:

```json
{
  "iss": "NLRI",
  "iat": 1734899417,
  "exp": 1734903017,
  "integration_id": 100,
  "product": "marketplace",
  "cnt": null,
  "login": "a47",
  "email": "foo@bar.baz",
  "market": true,
  "nle": "13.37"
}
```
