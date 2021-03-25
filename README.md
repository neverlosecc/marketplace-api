
# callbacks:
## notification on some event. sent to your url from neverlose server

### balance tranfser
data:
```
{
  "amount": 1,
  "username": "A49",
  "unique_id": 22,
  "signature": "51a6c1c281c4cd8c4e020f89cc3bdca6aa6c0747fc33f7112ef041064bd864a2"
}
```

### item purchase
data:
```
{
  "amount": 0.9,
  "username": "A49",
  "unique_id": 89968,
  "item_id": "E3yugw",
  "signature": "dc20a4d73447ac51689d6e03115aa135a8d734e610352dda818e830e70a60560"
}
```


# post requests:
## does something by your request. sent from your server to neverlose by url
id should be unique.

### give item to username
URL: `/api/market/give-for-free`

data:
```
{
  "username" : "darth", 
  "user_id": 1, 
  "id": 1338, 
  "code": "E3yugw", 
  "signature": "c0e8a7fa9c9fafe16d21ad0be087a6372bb7a9256fab212ff106666a152c6e0a"
}
```
CURL example:
`curl "https://neverlose.cc/api/market/give-for-free" --data '{"username" : "darth", "user_id": 1, "id": 1338, "code": "E3yugw", "signature": "c0e8a7fa9c9fafe16d21ad0be087a6372bb7a9256fab212ff106666a152c6e0a"}' -X POST --header "Content-Type: application/json"`

### balance transfer
URL: `/api/market/transfer-money`
data:
```
{
  "amount" : 2, 
  "username" : "a49", 
  "user_id": 1, 
  "id": 1337, 
  "signature": "32208d45c593478eceb0e15aa0f8013a1259c1ef32f755edf2c49b9df2072aa2"
}
```
CURL example:
`curl "https://neverlose.cc/api/market/transfer-money" --data '{"amount" : 2, "username" : "a49", "user_id": 1, "id": 1337, "signature": "32208d45c593478eceb0e15aa0f8013a1259c1ef32f755edf2c49b9df2072aa2"}' -X POST --header "Content-Type: application/json"`




# signature validation / generation



~~~
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

data = {
  "amount": 0.9,
  "username": "A49",
  "unique_id": 89968,
  "item_id": "E3yugw",
  "signature": "dc20a4d73447ac51689d6e03115aa135a8d734e610352dda818e830e70a60560"
}
sign_valid = market_api_validate_signature(data, "key")
print(sign_valid)

data = {
  "amount": 1,
  "username": "A49",
  "unique_id": 21,
  "signature": "baf17e67c9b0389433c8c55283935b5fa27f73e86d33b7123027797d6927f51b"
}
sign_valid = market_api_validate_signature(data, "secret")
print(sign_valid)
~~~


# Example callback handler
using python + bottle

~~~
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
~~~