# Maybank2u Pay API Reference

Use this reference for factual Maybank2u Pay Online Banking Payment Gateway guidance. Maybank publishes one SDK documentation page plus downloadable SDKs; everything below is distilled from those and the Maybank2u Pay portal. Facts were checked on 2026-10-21. Anything marked "Not documented" must be confirmed with Maybank (`maybank2upay@maybank.com`, SDK support, replies next business day).

## Source URLs

- SDK docs (single page): `https://m2upay.maybank2u.com.my/assets/sdk/index.html`
- Java SDK: `https://m2upay.maybank2u.com.my/assets/sdk/sdk/Java%20SDK%20v1.0.zip` (contains `M2UPayment.jar`, and inside it `docs/PaymentAPIReference-V1.docx`)
- JavaScript SDK: `https://m2upay.maybank2u.com.my/assets/sdk/sdk/JS%20SDK%20v1.0.zip` (`m2upay_backend/m2upay_backend.js`, `m2upay_frontend/m2upay_frontend.js`, `Sample/`)
- PHP SDK: `https://m2upay.maybank2u.com.my/assets/sdk/sdk/PHP%20SDK%20v1.0.zip` (`m2upay_backend/M2UPay.php`, `m2upay_frontend/m2upay_frontend.js`)
- FAQ: `https://m2upay.maybank2u.com.my/assets/faq/index.html`
- Merchant portal (application, Payee Code, technical requirement): `https://m2upay.maybank2u.com.my`

## What This Gateway Is

- Maybank2u Pay "Online Banking Payment Gateway": the customer pays by logging in to Maybank2u and debiting a Maybank current or savings account, confirming with a TAC. It is Maybank-only, not multi-bank FPX. (FAQ)
- Cards (Visa, Mastercard, Amex) go through a separate product, Maybank eBPG. Its technical spec is not public; see `account-setup.md`.
- Integration model: the merchant builds an encrypted string server-side, the browser posts it into a Maybank pop-up, and Maybank sends a server-to-server Realtime Payment Notification (RPN) to the merchant's Response URL.

## Endpoint Catalogue

Every documented operation (there are no REST APIs beyond these):

| # | Direction | Method | URL / function | Purpose |
|---|---|---|---|---|
| 1 | Merchant server (local SDK call) | n/a | `getEncryptionString(params, envType)` (JS, PHP); `PaymentGateway.getencryptedURL(PGParams, envType)` (Java jar) | Build and encrypt the send string; return `encryptedString` and `actionUrl` |
| 2 | Browser to Maybank | POST (HTML form, `target="M2UPayment"` pop-up) | `actionUrl` for the environment (table below), fields `q`, `i` | Open Maybank2u login and payment pop-up |
| 3 | Maybank to merchant | POST, JSON body | Merchant Response URL and port registered in the portal | Realtime Payment Notification (RPN) |
| 4 | Merchant to Maybank | HTTP response body, JSON | Response to request 3 | Acknowledge the RPN with a status code |
| 5 | Maybank to browser | GET redirect with query string | `callbackUrl` / `Callbackurl` passed in the send string | Return the customer after payment (described only in the Java SDK guide) |

Not documented in official docs (checked 2026-10-21): status query/requery, refund, void, cancel, capture, recurring, tokenisation, payout, settlement report API. Reports are downloaded from the Maybank Payee Report Portal instead.

## Environments

`envType` selects the `actionUrl` returned by the SDK:

| envType | Name | actionUrl (all three SDKs) | RPN sent? |
|---|---|---|---|
| `0` (also the default branch) | Playground | `https://api.discotech.io/v1.0/testM2uPayment` | No |
| `1` | UAT (User Acceptance Test) | `https://202.162.18.55:8443/testM2uPayment` | Yes |
| `2` | Production / Live | `https://www.maybank2u.com.my/mbb/m2u/m9006_enc/m2uMerchantLogin.do` | Yes |

