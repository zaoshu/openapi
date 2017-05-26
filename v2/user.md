# User

* [Get Account Info](#get-account-info)
* [Get Wallet Info](#get-wallet-info)

## Get Account Info

Get info of user account.

    GET /user/account
### Response

    Status: 200 OK

```
{
  "code": 0,
  "data": {
  	"email": "***@email.com",
  	"status" "normal"
  },
  "msg": ""
}
```



## Get Wallet Info

Get info of user wallet, including the balance zcoins.

    GET /user/wallet
### Response

    Status: 200 OK

```
{
  "code" :0,
  "data": {
    "balance": 1000,
    "freeConsumed": 1000,
  	"freeTotal": 1000,
  },
  "msg": ""
}
```