# Bayarcash API Documentation Repo Snapshot

Source: https://github.com/webimpian/bayarcash-api-documentation at `059d8c9e5a3075e802ca73cab28d99dc25d9e396`.

This file aggregates Markdown docs from the repository for offline skill use.

## Contents

- `README.md`
- `SUMMARY.md`
- `bank-list/fpx.md`
- `bank-list/duitnow-online-banking-wallets.md`
- `portal/list.md`
- `portal/create.md`
- `payment/payment-channel.md`
- `payment/payment-intent.md`
- `payment/payment-intent-id.md`
- `transaction/callback.md`
- `transaction/transaction-id.md`
- `transaction/all-transactions.md`
- `checksum/checksum-validation.md`
- `checksum/payment-intent.md`
- `checksum/transaction-callback.md`
- `direct-debit/e-mandate-enrollment.md`
- `direct-debit/e-mandate-maintenance.md`
- `direct-debit/e-mandate-termination.md`
- `direct-debit/activate-e-mandate.md`
- `direct-debit/deactivate-e-mandate.md`
- `direct-debit/callback.md`
- `direct-debit/mandate-id.md`
- `direct-debit/mandate-transaction-id.md`
- `enterprise-partner/merchant-registration.md`
- `enterprise-partner/payout-bank-list.md`
- `enterprise-partner/merchant-status.md`
- `enterprise-partner/merchant-payout-id.md`
- `enterprise-partner/all-merchant-payouts.md`


---

## File: `README.md`

# Welcome

***



Welcome to the Bayarcash API documentation site. Here you'll find comprehensive guides to help you start working with our API endpoint as quickly as possible.

Let's jump right in!



***

### Sandbox & Production

For testing & integration purpose, you might use our sandbox environment at:

