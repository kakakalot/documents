## flow stripe
- gen rsa pub/pri key from http://travistidwell.com/jsencrypt/demo/index.html
- public key -> app save
- secret key -> server save
- app call server api
- encrypt algorithm: **encrypt(json data)**

## API:
- ***/v2/stripe/checkout***
    - app encrypt request json and submit
    - server decrypt request and parse to JSON object, then call stripe `create_token` function `https://stripe.com/docs/api/node#create_card_token`
    - request:
    ```json
    {
        "number": '4242424242424242',
        "exp_month": 12,
        "exp_year": 2018,
        "cvc": '123'
    }
    ```
    - response:
    ```json
    //success
    {
        "token": "xyz"
    }

    //Error
    { type: "x", message : "yyy", code: "zzz" }
    ```
- ***/v2/stripe/charge***
    - app encrypt request json and submit
    - server decrypt request and parse json object
    - server call retrieve token to check if this token was used, `https://stripe.com/docs/api/node#retrieve_token`
    - after checking token success, server call charge function, `https://stripe.com/docs/api/node#create_charge`
    - request:
    ```json
    { "token": "xyx", "product_id": "abc" }
    ```
## **Credit Card Info storage**
#### _Create table:_
```yml
tbl_stripe_user:
    id
    user_id
    stripe_user_id
```
```yml
tbl_stripe_card:
    id
    stripe_card_id
    is_default
    brand
    fingerprint
    funding
    exp_month
    exp_year
    country
    last4
```

- ***/v2/stripe/card/create***
    - this API will be called after checkout
    - request JSON:
    ```json
    { "token": "xyz" }
    ```
    - app encrypt request json and submit
    - server decrypt request and parse json object
    - server call retrieve token to check if this token was used, `https://stripe.com/docs/api/node#retrieve_token`
    - after checking token success, do the following these:
        - _if THIS USER **HAVE NOT** registered on Stripe yet._ Server call create customer function, `https://stripe.com/docs/api#create_customer`

            **NOTE:** In the json that send to stripe, need to input `metadata` field
            ```json
            { "metadata": { "user_id": "xxx" }}
            ```
        - _if THIS USER **ALREADY HAVE** registered on Stripe._ Server call create card function, `https://stripe.com/docs/api#create_card`
    - The server must response HTTP code `200` or not

- ***/v2/stripe/card/default***
    - This API is used to update default card of specified user
    - app encrypt request json and submit
    - Server will call update customer function to update data of stripe
    `https://stripe.com/docs/api#update_customer`
    - Request
    ```js
    {
        card_id: "xyz"
    }
    ```

- ***/v2/stripe/cards***
    - This api use method `GET`
    - List all card of specified user
    - Response is an array
    ```js
    {
        id: "xxxx",
        stripe_card_id: "xxx",
        is_default: true, //false
        ...
    }
    ```
    
- ***/v2/stripe/charge***
    - The logic of this api is same with the above one
    - It just supports more request data format
    ```js
    { 
        product_id: "abc" 
    }
    ```