- Playground shows a status pop-up only; production asks the customer to log in to Maybank2u and confirm with TAC. (SDK docs)
- The older Java guide (`PaymentAPIReference-V1.docx`) lists a different UAT URL, `https://m2umobilesit.maybank.com.my/mbb/m2u/m9006_enc/m2uMerchantLogin.do?`. The SDK code uses the IP URL above. Confirm the current UAT URL with Maybank.
- Payee Code is per environment: a UAT (test) Payee Code is shown in the portal after approval; the production Payee Code is issued later. Replace both the Payee Code and `envType` at go-live. (SDK docs, FAQ, portal)
- Note: the JavaScript SDK switches on a number (`switch (envType)` with `case 1`, `case 2`), so passing the string `"2"` silently falls to the playground URL. Pass an integer.

## Auth Model

- There are no API keys, merchant secrets or signatures. Identity is the Payee Code inside the encrypted send string.
- The encryption key and salt are hardcoded and identical in every public SDK, so the encrypted string is obfuscation only. Anyone can decrypt, change and re-encrypt it. Integrity must come from the merchant's own checks on the RPN (amount and reference match). Maybank's Java guide: "Payee must match the response values with the request values sent to Maybank2u, particularly for amount and bill account and/or reference number. Please note this is mandatory step to prevent data manipulation by dishonest buyer."

## Step 1: Build The Request (getEncryptionString)

Input parameters (SDK docs table, plus SDK source):

| Field | Type | Length | Required | Notes |
|---|---|---|---|---|
| `amount` | Decimal | 9 | Mandatory | Ringgit with decimals, e.g. `100.00` for RM100.00. Note: the RPN returns cents. |
| `accountNumber` | Alphanumeric | 30 | Mandatory | Your purchase ref / invoice / bill number. Returned in the RPN as `AcctId`. Use it to match the order. |
| `payeeCode` | Numeric (docs) | 3 | Mandatory | Assigned by Maybank per environment. Note: the docs' own encrypted sample decrypts to a 5-character alphanumeric payee code, so treat the type/length as indicative and copy exactly what the portal shows. |
| `envType` | Numeric | 1 | Mandatory | `0` playground, `1` UAT, `2` production. Separate function argument. |
| `refNumber` (`refNum` in Java) | String | Not documented | Optional (SDK only) | Read by all three SDKs; not in the web docs table. The Java guide calls it "Reference number for the transaction". |
| `callbackUrl` (`Callbackurl` in Java) | URL | Not documented | Optional (SDK only) | "Redirection URL after the payment success/failure" (Java guide). Not in the web docs table. |

Send string format, copied from `M2UPay.php` and `m2upay_backend.js` (Java has the same `$1$`, `$$$1$`, `$$$` fragments):

```text
accountNumber empty, refNumber set:   Login$<payeeCode>$1$<amount>$1$<refNumber>$$$<callbackUrl>
accountNumber set, refNumber empty:   Login$<payeeCode>$1$<amount>$$$1$<accountNumber>$<callbackUrl>
otherwise (both set):                 Login$<payeeCode>$1$<amount>$1$<refNumber>$1$<accountNumber>$<callbackUrl>
```

The SDK docs' sample `encryptedString` decrypts (with the public SDK key) to the second form: `Login$<payee>$1$60$$$1$12345678$http://localhost/m2uPayPopupCallback`. This confirms the format and the algorithm below.

Gotchas from the SDK source:

- If `callbackUrl` is not passed, the JS SDK concatenates the literal text `undefined` and PHP appends an empty string (with a notice). The web docs samples do not pass it. Pass your return URL, or an empty string, explicitly. Whether Maybank requires it is not documented.
- `$` is the delimiter. Keep `$` out of `accountNumber`, `refNumber` and the callback URL (inference from the format).
- The JS SDK returns the string `"FAIL"` on exception; the Java guide says the same. PHP returns a JSON string (`json_encode`), not an array; the docs echo it straight into a `<script>` block.
- Java method name in the jar is `getencryptedURL(PGParams, int)`, returning `HashMap<String,String>` with keys `encryptedString` and `actionUrl`. The web docs call it `getEncryptionString`, which does not exist in the jar.

