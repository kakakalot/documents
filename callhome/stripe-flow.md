## flow stripe
- gen rsa pub/pri key from http://travistidwell.com/jsencrypt/demo/index.html
- public key -> app save
- secret key -> server save
- app call server api
- encrypt algorithm: **encrypt(json data + SHA1_keystore)**

## API:

- ***/v2/stripe/checkout***
    - app encrypt request json and submit
    - server decrypt request and parse to JSON object, then call stripe `create_token` function `https://stripe.com/docs/api/node#create_card_token`
    - request:
    ```js
    {
        "number": '4242424242424242',
        "exp_month": 12,
        "exp_year": 2018,
        "cvc": '123'
    }
    ```
    - response:
    ```js
    {
        token: "xyz"
    }
    ```
- ***/v2/stripe/charge***
    - app encrypt request json and submit
    - server decrypt request and parse json object
    - server call retrieve token to check if this token was used, `https://stripe.com/docs/api/node#retrieve_token`
    - after checking token success, server call charge function, `https://stripe.com/docs/api/node#create_charge`
    - request:
    ```js
    { token: "xyx", productId: "abc" }
    ```

