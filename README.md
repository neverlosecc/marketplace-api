# Neverlose.cc market api docs

<!-- vim-markdown-toc GFM -->

* [Webhook callbacks](#webhook-callbacks)
  * [Balance transfer](#balance-transfer)
  * [Item purchase](#item-purchase)
* [Api requests](#api-requests)
  * [Common parameters](#common-parameters)
  * [Curl example](#curl-example)
  * [Responses](#responses)
    * [Successful response](#successful-response)
    * [Failure response](#failure-response)
* [Api methods](#api-methods)
  * [Give item to user](#give-item-to-user)
  * [Transfer balance](#transfer-balance)
  * [Gift product](#gift-product)
  * [Get product prices](#get-product-prices)
  * [Check if user exists](#check-if-user-exists)
* [Signature creation and validation](#signature-creation-and-validation)
  * [Python example](#python-example)
  * [Javascript example](#javascript-example)
* [Example callback handler using python + bottle](#example-callback-handler-using-python--bottle)

<!-- vim-markdown-toc -->

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
|-------------|--------------------------|
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
|-------------|--------------------------|
| `kind`      | Event type               |
| `username`  | Buyer                    |
| `unique_id` | Incrementing purchase id |
| `item_id`   | Bought item code         |
| `signature` | Event signature          |


## Api requests

Api requests should be sent with POST method and `application/json` content-type

### Common parameters 

Common parameters used in all types of actions (unless specified otherwise):

| Parameter   | Description       | Additional                                                                       |
|-------------|-------------------|----------------------------------------------------------------------------------|
| `user_id`   | Your user id      | You can get it on market api settings page                                       |
| `signature` | Request signature | Refer to [signatures section](#signature-creation-and-validation)                |
| `id`        | Unique request id | Used to prevent erroneous repetitive requests. Not needed for read-only requests |

### Curl example

```bash
curl 'https://neverlose.cc/api/market/give-for-free' \
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

Successful responses may contain additional fields, refer to methods documentation below

#### Failure response

```json
{
  "succ": false,
  "success": false,
  "error": "Error message"
}
```

## Api methods

### Give item to user
URL: `/api/market/give-for-free`

```json
{
  "user_id": 1,
  "id": 1338,
  "username" : "darth", 
  "code": "E3yugw", 
  "signature": "c0e8a7fa9c9fafe16d21ad0be087a6372bb7a9256fab212ff106666a152c6e0a"
}
```

This request gives item `E3yugw` to user `darth`

| Parameter   | Description                          |
|-------------|--------------------------------------|
| `username`  | Username that will receive this item |
| `code`      | Market code of item you want to give |


### Transfer balance
**This method is available for official resellers only**

URL: `/api/market/transfer-money`

```json
{
  "user_id": 1,
  "id": 1337,
  "username" : "a49",
  "amount" : 2.00,
  "signature": "32208d45c593478eceb0e15aa0f8013a1259c1ef32f755edf2c49b9df2072aa2"
}
```

This request transfers `2 NLE` to user `a49`

| Parameter  | Description                        |
|------------|------------------------------------|
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
|------------|-------------------------------------|
| `username` | Username that will receive product  |
| `product`  | Only `csgo` product is available    |
| `cnt`      | Upgrade type (refer to table below) |

`cnt` parameter:

| `cnt` | Price     | Days |
|-------|-----------|------|
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

| Parameter  | Description                         |
|------------|-------------------------------------|
| `product`  | Only `csgo` product is available    |

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
|------------|-------------------|
| `username` | Username to check |

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
    #print(str_to_hash)
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
assert( market_api_validate_signature(event_data, "key") == True )

# Generation

request_data = {
  "user_id": 1,
  "id": 1337,
  "username" : "a49",
  "amount" : 2.00,
}
request_data.update(signature = market_api_generate_signature(request_data, "key"))
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


## Example callback handler using python + bottle

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
  res =  data["username"] + " bought item https://neverlose.cc/market/item?id=" + data["item_id"] 
  print(res, flush=True)
  return res    

app = application = bottle.Bottle()
run(host='0.0.0.0', port=8080)
```