## Encryption Algorithm (exact)

From `M2UPay.php`, `m2upay_backend.js` and the constant pool of `PaymentGateway.class`:

- Cipher: AES-128, ECB mode, PKCS5/PKCS7 padding, no IV.
- Key: 16 bytes `0x00 0x01 0x02 ... 0x0f` (hex `000102030405060708090a0b0c0d0e0f`). Published in every SDK; not a secret.
- Salt (prefix): `Maybank2u simple encryption`
- Iterations: 2. Each round encrypts `salt + previous_value` and Base64-encodes the ciphertext.
- Final step: URL-encode the Base64 (`urlencode` PHP, `encodeURIComponent` JS, `URLEncoder.encode` Java). This URL-encoded value is what goes into form field `q`.

```js
// Node.js equivalent of m2upay_backend.js (verified against the docs' sample output prefix)
const crypto = require('crypto');
const KEY = Buffer.from('000102030405060708090a0b0c0d0e0f', 'hex');
const SALT = 'Maybank2u simple encryption';
function m2uEncrypt(sendString) {
  let v = sendString;
  for (let i = 0; i < 2; i++) {
    const c = crypto.createCipheriv('aes-128-ecb', KEY, null); // PKCS7 padding by default
    v = Buffer.concat([c.update(SALT + v, 'utf8'), c.final()]).toString('base64');
  }
  return encodeURIComponent(v);
}
```

```php
// PHP 7.2+ replacement for the SDK's mcrypt code (same output)
function m2u_encrypt(string $send): string {
    $key = hex2bin('000102030405060708090a0b0c0d0e0f');
    $v = $send;
    for ($i = 0; $i < 2; $i++) {
        $v = openssl_encrypt('Maybank2u simple encryption' . $v, 'aes-128-ecb', $key); // base64, PKCS7
    }
    return urlencode($v);
}
```

Test vector (computed 2026-10-21 with the Node code above; placeholder payee code):

```text
input:  Login$123$1$100.00$$$1$A123456$https://example.com/return
output: uGJ6%2Bl4prlW5E%2BdLg%2FOaS%2BNAyl0xTJmTiSTUFbeyIUjjErW3eCZ7lSRB3dpQULlUl3YRbfCVHn%2Bem%2FDZowvTdg7YCMaH8Bz4AFQGTLsbMIhbCe7z%2FyNS%2BGPp%2BRYEDnkDm4PSmxA8K5WedIA1k259%2Fpo2VCP%2F03Vra65UOUCPn9ZabmdhhrSu41iYFMi3lun%2FCbhlPzFRJuVwwCljQq4acg%3D%3D
```

ECB is deterministic and both strings start with the same salt and `Login$`, so this output begins with the same characters as the docs' sample `encryptedString`. Decrypting the docs' full sample with this algorithm returns a valid send string. That is how the algorithm was confirmed. ECB is weak cryptography, but it is what Maybank's endpoint expects; do not "upgrade" it.

SDK portability notes:

- PHP SDK uses `mcrypt_*`, removed in PHP 7.2. Use the `openssl_encrypt` version above.
- Java SDK uses `sun.misc.BASE64Encoder` (seen in the class constant pool), which inserts a line break every 76 characters, while PHP and JS do not. Prefer the PHP/JS output and verify in UAT that the pop-up opens.
- JS SDK depends on `crypto-js` `^3.1.6`.

## Step 2: Front End Pop-up (m2upay.initPayment)

SDK docs and `m2upay_frontend.js` (same logic in all three SDKs):

1. Include `m2upay_frontend.js` and add `<div id="m2upay"></div>` to the payment page.
2. Call `m2upay.initPayment(encryptedString, actionUrl, 'OT')`.
3. The function injects a form `name="M2UPayment" method="post" target="M2UPayment"` with hidden inputs `q` = `encryptedString` and `i` = third argument, sets `action = actionUrl`, opens `window.open('about:self', 'M2UPayment', 'width=400,height=350')`, submits, then clears the div.

