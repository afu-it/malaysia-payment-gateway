# iPay88 Source Map

Source: iPay88's own Technical Specification v1.6.4.4 (11 Sep 2024), not published on an iPay88 domain; obtained from a community mirror (`https://github.com/karyamedia/ipay88/blob/master/iPay88%20iPay88%20Technical%20API%20v1.6.4.4.pdf`). Merchants receive the current spec from iPay88 Support; confirm against it.

All sources checked on 2026-10-21.

## Primary: OPSG Technical Specification v1.6.4.4 (For Malaysia Only)

- Mirror: `https://raw.githubusercontent.com/karyamedia/ipay88/master/iPay88%20iPay88%20Technical%20API%20v1.6.4.4.pdf`

Use for (by section):

- 1.3 to 1.6: prerequisites (Merchant Code/Key, Request URL, IP whitelist, MYR 1.00 test, 3 working days live notice), service hours.
- 2.1: entry.asp, enquiry.asp, webservice requery URLs; Request URL and Response URL definitions.
- 2.2: logo size and formats.
- 2.3 and 2.4: payment request fields and sample form.
- 2.5 and 2.6: response fields, `Status` 1/0, sample response handlers.
- 2.7 and 2.10: Backend Post rules, success-only, simultaneous posts, `RECEIVEOK`, 3 retries, sessionless page.
- 2.8 and 2.9: Appdeeplink, TNG eWallet and Boost, ShopeePay Android WebView workaround.
- 2.11: enquiry.asp fields and replies (`00`, `M88Admin`, ...).
- 2.12: SOAP webservice requery, Version 4, response fields, `Status` 6 and 20.
- 3.0 to 3.3: security controls, HMAC-SHA512 request and response signature strings, examples, .NET and PHP code.
- 4.0 to 4.3: online report URL, email notification disclaimer.
- 5.0 FAQ: test amounts per currency, error messages, Merchant Checklist, change requests.
- Changelog: SHA256 to HMACSHA512 change on 11 Sep 2024 (v1.6.4.4).

## Official iPay88 pages (payment.ipay88.com.my)

- Signature tester: `https://payment.ipay88.com.my/epayment/testing/TestSignatureIpay88.asp`
  Use for: authority on the request signature string (MerchantKey + MerchantCode + RefNo + Amount + Currency, Amount with "." and "," removed) and HMAC-SHA512 keyed with the Merchant Key; available SignatureType options (SHA1, SHA256, HMACSHA512); manual "full string to hash" check for Xfield1.
- Payment page: `https://payment.ipay88.com.my/epayment/entry.asp`
  Use for: confirming the live host responds; support contact and careline shown on its error page.
- Re-query: `https://payment.ipay88.com.my/epayment/enquiry.asp`
  Use for: confirming the plain-text reply (`Record not found` returned for an unknown RefNo).
- Webservice service page: `https://payment.ipay88.com.my/epayment/webservice/TxInquiryCardDetails/TxDetailsInquiry.asmx`
  Use for: the operation list (one operation, `TxDetailsInquiryCardInfo`).
- Webservice WSDL: `https://payment.ipay88.com.my/epayment/webservice/TxInquiryCardDetails/TxDetailsInquiry.asmx?WSDL`
  Use for: SOAPAction value, SOAP 1.1, SOAP 1.2, HTTP GET and HTTP POST bindings, the full `TransactionInquiry` response type (refund, capture, void, DCC, token elements), `RefundTransaction` and `CaptureTransaction` types.
- Webservice operation page: `https://payment.ipay88.com.my/epayment/webservice/TxInquiryCardDetails/TxDetailsInquiry.asmx?op=TxDetailsInquiryCardInfo`
  Use for: sample SOAP request/response with headers, HTTP POST test form.
- Live probes on 2026-10-21 with a made-up merchant code (`M00000`, RefNo `TEST-NOTEXIST-1`): SOAP and HTTP POST webservice both returned HTTP 200 with empty `Status` and `Errdesc` `Record not found`; `enquiry.asp` via GET returned `Record not found`.
  Use for: the "no row" behaviour in `ipay88-api.md` sections 4 and 5.
- Signature tester currency dropdown (same tester URL as above).
  Use for: currency code list in `ipay88-api.md` section 9.

## iPay88's former website www.ipay88.com (offline now; Internet Archive copies)

- Support FAQ, "For Web Developer" block: `https://web.archive.org/web/2024/https://www.ipay88.com/support/`
  Use for: HTTP POST only, Request and Response URL definitions, testing with any valid credit card, test amounts per currency, error message list, "Server Re-query" fallback, support hours (Mon to Fri, 9 am to 6 pm), Technical Support line. Corroborates spec FAQ.
- Recurring Payment product page: `https://web.archive.org/web/2024/https://www.ipay88.com/ecommerce/recurring-payment/`
  Use for: confirming recurring, tokenisation, mobile SDK and external API exist as products, with no public spec.
- Appendix page: `https://web.archive.org/web/20220518162544/https://www.ipay88.com/appendix/`
  Use for: nothing technical; it indexes merchant application form appendices, not the PaymentId "Appendix I".
- Also checked, nothing technical: `/asset-library/`, `/general/`, `/ecommerce/direct-link/` (empty capture), `/plugins/` (2019 capture is a payment page error).

## Company and commercial

- `https://www.ipay88.com/` (redirects to NTT DATA Payment Services Malaysia)
- `https://my.nttdatapay.com/en/payment-services/adaptis-e-commerce`
- `https://my.nttdatapay.com/en/contact-us`

