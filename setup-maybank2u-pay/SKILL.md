---
name: setup-maybank2u-pay
description: Set up, build, debug, review, and explain Maybank2u Pay (M2U Pay) Online Banking Payment Gateway integrations using the official Maybank2u Pay SDK docs and the official Java, JavaScript and PHP SDK source, getEncryptionString send-string building, AES-128-ECB double encryption, envType playground/UAT/production action URLs, m2upay.initPayment pop-up form post (q and i fields), Realtime Payment Notification (RPN) JSON to the Response URL, RPN acknowledgement status codes, Maybank IP allowlisting, Callbackurl redirect parameters, Payee Code, UAT sign-off, and portal technical requirements. Use when working with Maybank2u Pay, Maybank online banking checkout, Maybank current/savings account debit, M2U pop-up payments, payee code setup, RPN webhook settlement, or production readiness.
---

# Maybank2u Pay Payment Gateway

Use this skill for Maybank2u Pay Online Banking Payment Gateway integration work (customer pays from a Maybank current or savings account through a Maybank2u pop-up).

## Source

Read `references/maybank2u-pay-api.md` before giving factual guidance or writing integration code. It holds the endpoint catalogue, send-string format, the exact encryption algorithm, environments, front-end pop-up, Realtime Payment Notification (RPN) fields, the RPN acknowledgement contract, redirect callback parameters, IPs, and gotchas found in the SDK source.

Read `references/account-setup.md` when the user asks how to apply, which documents are needed, where the Payee Code comes from, how UAT sign-off works, Response URL and SSL certificate requirements, reporting portals, settlement, or the Cards Payment Gateway (Maybank eBPG).

Read `references/source-map.md` to pick the right official page or SDK file, and for coverage notes.

## Core Workflow

1. Identify environment: `envType` 0 playground, 1 UAT, 2 production.
2. Confirm the merchant has a Payee Code for that environment and has submitted the Payment Notification Technical Requirement (Response URL, port, SSL certificate) in the Maybank2u Pay portal.
3. Create a local pending payment with a unique reference before building the send string.
4. Server-side, build the send string and encrypt it exactly as the SDK does; return `encryptedString` and `actionUrl` to the page.
5. On the payment page, call `m2upay.initPayment(encryptedString, actionUrl, 'OT')` directly from a user click so the pop-up is not blocked.
6. Treat the pop-up result and any `Callbackurl` redirect as UX only.
7. Settle from the RPN: HTTP POST JSON from Maybank to the Response URL, from Maybank's documented IP, matched on `AcctId` and `Amt` (cents).
8. Reply to every RPN with JSON `{"Msg":{"PmtType":...,"RefId":...,"StatusCode":"0"}}` (or `-1`, `101`, `104`).
9. Settle idempotently on `RefId`; duplicate RPNs return `104`.
10. Reconcile against the Daily Payment Summary Report and Payment Raw Data File in the Maybank Payee Report Portal; there is no public status-query API.

## Core Facts

- Official docs: `https://m2upay.maybank2u.com.my/assets/sdk/index.html` (single page) plus SDK zips for Java, JavaScript and PHP.
- Production action URL: `https://www.maybank2u.com.my/mbb/m2u/m9006_enc/m2uMerchantLogin.do`.
- UAT action URL in all three SDKs: `https://202.162.18.55:8443/testM2uPayment`; playground: `https://api.discotech.io/v1.0/testM2uPayment`.
- Request inputs: `amount` (decimal ringgit, e.g. `100.00`, length 9), `accountNumber` (your invoice/bill ref, alphanumeric, 30), `payeeCode` (numeric, 3), plus SDK-only `refNumber` and `callbackUrl`.
- Send string: `Login$<payeeCode>$1$<amount>$1$<refNumber>$1$<accountNumber>$<callbackUrl>` (empty-field variants in the API reference).
- Encryption: twice, `v = base64(AES-128-ECB-PKCS7(key, "Maybank2u simple encryption" + v))`, key bytes `00 01 02 ... 0f`, then URL-encode. Key and salt are public in every SDK, so this is obfuscation, not integrity.
- Front end posts a form to `actionUrl` into a pop-up named `M2UPayment` with fields `q` (encrypted string) and `i` (sub-service, `OT` in docs and samples).
- RPN: JSON `Msg` with `PmtType` (`CashPayment` or `CCardPayment`), `RefId`, `TrnDateTime` (`YYYYMMDDhhmmss`), `AcctId`, `Amt` (cents), `StatusCode` (`00` success, `01` unsuccessful), `StatusDesc`.
- RPN is UAT and production only; the playground does not send it.
- RPN source IPs: UAT `202.162.18.247`, production `202.162.17.179`.
- RPN reply codes: `-1` internal error (resend), `0` OK, `101` rejected by merchant, `104` duplicate. `998` and `999` are reserved for the bank.
- The RPN carries no signature, hash or HMAC field (checked 2026-10-21).
- Response URL changes take 5 working days to take effect.
- No refund, requery, recurring or tokenisation API is documented (checked 2026-10-21).