| Form field | Value | Notes |
|---|---|---|
| `q` | URL-encoded encrypted send string | As returned by step 1, unchanged |
| `i` | `OT` | Docs and all samples use `OT`. The Java guide says the transaction type may be `OT`, `DPE` or `OPR` ("Consult with Virtual Banking"); meanings not documented |

- If the pop-up shows an error, repeat step 1 and try again. (SDK docs)
- The JS sample calls `initPayment` inside an AJAX success callback. Browsers often block `window.open` outside a direct click. Pre-fetch the encrypted string, or open the pop-up synchronously in the click handler.
- FAQ: customers should disable pop-up blockers.
- You can reimplement the form yourself (same fields, method, target) instead of loading the SDK file.

## Step 3: Realtime Payment Notification (RPN)

- Server-to-server HTTP POST from Maybank to the Response URL (and port) you registered in the portal's Payment Notification Technical Requirement, after the payment succeeds or fails. (SDK docs, portal)
- Message format: JSON. Sent for all transactions, status `00` success or `01` fail. (Portal)
- UAT and production only; not available in the playground.
- Whitelist Maybank IPs if you have a firewall: UAT `202.162.18.247`, production `202.162.17.179`.
- The Response URL must be HTTPS with an SSL certificate you upload to the portal (`.cer`/`.crt` or zip; CA examples given: Comodo or Verisign). "Payment notification will fail to reach you if you provide an incorrect SSL Certificate." It is "not a redirect URL". (Portal)

Sample body (SDK docs):

```json
{
  "Msg": {
    "PmtType": "CashPayment",
    "RefId": "M2U112211255",
    "TrnDateTime": "20161109105609",
    "AcctId": "A123456",
    "Amt": "10000",
    "StatusCode": "00",
    "StatusDesc": "Successful"
  }
}
```

| Field | Type | Length | Notes |
|---|---|---|---|
| `AcctId` | Alphanumeric | - | Echo of your `accountNumber`. Match to the order. |
| `Amt` | Numeric | - | Amount in cents (sen). RM100.00 is `10000`. Sample shows it as a JSON string. |
| `PmtType` | Alpha | - | `CashPayment` or `CCardPayment` |
| `RefId` | Alphanumeric | - | Unique bank transaction ID, e.g. `M2UT3695990209302989`. Use as the idempotency key. |
| `StatusCode` | Numeric | 2 | `00` successful, `01` unsuccessful |
| `StatusDesc` | Alpha | - | Text status, e.g. `Successful` |
| `TrnDateTime` | Numeric | - | `YYYYMMDDhhmmss`, e.g. `20161109105609`. Time zone not documented (Malaysia time assumed). |

Verification: the RPN has no signature, hash, HMAC or checksum field (Not documented in official docs, checked 2026-10-21). Defences available: TLS to your registered URL, source IP allowlist, and strict matching of `AcctId` + `Amt` against a pending local order. Content-Type header not documented; parse the raw body as JSON and tolerate `application/json` or form encoding by logging the raw body first.

## Step 4: Acknowledge The RPN

Return this JSON in the HTTP response, echoing `RefId` and `PmtType` from the notification:

```json
{ "Msg": { "PmtType": "CashPayment", "RefId": "M2U112211255", "StatusCode": "0" } }
```

| StatusCode | Description | Meaning |
|---|---|---|
| `-1` | Internal error | Error at payee; asks Maybank to resend |
| `0` | OK | Successful |
| `101` | Unsuccessful | Transaction rejected by merchant |
| `104` | Duplicate transaction | Merchant has already been notified |

Codes `998` and `999` are reserved for bank use. Retry count, retry interval and timeout are not documented.

Handler pseudo-code:

