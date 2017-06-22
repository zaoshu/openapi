# User

* [帐号信息](#get-account-info)
* [钱包信息](#get-wallet-info)

## 帐号信息 

获取用户帐号信息。

    GET /user/account

### 返回

    Status: 200 OK

    {
      "code": 0,
      "data": {
        "email": "***@email.com",
        "status": "normal/unactived"
      },
      "msg": ""
    }



## 钱包信息 

获取用户钱包信息，包括帐号剩余造数点信息。

    GET /user/wallet

### 返回

    Status: 200 OK

    {
      "code": 0,
      "data": {
        "balance": 1000,
        "freeConsumed": 1000,
        "freeTotal": 1000,
      },
      "msg": ""
    }