Use for: brand status, sales contact, pricing link. No developer docs here.

- Bank Negara Malaysia FSP directory, non-bank merchant acquirers: `https://www.bnm.gov.my/regulations/fsp-directory?p_p_id=com_liferay_asset_publisher_web_portlet_AssetPublisherPortlet_INSTANCE_jXC730NRlqU0&p_p_lifecycle=0&p_p_state=normal&p_p_mode=view&p_r_p_tag=merchant-acquirer-nonbank`
  Use for: licensed entity name (NTT DATA eCommerce Solutions Sdn. Bhd., formerly iPay88 (M) Sdn. Bhd.), as recorded in the MPG listing checked 2026-10-21.

## iPay88 Indonesia (official iPay88 group domain, out of scope)

- `https://ipay88.co.id/docs/pdf/iPay88_API_2.0.9_Technical_Documentation.pdf`
  Use for: nothing in this skill. It is the Indonesia JSON API 2.0; its section 8.4 "Malaysia Payment Method" lists 4-digit PaymentIDs for that gateway. Mentioned in `ipay88-api.md` only as a warning not to reuse them on the Malaysia OPSG.

## Older iPay88 specs (context only)

- v1.6.1 (marked "Private & Confidential"): `https://cdn1.npcdn.net/userfiles/11950/attachment/2292242/20190508_f3ddb86affa4bc8670c407ec4790c8d5/iPay88_Technical_Spec_V1.6.1.pdf`
- v1.6.2: `https://tzuchimalaysia.ddns.net/wp-content/uploads/events/20221201_20221201_test5December2022/20221201_iPay88-Technical-Specification-v1.6.2-For-Malaysia-Only2.pdf`

Use for: history only (SHA1 then SHA256 signatures, Merchant Checklist as Appendix A, flow diagram showing merchant re-query after success). Do not use for current signature rules.

## Coverage

Checked 2026-10-21.

- Index used: none published. `www.ipay88.com.my` does not resolve; `www.ipay88.com` redirects to `my.nttdatapay.com`, which has no developer section (its `sitemap.xml` and `llms.txt` return the site error page). `payment.ipay88.com.my` `robots.txt`, `sitemap.xml` and `llms.txt` all return the generic payment-page error, and `/epayment/testing/`, `/epayment/webservice/` have no listing.
- Official code: GitHub orgs `ipay88` and `ipay88-product` have no public repos. WordPress.org plugin search for "ipay88" returns no iPay88-authored plugin. `ipay88cambodia` (Cambodia SDKs) and `iPay88ID` (Indonesia) are out of scope.
- Internet Archive index of `ipay88.com`, `www.ipay88.com`, `ipay88.com.my` searched for PDFs and for URLs containing develop, plugin, appendix: no technical spec was ever published there (only application forms, merchant agreements, user manuals from 2008 to 2011, and campaign PDFs).
- Spec: 1 document, 5 chapters, 39 headings in its contents page (1.0 to 5.0 with subsections) plus the changelog. Read: 39 of 39.
- Official machine-readable source: 1 WSDL (1 operation, 4 bindings, 4 types). Read fully.
- Official pages read: 9 (signature tester, entry.asp, enquiry.asp, `.asmx` service page, `.asmx?op=` page, `?WSDL`, report login page, `sandbox.ipay88.com.my` reachability, robots/sitemap/llms probes). Archived www.ipay88.com pages read: 7 (support, recurring-payment, appendix, asset-library, general, direct-link, plugins).
- Endpoints documented: 11 rows in the catalogue in `ipay88-api.md` (8 operations, the webservice counted once per binding).

## Corrections made in the 2026-10-21 top-up

- `ipay88-api.md` section 5 said the SOAPAction was not printed and to read it from the service description. It is now given: `"https://www.mobile88.com/epayment/webservice/TxDetailsInquiryCardInfo"` (from the WSDL).
- Section 5 said the webservice response "carries no Signature". The spec has none, but the WSDL declares an optional `Signature` element with no documented string-to-sign. Text now says not to verify or rely on it.
- Section 5 named the response root as `TransactionInquiry` only. That is the HTTP GET/POST shape; SOAP wraps the fields in `TxDetailsInquiryCardInfoResponse/TxDetailsInquiryCardInfoResult`. Both are now documented.
- Status table did not cover the "no matching transaction" reply (empty `Status`); added from a live probe.

## Not documented / left out

- PaymentId table ("Appendix I.pdf", "Appendix II.pdf"): separate iPay88 files not in the mirror and not on any official host. An undated copy titled "Appendix I" is on Scribd (`https://www.scribd.com/document/83084969/Appendix-I`, login wall, third-party host, age unknown); not read or used. Community lists in GitHub libraries are not iPay88-authored, so they are not included.
- A community package page (Packagist `karyamedia/ipay88`) reposts an iPay88 notice with a 31 January 2025 HMAC-SHA512 deadline. Not an official host, so the date is not stated as fact in the skill.
- Refund, void, capture, recurring, tokenisation, payout, settlement report API, rate limits, iPay88 outbound IPs, sandbox host: not in the spec.
- Mobile SDKs for Malaysia: not published by iPay88 Malaysia.
- WooCommerce marketplace doc (`https://woocommerce.com/document/ipay88-payment-gateway/`): vendor not identified as iPay88, so not used for facts.