```text
if remote_ip not in allowed_ips[env]: return 403
raw = read_body(); log(raw); msg = json(raw).Msg
order = find_pending_or_paid(order_ref = msg.AcctId)
if not order: reply(msg, "101")
if exists_payment(ref_id = msg.RefId): reply(msg, "104")
if int(msg.Amt) != order.amount_sen: reply(msg, "101")      # flag for review
in one DB transaction:
    insert payment(ref_id unique = msg.RefId, status = msg.StatusCode, raw)
    if msg.StatusCode == "00": mark order paid
    else: mark order failed
on DB error: reply(msg, "-1")
reply(msg, "0")
```

## Step 5: Browser Return (Callbackurl)

Only the Java guide (`PaymentAPIReference-V1.docx`) describes it: "On successful login/transaction in M2u, Callbackurl will be provided by M2U to handle transaction status." Sample:

```text
http://localhost:1234/m2uPayPopupCallback?transDate=2016-09-16T18:10:36.195+08:00&status=00&transAmount=1000&referenceNo=ABC1234&payeeCode=ABC&approvalCode=&bankRefNo=1111133868&accountNo&corpName=XYZ&accountnumber=12345612
```

| Param | Notes |
|---|---|
| `transDate` | ISO 8601 with `+08:00` |
| `status` | `00` success, `01` failure |
| `transAmount` | Amount without decimal places (RM1.25 = `125`) |
| `referenceNo` | Your reference |
| `payeeCode` | Your Payee Code |
| `approvalCode` | May be empty |
| `bankRefNo` | Bank reference |
| `accountNo`, `accountnumber` | Account / bill fields (both appear in the sample) |
| `corpName` | Merchant name |

The redirect is unsigned and arrives inside the pop-up. Use it to show a "checking payment" page and wait for the RPN. Never settle from it.

## Amount Rules

- Request `amount`: ringgit decimal string, e.g. `100.00` (max length 9).
- RPN `Amt` and redirect `transAmount`: integer cents.
- Store the expected amount in sen and compare integers. Currency is MYR only (service is Malaysia-only, FAQ). No currency field exists.

## Status Values

| Where | Field | Values |
|---|---|---|
| RPN | `StatusCode` | `00` successful, `01` unsuccessful |
| Redirect | `status` | `00` success, `01` failure |
| Merchant reply | `StatusCode` | `-1`, `0`, `101`, `104` (bank reserved `998`, `999`) |

## Testing

- Playground (`envType` 0): pop-up shows status, no RPN, Payee Code from the portal.
- UAT (`envType` 1): UAT Payee Code, real RPN to your UAT Response URL from `202.162.18.247`.
- UAT sign-off in the portal requires uploading a screenshot of the payment pop-up and a screenshot of the RPN received, then confirming UAT passed with no open defects. (Portal)
- No public test bank accounts or credentials are published.

## Limits And Ops

- Rate limits: not documented.
- Pop-up size 400x350 (SDK front end).
- Response URL changes take 5 working days and delay testing and go-live. (Portal)
- Reporting: "Daily Payment Summary Report" and "Payment Raw Data File" in the Maybank Payee Report Portal (`https://payee.maybank2u.com.my/payee/LogIn.do`, linked from the portal); credentials are emailed after approval.

## Common Errors

| Symptom | Likely cause |
|---|---|
| Pop-up never opens | Blocked by browser because `initPayment` ran outside a click; missing `<div id="m2upay">` |
| Pop-up shows error page | Bad encryption output (Java line wrap, double URL encoding, wrong key), wrong Payee Code for the environment, or wrong `actionUrl`. Rebuild and retry. |
| Production goes to playground | `envType` passed as a string to the JS SDK |
| No RPN received | Playground environment, firewall not allowing Maybank IP, SSL certificate not matching the one uploaded, wrong Response URL/port, or URL changed less than 5 working days ago |
| Amount mismatch | Comparing ringgit request amount to cents `Amt` |
| PHP fatal error | `mcrypt` not available on PHP 7.2+ |
| Callback URL ends with `undefined` | JS SDK called without `callbackUrl` |