## Critical Rules

- Build and encrypt the send string server-side only; never let the browser choose `amount`, `payeeCode` or `accountNumber`.
- Do not trust the encrypted string for integrity: anyone can decrypt and re-encrypt it with the public key.
- Do not mark paid from the pop-up, the `Callbackurl` redirect, or front-end JavaScript.
- Accept RPN only over HTTPS at the registered Response URL, only from the documented Maybank IP for that environment.
- Match `AcctId` to a pending local order and `Amt` (cents) to the expected amount before settling; Maybank's own Java guide calls this mandatory.
- Make `RefId` unique in the database; reply `104` for repeats, never double-fulfil.
- Return `-1` only when you want Maybank to resend (for example your database is down).
- Keep `accountNumber` free of `$` because `$` is the send-string delimiter (inference from the format; not stated by Maybank).
- Swap the UAT Payee Code for the production Payee Code and `envType` 2 together at go-live.
- PHP SDK uses `mcrypt`, removed in PHP 7.2; port to `openssl_encrypt` with identical output, and confirm in UAT.

## Completion Criteria

A Maybank2u Pay integration is done only when:

1. Send string and encryption run server-side and match the official SDK output byte for byte.
2. Pop-up opens from a direct user click and posts `q` and `i` to the correct `actionUrl` for the environment.
3. Response URL, port and SSL certificate are registered for UAT and production in the portal.
4. RPN handler checks source IP, parses JSON `Msg`, matches `AcctId` and `Amt`, checks `StatusCode` `00`, and replies with the documented JSON.
5. Duplicate RPNs return `104`; failures return `101`; transient errors return `-1`.
6. Tests cover wrong IP, unknown `AcctId`, wrong amount, `01` status, duplicate `RefId`, and success.
7. UAT screenshots (pop-up and RPN received) are uploaded and UAT is confirmed in the portal.
8. Deployment docs list `M2UPAY_PAYEE_CODE`, `M2UPAY_ENV_TYPE`, Response URL, and allowed RPN IP per environment.

## Loophole Review Loop

When asked whether a Maybank2u Pay strategy is "100% confident", run this loop until no factual gaps remain:

1. List trust boundaries: checkout page, encryption endpoint, pop-up, Maybank2u login/TAC, Callbackurl redirect, RPN receiver, firewall, database, fulfilment job, payee report portal.
2. Check bypasses: tampered amount re-encrypted with the public key, fake redirect, forged RPN from a non-Maybank IP, RPN for an unknown or already-paid `AcctId`, amount in ringgit vs cents mix-up, duplicate RPN, RPN lost while returning `0`, UAT payee code in production, popup blocked, PHP mcrypt missing, Java line-wrapped Base64 mismatch.
3. Add fixes: server-only send string, strict IP allowlist, amount and reference match, unique `RefId`, durable event log, correct reply codes, daily report reconciliation, environment-specific config.
4. Re-read `references/maybank2u-pay-api.md` sections relevant to any gap.
5. State remaining uncertainty: no RPN signature, no documented requery or refund API, `i` values other than `OT`, `refNumber` and `callbackUrl` usage, current UAT URL, and current IPs must be confirmed with Maybank (`maybank2upay@maybank.com`).