* [https://console.bayarcash-sandbox.com](https://console.bayarcash-sandbox.com)
* API v2 - `console.bayarcash-sandbox.com/api/v2`
* API v3 - `api.console.bayarcash-sandbox.com/v3`

Our production URL are:

* [https://console.bayar.cash](https://console.bayar.cash)
* API v2 - `console.bayar.cash/api/v2`
* API v3 - `api.console.bayar.cash/v3`



***

### Plugin

For WordPress user, we have prepared few official plugin which can be downloaded directly from [our WordPress.org repository account](https://profiles.wordpress.org/webimpian/#content-plugins).

List of all plugin & integration, please refer here [plugin.bayarcash.com](https://plugin.bayarcash.com/). Demo usage of Bayarcash can also be viewed here [bayarcash-demo.com](https://bayarcash-demo.com/).



***

### Software Development Kit (SDK)

You might also interested with our official SDK and demo usage of Bayarcash API endpoint.

* [Bayarcash PHP SDK](https://github.com/webimpian/bayarcash-php-sdk)
* [Demo page using vanilla PHP](https://github.com/webimpian/bayarcash-php-demo)


---

## File: `SUMMARY.md`

# Table of contents

* [Welcome](README.md)

## Bank List

* [FPX](bank-list/fpx.md)
* [DuitNow Online Banking/Wallets](bank-list/duitnow-online-banking-wallets.md)

## Portal

* [Portal List](portal/list.md)
* [Create Portal](portal/create.md)

## Payment

* [Payment Channel](payment/payment-channel.md)
* [Payment Intent](payment/payment-intent.md)
* [Payment Intent ID](payment/payment-intent-id.md)

## Transaction

* [Callback](transaction/callback.md)
* [Transaction ID](transaction/transaction-id.md)
* [All Transactions](transaction/all-transactions.md)

## Checksum

* [Checksum Validation](checksum/checksum-validation.md)
* [Payment Intent](checksum/payment-intent.md)
* [Transaction Callback](checksum/transaction-callback.md)

## Direct Debit

* [e-Mandate Enrollment](direct-debit/e-mandate-enrollment.md)
* [e-Mandate Maintenance](direct-debit/e-mandate-maintenance.md)
* [e-Mandate Termination](direct-debit/e-mandate-termination.md)
* [Activate e-Mandate](direct-debit/activate-e-mandate.md)
* [Deactivate e-Mandate](direct-debit/deactivate-e-mandate.md)
* [Callback](direct-debit/callback.md)
* [Mandate ID](direct-debit/mandate-id.md)
* [Mandate Transaction ID](direct-debit/mandate-transaction-id.md)

## Enterprise Partner

* [Merchant Registration](enterprise-partner/merchant-registration.md)
* [Payout Bank List](enterprise-partner/payout-bank-list.md)
* [Merchant Status](enterprise-partner/merchant-status.md)
* [Merchant Payout ID](enterprise-partner/merchant-payout-id.md)
* [All Merchant Payouts](enterprise-partner/all-merchant-payouts.md)

## Sitelink

* [GitHub @webimpian](https://github.com/webimpian)


---

## File: `bank-list/fpx.md`

# FPX

***

<mark style="color:red;">v2</mark>  <mark style="color:blue;">`GET`</mark>  `console.bayar.cash/api/v2/banks`\
<mark style="color:red;">v3</mark> <mark style="color:blue;">`GET`</mark>  `api.console.bayar.cash/v3/banks`

***



This API will return list of bank codes for FPX (B2C) and Direct Debit payment channel. Use this list for usage of `payer_bank_code` and `payer_bank_name` in payment request.

Example of sending <mark style="color:blue;">`GET`</mark> request with cURL.



```markup
curl -X GET https://api.console.bayar.cash/v3/banks \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer <Personal_Access_Token>'
```



Example of JSON structured response.



```json
[
    {
        "bank_display_name": "Affin Bank",
        "bank_name": "Affin Bank Berhad",
        "bank_code": "ABB0233",
        "bank_code_hashed": "472e729c398dbc372d96f5d852393b76",
        "bank_availability": true
    },
    {
        "bank_display_name": "AGRONet",
        "bank_name": "Bank Pertanian Malaysia Berhad (AgroBank)",
        "bank_code": "AGRO01",
        "bank_code_hashed": "6f692c003561e3c52f0d993805b6be01",
        "bank_availability": true
    }
]
```


---

## File: `bank-list/duitnow-online-banking-wallets.md`

# DuitNow Online Banking/Wallets

***

<mark style="color:red;">v2</mark>  <mark style="color:blue;">`GET`</mark>  `console.bayar.cash/api/v2/duitnow/dobw/banks`\
<mark style="color:red;">v3</mark>  <mark style="color:blue;">`GET`</mark>  `api.console.bayar.cash/v3/duitnow/dobw/banks`

***



This API will return list of bank codes for DuitNow Online Banking/Wallets (also known as DOBW or DuitNow OBW) payment channel. Use this list for usage of `payer_bank_code` and `payer_bank_name` in payment request.

Example of sending <mark style="color:blue;">`GET`</mark> request with cURL.



```markup
curl -X GET https://api.console.bayar.cash/v3/duitnow/dobw/banks \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer <Personal_Access_Token>'
```



Example of JSON structured response.



```json
[
    {
        "bank_name": "Affin Bank",
        "bank_code": "PHBMMYKL",
        "bank_availability": true
    },
    {
        "bank_name": "Agro Bank",
        "bank_code": "AGRO01",
        "bank_availability": true
    }
]
```


---

## File: `portal/list.md`

# Portal List

***

<mark style="color:red;">v2</mark>  <mark style="color:blue;">`GET`</mark>  `console.bayar.cash/api/v2/portals`\
<mark style="color:red;">v3</mark>  <mark style="color:blue;">`GET`</mark>  `api.console.bayar.cash/v3/portals`

***



Retrieve list of portal in Bayarcash. Example of sending <mark style="color:blue;">`GET`</mark> request with cURL.



```markup
curl -X GET https://api.console.bayar.cash/v3/portals \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer <Personal_Access_Token>'
```



Example of JSON structured response.



```json
{
    "data": [
        {
            "id": 931,
            "created_at": "2024-10-19 12:47:07",
            "portal_key": "8baf5e234dd88c2d1375d5f386d78d8f",
            "portal_name": "Payment Link",
            "url": "https://bcl.my/",
            "transaction_notification_email": "hai@bayarcash.com",
            "custom_payment_button_text": null,
            "payment_channels": [
                {
                    "id": 1,
                    "code": "FPX",
                    "name": "FPX"
                },
                {
                    "id": 3,
                    "code": "FpxDirectDebit",
                    "name": "Direct Debit"
                },
                {
                    "id": 4,
                    "code": "FpxLineOfCredit",
                    "name": "FPX Line of Credit"
                }
            ]
        }
    ],
    "links": {
        "first": "https://api.console.bayar.cash/v3/portals?page=1",
        "last": "https://api.console.bayar.cash/v3/portals?page=1",
        "prev": null,
        "next": null
    },
    "meta": {
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "links": [
            {
                "url": null,
                "label": "pagination.previous",
                "active": false
            },
            {
                "url": "https://api.console.bayar.cash/v3/portals?page=1",
                "label": "1",
                "active": true
            },
            {
                "url": null,
                "label": "pagination.next",
                "active": false
            }
        ],
        "path": "https://api.console.bayar.cash/v3/portals",
        "per_page": 15,
        "to": 2,
        "total": 2,
        "merchant": {
            "name": "Bayarcash Sdn. Bhd.",
            "email": "hai@bayarcash.com",
            "created_at": "2024-10-19 12:45:42"
        }
    }
}
```


---

## File: `portal/create.md`

# Create Portal

***

<mark style="color:red;">v2</mark>  <mark style="color:green;">`POST`</mark>  `console.bayar.cash/api/v2/portals`\
<mark style="color:red;">v3</mark>  <mark style="color:green;">`POST`</mark>  `api.console.bayar.cash/v3/portals`

***



Create new portal via API. Example of sending <mark style="color:green;">`POST`</mark> request with cURL.


---

## File: `payment/payment-channel.md`

# Payment Channel

***



Make sure you have activate selected payment channel before making any request to our endpoint.



| Code | Description                                                    |
| ---- | -------------------------------------------------------------- |
| 1    | FPX (Default payment channel)                                  |
| 2    | Manual Bank Transfer                                           |
| 3    | Direct Debit (enrollment, maintenance and termination via FPX) |
| 4    | FPX Line of Credit                                             |
| 5    | DuitNow Online Banking/Wallets                                 |
| 6    | DuitNow QR                                                     |
| 7    | SPayLater (BNPL from Shopee)                                   |
| 8    | Boost PayFlex (BNPL from Boost)                                |
| 9    | QRIS Indonesia Online Banking                                  |
| 10   | QRIS Indonesia eWallet                                         |
| 11   | NETS Singapore                                                 |


---

## File: `payment/payment-intent.md`

# Payment Intent

***

<mark style="color:red;">v2</mark>  <mark style="color:green;">`POST`</mark>  `console.bayar.cash/api/v2/payment-intents`\
<mark style="color:red;">v3</mark>  <mark style="color:green;">`POST`</mark>  `api.console.bayar.cash/v3/payment-intents`

***



Initialize payment intent request to Bayarcash. Make sure your account is enabled for selected payment channel. By default only FPX channel is activated.



<table data-full-width="true"><thead><tr><th width="269">Name</th><th width="546">Description</th><th width="121">Type</th><th>Condition</th></tr></thead><tbody><tr><td><code>payment_channel</code></td><td>Refer payment channel page</td><td><code>integer</code></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>portal_key</code></td><td>Portal key retrieve from Bayarcash console</td><td><code>string</code></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>order_number</code></td><td></td><td><code>string</code></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>amount</code></td><td></td><td><code>integer</code></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>payer_name</code></td><td></td><td><code>string</code></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>payer_email</code></td><td></td><td><code>string</code></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>payer_telephone_number</code></td><td>Currently we only accept Malaysia number</td><td><code>integer</code></td><td>Optional</td></tr><tr><td><code>payer_bank_code</code></td><td></td><td><code>string</code></td><td>Optional</td></tr><tr><td><code>payer_bank_name</code></td><td></td><td><code>string</code></td><td>Optional</td></tr><tr><td><code>metadata</code></td><td>Currently only support order items from WooCommerce plugin</td><td><code>string</code></td><td>Optional</td></tr><tr><td><code>return_url</code></td><td>Server to browser redirect callback (use <mark style="color:blue;"><code>GET</code></mark>)</td><td><code>string</code></td><td>Optional</td></tr><tr><td><code>callback_url</code></td><td>Server to server redirect callback (use <mark style="color:green;"><code>POST</code></mark>) - only available on v3</td><td><code>string</code></td><td>Optional</td></tr><tr><td><code>platform_id</code></td><td></td><td><code>string</code></td><td>Optional</td></tr><tr><td><code>checksum</code></td><td></td><td><code>string</code></td><td>Optional</td></tr></tbody></table>

***



Example of sending <mark style="color:green;">`POST`</mark> request with cURL.



```markup
curl -X POST https://api.console.bayar.cash/v3/payment-intents \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer <Personal_Access_Token>' \
  --data-raw '{
        "payment_channel": 5,
        "portal_key": string,
        "order_number": string,
        "amount": 100,
        "payer_name": string,
        "payer_email": string,
        "payer_telephone_number": integer,
        "payer_bank_code": string,
        "payer_bank_name": string,
        "metadata": string,
        "return_url": string,
        "platform_id": string,
        "checksum": string
      }'
```



Example of JSON structured response.



```json
{
    "type": "payment_intent", // only available on v3
    "id": "pi_MGWpzp", // only available on v3
    "payer_name": "Mohd Ali",
    "payer_email": "mohd.ali@gmail.com",
    "payer_telephone_number": "60193000123",
    "order_number": "1351",
    "amount": "21.10",
    "url": "https://console.bayar.cash/payment-intent/pi_MGWpzp"
}
```


---

## File: `payment/payment-intent-id.md`

# Payment Intent ID

***

<mark style="color:red;">v3</mark>  <mark style="color:blue;">`GET`</mark>  `api.console.bayar.cash/v3/payment-intents/{payment_intent_id}`

***



At any given time, you can request a payment intent ID to check on the payment status. It will return the payment intent object.

Example of sending <mark style="color:blue;">`GET`</mark> request with cURL.



```markup
curl -X GET https://api.console.bayar.cash/v3/payment-intents/trx_z88ymJ \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer <Personal_Access_Token>'
```



Example of JSON structured response.



```json
{
    "type": "payment_intent",
    "id": "pi_PGPP2G",
    "status": "paid",
    "last_attempt": "2024-12-30 12:07:57",
    "paid_at": "2024-12-30 12:07:57",
    "order_number": "ORD001",
    "amount": "10.50",
    "currency": "MYR",
    "payer_name": "Mohd Ali",
    "payer_email": "mohd.ali@gmail.com",
    "payer_telephone_number": "+60169166656",
    "attempts": [
        {
            "transaction_id": "trx_YdoXvn",
            "created_at": "2024-12-30 12:07:57",
            "payer_name": "Mohd Ali",
            "payer_email": "mohd.ali@gmail.com",
            "payer_telephone_number": "+60169166656",
            "order_number": "ORD001",
            "currency": "MYR",
            "amount": "10.50",
            "exchange_reference_number": "1-735-272-902-315691",
            "exchange_transaction_id": "2412271215020084",
            "payer_bank_name": "SBI Bank A",
            "status": 3,
            "status_description": "Approved",
            "payment_gateway": {
                "id": 1,
                "name": "FPX",
                "code": "FPX"
            }
        },
        {
            "transaction_id": "trx_GWlpvW",
            "created_at": "2024-12-27 12:15:02",
            "payer_name": "Mohd Ali",
            "payer_email": "mohd.ali@gmail.com",
            "payer_telephone_number": "+60169166656",
            "order_number": "ORD001",
            "currency": "MYR",
            "amount": "10.50",
            "exchange_reference_number": "1-735-272-902-315691",
            "exchange_transaction_id": "2412271215020084",
            "payer_bank_name": "SBI Bank A",
            "status": 2,
            "status_description": "Unsuccessful",
            "payment_gateway": {
                "id": 1,
                "name": "FPX",
                "code": "FPX"
            }
        }
    ]
}
```


---

## File: `transaction/callback.md`

# Callback

***

<mark style="color:red;">v2</mark>  <mark style="color:green;">`POST`</mark>  `{return_url}`\
<mark style="color:red;">v3</mark>  <mark style="color:blue;">`GET`</mark>  `{return_url}`\
<mark style="color:red;">v3</mark>  <mark style="color:green;">`POST`</mark>  `{callback_url}`

***



If your server accept a callback from Bayarcash, make sure the response return `200` status code. Maximum retry from our side is 5 attempts every 300 seconds (5 minutes) from last attempts. Below are the status value & its corresponding label.



| Status Code | Status Description |
| ----------- | ------------------ |
| 0           | New                |
| 1           | Pending            |
| 2           | Failed             |
| 3           | Success            |
| 4           | Cancelled          |



***

### Transaction

Example of JSON structured request.



```json
{
    "record_type": "transaction",
    "transaction_id": string,
    "exchange_reference_number": string,
    "exchange_transaction_id": string,
    "order_number": string,
    "currency": string,
    "amount": string,
    "payer_name": string,
    "payer_email": string,
    "payer_bank_name": string,
    "status": integer,
    "status_description": string,
    "datetime": string,
    "checksum": string
}
```



***

### Transaction Receipt

Example of JSON structured request.



```json
{
    "transaction_id": string,
    "exchange_reference_number": string,
    "exchange_transaction_id": string,
    "order_number": string,
    "currency": string,
    "amount": string,
    "payer_bank_name": string,
    "status": string,
    "status_description": string,
    "checksum": string
}
```


---

## File: `transaction/transaction-id.md`

# Transaction ID

***

<mark style="color:red;">v2</mark>  <mark style="color:blue;">`GET`</mark>  `console.bayar.cash/api/v2/transactions/{transaction_id}`\
<mark style="color:red;">v3</mark>  <mark style="color:blue;">`GET`</mark>  `api.console.bayar.cash/v3/transactions/{transaction_id}`

***



At any given time, you can request a transaction ID to check on the current status. It will return the transaction object.

Example of sending <mark style="color:blue;">`GET`</mark> request with cURL.



```markup
curl -X GET https://api.console.bayar.cash/v3/transactions/trx_z88ymJ \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer <Personal_Access_Token>'
```



Example of JSON structured response.



```json
{
    "id": "trx_q6LJdl",
    "updated_at": "2023-05-26 13:35:01",
    "datetime": "2023-05-26 13:29:06",
    "order_number": "24354620275",
    "payer_name": "Mohd Ali",
    "payer_email": "mohd.ali@gmail.com",
    "payer_telephone_number": "60111157245",
    "currency": "MYR",
    "amount": 50,
    "exchange_reference_number": "1-685-0718-946-53336",
    "exchange_transaction_id": "23052613290110936",
    "payer_bank_name": "Bank Islam",
    "status": 3,
    "status_description": "Approved",
    "return_url": "https://website.net/api/return_url",
    "metadata": null,
    "payout": {
        "reference_number": "P1685011113148329"
    },
    "payment_gateway": {
        "name": "FPX",
        "code": "FPX"
    },
    "portal": "Portal ABC",
    "merchant": {
        "name": "Web Impian Sdn. Bhd.",
        "email": "webimpian.merchant@gmail.com"
    }
}
```


---

## File: `transaction/all-transactions.md`

# All Transactions

***

<mark style="color:red;">v3</mark>  <mark style="color:blue;">`GET`</mark>  `api.console.bayar.cash/v3/transactions?{query_parameter}`

***



At any given time, you can request a transaction to check on the current status. It will return the transaction object. Available query parameters are as below:



| Name                        | Description                                                                                                        |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `order_number`              |                                                                                                                    |
| `status`                    | Please [refer this page](https://api.webimpian.support/bayarcash/transaction/callback) for status code             |
| `payment_channel`           | Please [refer this page](https://api.webimpian.support/bayarcash/payment/payment-channel) for payment channel code |
| `exchange_reference_number` |                                                                                                                    |
| `payer_email`               |                                                                                                                    |

***



Example of sending <mark style="color:blue;">`GET`</mark> request with cURL.



```markup
curl -X GET https://api.console.bayar.cash/v3/transactions?order_number=24354620275 \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer <Personal_Access_Token>'
```



Example of JSON structured response.



```json
{
    "data": [
        "id": "trx_q6LJdl",
        "updated_at": "2023-05-26 13:35:01",
        "datetime": "2023-05-26 13:29:06",
        "order_number": "24354620275",
        "payer_name": "Mohd Ali",
        "payer_email": "mohd.ali@gmail.com",
        "payer_telephone_number": "60111157245",
        "currency": "MYR",
        "amount": 50,
        "exchange_reference_number": "1-685-0718-946-53336",
        "exchange_transaction_id": "23052613290110936",
        "payer_bank_name": "Bank Islam",
        "status": 3,
        "status_description": "Approved",
        "return_url": "https://website.net/api/return_url",
        "metadata": null,
        "payout": {
            "reference_number": "P1685011113148329"
        },
        "payment_gateway": {
            "name": "FPX",
            "code": "FPX"
        },
        "portal": "Portal ABC",
        "merchant": {
            "name": "Web Impian Sdn. Bhd.",
            "email": "webimpian.merchant@gmail.com"
        }
    ],
    "links": {
        "first": "https://api.console.bayar.cash/v3/transactions?page=1",
        "last": "https://api.console.bayar.cash/v3/transactions?page=1",
        "prev": null,
        "next": null
    },
    "meta": {
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "links": [
            {
                "url": null,
                "label": "&laquo; Previous",
                "active": false
            },
            {
                "url": "https://api.console.bayar.cash/v3/transactions?page=1",
                "label": "1",
                "active": true
            },
            {
                "url": null,
                "label": "Next &raquo;",
                "active": false
            }
        ],
        "path": "https://api.console.bayar.cash/v3/transactions",
        "per_page": 15,
        "to": 6,
        "total": 6
    }
}
```


---

## File: `checksum/checksum-validation.md`

# Checksum Validation

***



To ensure a secured request to Bayarcash API, you need to generate a checksum value using your API secret key. This checksum must be included in your request to validate its integrity and authenticity.&#x20;

You can generate your API secret key from Bayarcash portal at profile page.



* Ensure the API secret key is securely stored and not exposed in your client-side code.
* Always verify the checksum in the callback data to ensure data integrity and authenticity,
* The `ksort` function sorts the payload data by key, which is essential for checksum consistency.
* The `implode` function concatenates the sorted payload values using a delimiter (in this case is |), forming the string for the checksum calculation.
* The `hash_hmac` function generates the checksum using the SHA256 algorithm and your secret key.
* **Trim the payload data** before generating the checksum to ensure no empty spaces are included in the calculation.



{% hint style="success" %}
Note: The checksum value and checksum validation are optional, but it is recommended for enhanced security.
{% endhint %}


---

## File: `checksum/payment-intent.md`

# Payment Intent Request

***



Sample code using PHP to validate payment intent checksum.



```php
<?php
    $secretKey = 'xxxxx';  // Your API secret key obtain from Bayarcash portal
    
    $payloadData = [
        "payment_channel"  => 1,
        "order_number"     => "ORD-0060",
        "amount"           => "60.00",
        "payer_name"       => "Mohd Ali",
        "payer_email"      => "mohd.ali@gmail.com"
    ];
    
    ksort($payloadData);  // Sort the payload data by key
    $payloadString = implode('|', $payloadData);  // Concatenate values with '|'
    
    $checksum = hash_hmac('sha256', $stringPayload, $secretKey); // Generate HMAC SHA256 checksum
```


---

## File: `checksum/transaction-callback.md`

# Transaction Callback

***



Sample code using PHP to validate transaction callback checksum.

This example is tailored for the v2 endpoint (`console.bayar.cash/api/v2`) and can be used to validate data received via the `return_url`.

If you’re using the v3 endpoint (`api.console.bayar.cash/v3`), the same logic applies for validating payload data received from the `callback_url`.



```php
<?php
    $secretKey = 'xxxxx';  // Your API secret key obtain from Bayarcash portal
    
    $callbackChecksum = $callbackData['checksum'];
    
    $payloadData = [
        "record_type"               => $callbackData['record_type'],
        "transaction_id"            => $callbackData['transaction_id'],
        "exchange_reference_number" => $callbackData['exchange_reference_number'],
        "exchange_transaction_id"   => $callbackData['exchange_transaction_id'],
        "order_number"              => $callbackData['order_number'],
        "currency"                  => $callbackData['currency'],
        "amount"                    => $callbackData['amount'],
        "payer_name"                => $callbackData['payer_name'],
        "payer_email"               => $callbackData['payer_email'],
        "payer_bank_name"           => $callbackData['payer_bank_name'],
        "status"                    => $callbackData['status'],
        "status_description"        => $callbackData['status_description'],
        "datetime"                  => $callbackData['datetime'],
    ];
    
    ksort($payloadData);  // Sort the payload data by key
    $payloadString = implode('|', $payloadData);  // Concatenate values with '|'
    
    $validResponse = hash_hmac('sha256', $payloadString, $secretKey) === $callbackChecksum;  // Validate checksum
```




For v3 endpoint (`api.console.bayar.cash/v3`), below are example with payload data from `return_url`.


```php
<?php
    $secretKey = 'xxxxx';  // Your API secret key obtain from Bayarcash portal
    
    $callbackChecksum = $callbackData['checksum'];
    
    $payloadData = [
        'transaction_id'            => $callbackData['transaction_id'],
        'exchange_reference_number' => $callbackData['exchange_reference_number'],
        'exchange_transaction_id'   => $callbackData['exchange_transaction_id'],
        'order_number'              => $callbackData['order_number'],
        'currency'                  => $callbackData['currency'],
        'amount'                    => $callbackData['amount'],
        'payer_bank_name'           => $callbackData['payer_bank_name'],
        'status'                    => $callbackData['status'],
        'status_description'        => $callbackData['status_description'],
    ];
    
    ksort($payloadData);  // Sort the payload data by key
    $payloadString = implode('|', $payloadData);  // Concatenate values with '|'
    
    $validResponse = hash_hmac('sha256', $payloadString, $secretKey) === $callbackChecksum;  // Validate checksum
```


---

## File: `direct-debit/e-mandate-enrollment.md`

---
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# e-Mandate Enrollment

***

<mark style="color:red;">v2</mark>  <mark style="color:green;">`POST`</mark>  `console.bayar.cash/api/v2/mandates`\
<mark style="color:red;">v3</mark> <mark style="color:green;">`POST`</mark>  `api.console.bayar.cash/v3/mandates`

***



Initialize enrollment request to Bayarcash. Make sure your account is enabled for Direct Debit payment channel.



***

<table data-full-width="true"><thead><tr><th>Name</th><th>Description</th><th>Type</th><th>Condition</th></tr></thead><tbody><tr><td><code>portal_key</code></td><td>Portal key retrieve from Bayarcash console</td><td><code>string</code></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>order_number</code></td><td></td><td><code>string</code></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>amount</code></td><td></td><td><code>integer</code></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>payer_id_type</code></td><td></td><td><code>integer</code></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>payer_id</code></td><td></td><td><code>integer</code></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>payer_name</code></td><td></td><td><code>string</code></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>payer_email</code></td><td></td><td><code>string</code></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>payer_telephone_number</code></td><td></td><td><code>integer</code></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>frequency_mode</code></td><td></td><td><code>string</code></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>application_reason</code></td><td></td><td><code>string</code></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>effective_date</code></td><td></td><td></td><td><mark style="color:red;">Required</mark></td></tr><tr><td><code>metadata</code></td><td></td><td><code>string</code></td><td>Optional</td></tr><tr><td><code>return_url</code></td><td>Callback data will be sent to this URL</td><td><code>string</code></td><td>Optional</td></tr><tr><td><code>success_url</code></td><td></td><td><code>string</code></td><td>Optional</td></tr><tr><td><code>failed_url</code></td><td></td><td><code>string</code></td><td>Optional</td></tr><tr><td><code>checksum</code></td><td></td><td><code>string</code></td><td>Optional</td></tr></tbody></table>

***



Example of sending <mark style="color:green;">`POST`</mark> request with cURL.



```markup
curl -X POST https://api.console.bayar.cash/v3/mandates \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer <Personal_Access_Token>' \
  --data-raw '{
          "portal_key": string,
          "order_number": DD001,
          "amount": 100,
          "payer_id_type": 1,
          "payer_id": "123456121234",
          "payer_name": "Mohd Ali",
          "payer_email": "mohd.ali@gmail.com",
          "payer_telephone_number": "60191122000",
          "frequency_mode": "MT",
          "application_reason": "Enrollment for DD001",
          "effective_date": "2024-08-01",
          "expiry_date": "2025-07-01",
          "metadata": string,
          "return_url": "https://website.net/callback",
          "success_url": "https://website.net/success",
          "failed_url": "https://website.net/failed",
          "checksum": string
      }'
```



Example of JSON structured response.



```json
{
    "payer_name": "Mohd Ali",
    "payer_id_type": 1,
    "payer_id": "910109021234",
    "payer_email": "mohd.ali@gmail.com",
    "payer_telephone_number": "60196019001",
    "order_number": "DD001",
    "amount": 30,
    "application_type": "Enrollment",
    "application_reason": "Enrollment of DD001",
    "frequency_mode": "MT",
    "effective_date": "2024-06-15",
    "expiry_date": "2024-08-15",
    "url": "https://console.bayar.cash/payment-intent/pi_pGwZaY"
}
```


---

## File: `direct-debit/e-mandate-maintenance.md`

# e-Mandate Maintenance

***

<mark style="color:red;">v2</mark>  <mark style="color:orange;">`PUT`</mark>  `console.bayar.cash/api/v2/mandates/{mandate_id}`\
<mark style="color:red;">v3</mark>  <mark style="color:orange;">`PUT`</mark>  `api.console.bayar.cash/v3/mandates/{mandate_id}`

***



Example of JSON structured response.



```json
{
    "payer_name": "Mohd Ali",
    "payer_id_type": 1,
    "payer_id": "910109021234",
    "payer_email": "mohd.ali@gmail.com",
    "payer_telephone_number": "60198109001",
    "order_number": "DD001",
    "amount": 30,
    "application_type": "Maintenance",
    "application_reason": "Maintenance of DD001",
    "frequency_mode": "YR",
    "effective_date": "2024-06-15",
    "expiry_date": "2024-08-15",
    "url": "https://console.bayar.cash/payment-intent/pi_pGwZaY"
}
```


---

## File: `direct-debit/e-mandate-termination.md`

# e-Mandate Termination

***

<mark style="color:red;">v2</mark><mark style="color:red;">`DELETE`</mark>  `console.bayar.cash/api/v2/mandates/{mandate_id}`\
<mark style="color:red;">v3</mark><mark style="color:red;">`DELETE`</mark>  `api.console.bayar.cash/v3/mandates/{mandate_id}`

***



Example of JSON structured response.



```json
{
    "payer_name": "Mohd Ali",
    "payer_id_type": 1,
    "payer_id": "910109021234",
    "payer_email": "mohd.ali@gmail.com",
    "payer_telephone_number": "60198109001",
    "order_number": "DD001",
    "amount": 30,
    "application_type": "Termination",
    "application_reason": "Termination of DD001",
    "frequency_mode": "YR",
    "effective_date": "2024-06-15",
    "expiry_date": "2024-08-15",
    "url": "https://console.bayar.cash/payment-intent/pi_pGwZaY"
}
```


---

## File: `direct-debit/activate-e-mandate.md`

# Update Inactive e-Mandate Status to Active

***

<mark style="color:red;">v2</mark> <mark style="color:red;">`PATCH`</mark>  `console.bayar.cash/api/v2/mandates/{mandate_id}/activate`\
<mark style="color:red;">v3</mark> <mark style="color:red;">`PATCH`</mark>  `api.console.bayar.cash/v3/mandates/{mandate_id}/activate`

***


At any given time, you can request to update any `inactive` Direct Debit to `active`.

> Notes: This action will re-activate and include the Direct Debit in the next deduction. This action only allowed for the Direct Debit with status `inactive`.


Example of JSON structured response.



```json
{
  "id": "md_MGWpzp",
  "updated_at": "2024-07-21T15:07:09.000000Z",
  "mandate_reference_number": "E-20230408F00230289",
  "order_number": "1-680-631-498-908819",
  "application_reason": "Enrollment for 1-680-631-498-908819",
  "frequency_mode": "MT",
  "frequency_mode_label": "Monthly",
  "effective_date": "2023-04-05",
  "expiry_date": null,
  "currency": "MYR",
  "amount": 10,
  "payer_name": "MOHD ALI",
  "payer_id": "800810065921",
  "payer_id_type": "1",
  "payer_bank_account_number": "1602119463111",
  "payer_email": "m.ali@gmail.com",
  "payer_telephone_number": "0199991122",
  "status": 3,
  "status_description": "Active",
  "return_url": "http://website.net/donations/mandates/callback",
  "metadata": null,
  "portal": "Portal A",
  "merchant": {
    "name": "Merchant A",
    "email": "ska@gmail.com"
  }
}
```


---

## File: `direct-debit/deactivate-e-mandate.md`

# Update Active e-Mandate Status to Inactive

***

<mark style="color:red;">v2</mark> <mark style="color:red;">`PATCH`</mark>  `console.bayar.cash/api/v2/mandates/{mandate_id}/deactivate`\
<mark style="color:red;">v3</mark> <mark style="color:red;">`PATCH`</mark>  `api.console.bayar.cash/v3/mandates/{mandate_id}/deactivate`

***


At any given time, you can request to update any `active` Direct Debit to `inactive`.

> Notes: This action will only exclude the Direct Debit from the deduction process and doesn't terminate the Direct Debit. Termination must be done by payer. This action also allowed only for the Direct Debit with status `active`.


Example of JSON structured response.



```json
{
  "id": "md_MGWpzp",
  "updated_at": "2024-07-21T15:07:09.000000Z",
  "mandate_reference_number": "E-20230408F00230289",
  "order_number": "1-680-631-498-908819",
  "application_reason": "Enrollment for 1-680-631-498-908819",
  "frequency_mode": "MT",
  "frequency_mode_label": "Monthly",
  "effective_date": "2023-04-05",
  "expiry_date": null,
  "currency": "MYR",
  "amount": 10,
  "payer_name": "MOHD ALI",
  "payer_id": "800810065921",
  "payer_id_type": "1",
  "payer_bank_account_number": "1602119463111",
  "payer_email": "m.ali@gmail.com",
  "payer_telephone_number": "0199991122",
  "status": 10,
  "status_description": "Inactive",
  "return_url": "http://website.net/donations/mandates/callback",
  "metadata": null,
  "portal": "Portal A",
  "merchant": {
    "name": "Merchant A",
    "email": "ska@gmail.com"
  }
}
```


---

## File: `direct-debit/callback.md`

# Callback

***

<mark style="color:red;">v2</mark>  <mark style="color:green;">`POST`</mark>  `{return_url}` \
<mark style="color:red;">v3</mark> <mark style="color:green;">`POST`</mark>  `{return_url}`

***



If your server accept a callback from Bayarcash, make sure the response return `200` status code. Maximum retry from our side is 5 attempts every 300 seconds (5 minutes) from last attemps. Below are the status value & its corresponding label.



***

### Authorization

Example of JSON structured request. Applicable to enrollment, maintenance and termination.



```json
{
    "record_type": "authorization",
    "transaction_id": string,
    "mandate_id": string,
    "exchange_reference_number": string,
    "exchange_transaction_id": string,
    "order_number": 100,
    "currency": string,
    "amount": string,
    "payer_name": string,
    "payer_email": string,
    "payer_bank_name": string,
    "status": string,
    "status_description": string,
    "datetime": string,
    "checksum": string
}
```



***

### Bank Approval

Example of JSON structured request. Applicable to enrollment, maintenance and termination.



```json
{
    "record_type": "bank_approval",
    "approval_date": string,
    "approval_status": string,
    "mandate_id": string,
    "mandate_reference_number": string,
    "order_number": string,
    "payer_bank_code_hashed": string,
    "payer_bank_code": string,
    "payer_bank_account_no": string,
    "application_type": 01,
    "checksum": string
}
```



***

### Transaction

Example of JSON structured request. Only applicable to enrollment.



```json
{
    "record_type": "transaction",
    "batch_number": string,
    "mandate_id": string,
    "mandate_reference_number": string,
    "transaction_id": string,
    "datetime": string,
    "reference_number": string,
    "amount": 100,
    "status": string,
    "status_description": string,
    "cycle": 1,
    "checksum": string
}
```


---

## File: `direct-debit/mandate-id.md`

# Mandate ID

***

<mark style="color:red;">v2</mark>  <mark style="color:blue;">`GET`</mark>  `console.bayar.cash/api/v2/mandates/{mandate_id}`\
<mark style="color:red;">v3</mark>  <mark style="color:blue;">`GET`</mark>  `api.console.bayar.cash/v3/mandates/{mandate_id}`

***



Example of JSON structured response.



```json
{
    "id": "md_MGWpzp",
    "updated_at": "2024-07-21T15:07:09.000000Z",
    "mandate_reference_number": "E-20230408F00230289",
    "order_number": "1-680-631-498-908819",
    "application_reason": "Enrollment for 1-680-631-498-908819",
    "frequency_mode": "MT",
    "frequency_mode_label": "Monthly",
    "effective_date": "2023-04-05",
    "expiry_date": null,
    "currency": "MYR",
    "amount": 10,
    "payer_name": "Mohd Ali",
    "payer_id": "910810065921",
    "payer_id_type": "1",
    "payer_bank_account_number": "1602219363111",
    "payer_email": "mohd.ali@gmail.com",
    "payer_telephone_number": "60199971822",
    "status": 3,
    "status_description": "Active",
    "return_url": "http://website.net/transaction/mandates/callback",
    "metadata": null,
    "portal": "Portal ABC",
    "merchant": {
        "name": "Web Impian Sdn. Bhd.",
        "email": "webimpian.merchant@gmail.com"
    }
}
```


---

## File: `direct-debit/mandate-transaction-id.md`

# Mandate Transaction ID

***

<mark style="color:red;">v2</mark>  <mark style="color:blue;">`GET`</mark>  `console.bayar.cash/api/v2/mandates/transactions/{transaction_id}`\
<mark style="color:red;">v3</mark> <mark style="color:blue;">`GET`</mark>  `api.console.bayar.cash/v3/mandates/transactions/{transaction_id}`

***



Example of JSON structured response.



```json
{
    "id": "trx_GPk868",
    "updated_at": "2024-06-27 17:57:01",
    "datetime": "2024-06-27 17:56:42",
    "order_number": "DDD-24060323",
    "payer_name": "Mohd Ali",
    "payer_email": "mohd.ali@gmail.com",
    "payer_telephone_number": "60197019001",
    "currency": "MYR",
    "amount": 10,
    "exchange_reference_number": "1-719-482-202-565809",
    "exchange_transaction_id": "2406271756420574",
    "payer_bank_name": "SBI Bank A",
    "status": 3,
    "status_description": "Approved",
    "return_url": "https://website.net/transactions/mandates/callback",
    "metadata": null,
    "payout": {
        "ref_no": null
    },
    "payment_gateway": {
        "name": "Direct Debit",
        "code": "FpxDirectDebit"
    },
    "portal": "Portal ABC",
    "merchant": {
        "name": "Web Impian Sdn. Bhd.",
        "email": "webimpian.merchant@gmail.com"
    },
    "mandate": {
        "id": "md_nq5oAG",
        "updated_at": "2024-06-27T09:57:27.000000Z",
        "mandate_reference_number": "E-20241719482236",
        "order_number": "DDD-24060323",
        "application_reason": "Enrollment of DDD-24060323",
        "frequency_mode": "MT",
        "frequency_mode_label": "Monthly",
        "effective_date": "2024-06-30",
        "expiry_date": "2025-06-30",
        "currency": "MYR",
        "amount": 200,
        "payer_name": "Mohd Ali",
        "payer_id": "910909122211",
        "payer_id_type": "1",
        "payer_bank_account_number": "033112228108",
        "payer_email": "mohd.ali@gmail.com",
        "payer_telephone_number": "60197019001",
        "status": 4,
        "status_description": "Terminated",
        "return_url": "https://website.net/transactions/mandates/callback",
        "metadata": null,
        "portal": "Portal ABC",
        "merchant": {
            "name": "Web Impian Sdn. Bhd.",
            "email": "webimpian.merchant@gmail.com"
        }
    }
}
```


---

## File: `enterprise-partner/merchant-registration.md`

# Merchant Registration

***

<mark style="color:red;">v2</mark>  <mark style="color:green;">`POST`</mark>  `console.bayar.cash/api/v2/merchant-registrations`\
<mark style="color:red;">v3</mark> <mark style="color:green;">`POST`</mark>  `api.console.bayar.cash/v3/merchant-registrations`

***



Please note this section only for **Bayarcash Enterprise Partner**. Please refer to your account manager for further details.

Example of sending <mark style="color:green;">`POST`</mark> request with cURL.



```markup
curl -X POST https://api.console.bayar.cash/v3/merchant-registrations \
  --header 'Content-Type: application/json' \
  --header 'X-API-Key: <Enterprise_Partner_API_Key>' \
  --header 'Authorization: Bearer <Personal_Access_Token>' \
  --data-raw '{
        "organisation_name": string,
        "organisation_registration_number": string,
        "organisation_bank_id": integer, // Please refer Payout Bank List
        "organisation_bank_account_holder": string,
        "organisation_bank_account_number": string,
        "login_name": string,
        "login_email": string,
        "login_mobile_number": integer,
        "password": string,
        "password_confirmation": string
      }'
```



Example of JSON structured response.



```json
{
    "success": true,
    "message": "Merchant registration success.",
    "data": {
        "id": "mr_3qjZwq",
        "name": "Mohd Ali Bin Seman",
        "email": "namasyarikat@gmail.com",
        "referral_id": null,
        "registration_status": "Incomplete",
        "login_token": "$778374mifnminvrunvrvmrmri997uIO",
        "user_profiles": {
            "organisation_name": "Nama Syarikat Sdn. Bhd.",
            "organisation_registration_number": "123456789-H",
            "contact_first_name": null,
            "contact_last_name": null,
            "contact_mobile_number": "60169165576",
            "contact_email": "namasyarikat@gmail.com",
            "organisation_bank_id": 1,
            "organisation_bank_name": "Affin Bank Berhad",
            "organisation_bank_account_number": "221233898832",
            "organisation_bank_account_holder": "Nama Syarikat Sdn. Bhd." 
        },
        "active_package_subscription": {
            "reference_number": "112234434TE",
            "package": {
                "name": "DuitNow T+ Package",
                "code": "T+"
            },
            "start_date": "2024-11-13",
            "expiry_date": "2025-11-13",
            "minimum_balance": "RM50.00",
            "payouts_schedule": "Payout next working day T+1 starting 01.00 AM"
        },
        "portal": {
            "name": "Default",
            "api_key": "3888499948593892894",
            "merchant_transaction_notification_email": "financesyarikat@gmail.com",
            "url": "https://bcl.my",
            "payment_gateways": [
                {
                    "name": "FPX",
                    "code": "FPX"
                }
            ]
        }
    }
}
```


---

## File: `enterprise-partner/payout-bank-list.md`

# Payout Bank List

***



<table><thead><tr><th width="103">ID</th><th width="440">Bank Name</th><th>Code</th></tr></thead><tbody><tr><td>1</td><td>Affin Bank Berhad</td><td>ABB</td></tr><tr><td>2</td><td>Alliance Bank Malaysia Berhad</td><td>ABMB</td></tr><tr><td>3</td><td>AmBank Malaysia Berhad</td><td>AMB</td></tr><tr><td>4</td><td>Bank Islam Malaysia Berhad</td><td>BIMB</td></tr><tr><td>5</td><td>Bank Muamalat Malaysia Berhad</td><td>BMMB</td></tr><tr><td>6</td><td>Bank Kerjasama Rakyat Malaysia Berhad</td><td>BKRM</td></tr><tr><td>7</td><td>Bank Simpanan Nasional</td><td>BSN</td></tr><tr><td>8</td><td>CIMB Bank Berhad</td><td>CIMB</td></tr><tr><td>9</td><td>Hong Leong Bank Berhad</td><td>HLBB</td></tr><tr><td>10</td><td>HSBC Bank Malaysia Berhad</td><td>HSBC</td></tr><tr><td>11</td><td>Kuwait Finance House (Malaysia) Berhad</td><td>KFH</td></tr><tr><td>12</td><td>Malayan Banking Berhad</td><td>MBB</td></tr><tr><td>13</td><td>OCBC Bank Malaysia Berhad</td><td>OCBC</td></tr><tr><td>14</td><td>Public Bank Berhad</td><td>PBB</td></tr><tr><td>15</td><td>RHB Bank Berhad</td><td>RHB</td></tr><tr><td>16</td><td>Standard Chartered Bank</td><td>SCB</td></tr><tr><td>17</td><td>United Overseas Bank</td><td>UOB</td></tr><tr><td>18</td><td>Bank Pertanian Malaysia Berhad (Agrobank)</td><td>AGRO</td></tr></tbody></table>


---

## File: `enterprise-partner/merchant-status.md`

# Merchant Status

***

<mark style="color:red;">v2</mark>  <mark style="color:blue;">`GET`</mark>  `console.bayar.cash/api/v2/merchant-registrations/{id}`\
<mark style="color:red;">v3</mark> <mark style="color:blue;">`GET`</mark>  `api.console.bayar.cash/v3/merchant-registrations/{id}`

***



As our enterprise partners, you can check for merchant registration status via API. All registration will be approved manually by our account manager. Registration status consist of:



<table><thead><tr><th width="227">Status</th><th>Description</th></tr></thead><tbody><tr><td>Incomplete</td><td>Our system successfully recorded the registration. Merchant can start collect the payment but the payout will be put on hold until merchant complete the submission of needed documents.</td></tr><tr><td>Pending Approval</td><td>Merchant submitted the KYC/KYB documents &#x26; our account manager is reviewing.</td></tr><tr><td>Approved</td><td>Merchant registration is approved &#x26; payout is processing.</td></tr><tr><td>Rejected</td><td>Merchant registration is rejected. Further details, please contact our account manager.</td></tr></tbody></table>



Example of JSON structured response.



```json
{
    "id": "mr_3qjZwq",
    "name": "Mohd Ali Bin Seman",
    "email": "namasyarikat@gmail.com",
    "referral_id": null,
    "registration_status": "Incomplete",
    "login_token": "$778374mifnminvrunvrvmrmri997uIO",
    "user_profiles": {
        "organisation_name": "Nama Syarikat Sdn. Bhd.",
        "organisation_registration_number": "123456789-H",
        "contact_first_name": null,
        "contact_last_name": null,
        "contact_mobile_number": "60169165576",
        "contact_email": "namasyarikat@gmail.com",
        "organisation_bank_id": 1,
        "organisation_bank_name": "Affin Bank Berhad",
        "organisation_bank_account_number": "221233898832",
        "organisation_bank_account_holder": "Nama Syarikat Sdn. Bhd." 
    },
    "active_package_subscription": {
        "reference_number": "112234434TE",
        "package": {
            "name": "DuitNow T+ Package",
            "code": "T+"
        },
        "start_date": "2024-11-13",
        "expiry_date": "2025-11-13",
        "minimum_balance": "RM50.00",
        "payouts_schedule": "Payout next working day T+1 starting 01.00 AM"
    },
    "portal": {
        "name": "Default",
        "api_key": "3888499948593892894",
        "merchant_transaction_notification_email": "financesyarikat@gmail.com",
        "url": "https://bcl.my",
        "payment_gateways": [
            {
                "name": "FPX",
                "code": "FPX"
            }
        ]
    }
}
```


---

## File: `enterprise-partner/merchant-payout-id.md`

# Merchant Payout ID

***

<mark style="color:red;">v2</mark>  <mark style="color:blue;">`GET`</mark>  `console.bayar.cash/api/v2/payouts/{payout_id}`\
<mark style="color:red;">v3</mark> <mark style="color:blue;">`GET`</mark>  `api.console.bayar.cash/v3/payouts/{payout_id}`&#x20;

***



Enterprise partners are able to retrieve settlement summary for all merchants registered from their API key. Example of JSON structured response.



```json
{
    "id": "pay_qK7gOq",
    "created_at": "2025-01-14 10:57:33",
    "merchant": {
        "id": "usr_Zz29gG",
        "name": "ABC Sdn. Bhd.",
        "email": "admin@abc.com"
    },
    "status": "Processing",
    "reference_number": "P1736823453599915",
    "split_payout": "-",
    "bank_transaction_reference_number": null,
    "amount": "1.50"
}
```


---

## File: `enterprise-partner/all-merchant-payouts.md`

# Merchant Settlement Summary

***

<mark style="color:red;">v2</mark>  <mark style="color:blue;">`GET`</mark>  `console.bayar.cash/api/v2/payouts`\
<mark style="color:red;">v3</mark> <mark style="color:blue;">`GET`</mark>  `api.console.bayar.cash/v3/payouts`&#x20;

***



Enterprise partners are able to retrieve settlement summary for all merchants registered from their API key. Example of JSON structured response.



```json
{
    "data": [
        {
            "id": "pay_YZ6jNY",
            "created_at": "2024-11-01 14:20:06",
            "status": "Processing",
            "package_subscription_id": 399,
            "payment_channel": "FPX",
            "reference_number": "P11229399384849",
            "split_payout": "Parent",
            "bank_transaction": null,
            "amount": "97.00"
        }
    ],
    "links": {
        "first": "https://api.console.bayar.cash/v3/payouts?page=1",
        "last": "https://api.console.bayar.cash/v3/payouts?page=2",
        "prev": null,
        "next": "https://api.console.bayar.cash/v3/payouts?page=2"
    },
    "meta": {
        "current_page": 1,
        "from": 1,
        "last_page": 2,
        "links": [
            {
                "url": null,
                "label": "&laquo; Previous",
                "active": false
            },
            {
                "url": "https://api.console.bayar.cash/v3/payouts?page=1",
                "label": "1",
                "active": true
            },
            {
                "url": "https://api.console.bayar.cash/v3/payouts?page=2",
                "label": "2",
                "active": false
            },
            {
                "url": "https://api.console.bayar.cash/v3/payouts?page=2",
                "label": "Next &raquo;",
                "active": false
            }
        ],
        "path": "https://api.console.bayar.cash/v3/payouts",
        "per_page": 15,
        "to": 15,
        "total": 27
    }
}
```
