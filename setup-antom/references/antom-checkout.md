# Antom Checkout And Payment Creation

Field tables for creating payments. Generated from the official API reference pages on 2026-10-21; descriptions are shortened (ending in `...`). Open the source page for the full text.

## Source URLs

- pay (One-time Payments): `https://docs.antom.com/ac/ams/payment_cashier.md`
- createPaymentSession (One-time Payments): `https://docs.antom.com/ac/ams/session_cashier.md`
- consult (One-time Payments): `https://docs.antom.com/ac/ams/consult.md`
- Hosted Checkout Page guide: `https://docs.antom.com/ac/cashierpay/HOSTEDCKP.md`
- Embedded Checkout Page guide: `https://docs.antom.com/ac/cashierpay/embeddedckp.md`
- Payment Element guide: `https://docs.antom.com/ac/cashierpay/element.md`
- APM API-only guide: `https://docs.antom.com/ac/cashierpay/apm_api.md`
- URLs returned per method: `https://docs.antom.com/ac/cashierpay/urls.md`

## Integration Modes

| Mode | Server call | Client step | Notes |
| --- | --- | --- | --- |
| Hosted Checkout Page | createPaymentSession, `productScene` `CHECKOUT_PAYMENT` | Redirect or open `normalUrl` (`https://checkout.antom.com/...`) | Low-code; Antom hosts method selection |
| Embedded Checkout Page | createPaymentSession, `CHECKOUT_PAYMENT` | Pass `paymentSessionData` unchanged to the web or mobile SDK | FPX does not support Embedded Checkout Page |
| Payment Element | createPaymentSession, `ELEMENT_PAYMENT` | `AMSElement` (or `loadAntom()` and `antom.createElement()`), `mount()`, `submitPayment()` | Web SDK 1.46.0 or later |
| API-only (APM) | pay with `paymentMethod.paymentMethodType` | Open `normalUrl`, `applinkUrl`, or `schemeUrl`, or show `orderCodeForm` | You build the method list (consult or brand assets) |
| Card server-to-server | pay with card fields in `paymentMethodMetaData` | Handle 3DS redirect if returned | Needs PCI qualification |

- Do not modify `paymentSessionData` before passing it to the client.
- For card payments on Checkout Page, `order.buyer` must carry at least one of `referenceBuyerId`, `buyerPhoneNo`, or `buyerEmail`.
- Cards, Apple Pay, and Google Pay use authorize then capture. Default `paymentFactor.captureMode` is `AUTOMATIC`; set `MANUAL` to call capture yourself.
- For APMs other than Apple Pay, Google Pay, and Pay by bank, leave `paymentFactor.isAuthorization` empty or `false`.
- `paymentRedirectUrl` is the result page. Antom puts no result data in it, and a redirect does not prove payment. Do not hard-code it as a success page. For apps, set it to your app scheme URL.
- If the buyer never submits a payment, no notification is sent when the session expires.

### terminalType

| Buyer device | `env.terminalType` | Extra |
| --- | --- | --- |
| PC browser | `WEB` | |
| Mobile browser | `WAP` | Add `env.osType` `ANDROID` or `IOS` |
| Mobile app | `APP` | Add `env.osType` |

### Returned URLs

| Field | Meaning |
| --- | --- |
| `normalUrl` | HTTPS page in the same browser or WebView |
| `applinkUrl` | Android App Link or iOS Universal Link; opens the app if installed, else a web page |
| `schemeUrl` | App scheme; fails if the app is not installed |

For FPX, all terminals return `normalUrl` to a login page. For Touch 'n Go eWallet, WEB returns a code-scanning `normalUrl`; WAP and APP return `applinkUrl` and `normalUrl`. Boost WAP and APP can return `schemeUrl`, `applinkUrl`, and `normalUrl`. GrabPay MY returns `normalUrl` only.

### Timeouts

- `paymentExpiryTime` on pay: default 14 minutes (48 hours for bank transfer). A custom value must be under 10 minutes, or under 48 hours for bank transfer.
- `paymentSessionExpiryTime` on createPaymentSession: default 1 hour, settable from 0 to 1 hour.

## Minimal MYR pay Request (API-only, FPX)

Placeholder values only. Field names follow the pay API reference.

```json
{
  "productCode": "CASHIER_PAYMENT",
  "paymentRequestId": "<UNIQUE_PAYMENT_REQUEST_ID>",
  "paymentAmount": { "currency": "MYR", "value": "1314" },
  "order": {
    "referenceOrderId": "<YOUR_ORDER_ID>",
    "orderDescription": "<ORDER_DESCRIPTION>",
    "orderAmount": { "currency": "MYR", "value": "1314" }
  },
  "paymentMethod": { "paymentMethodType": "ONLINEBANKING_FPX" },
  "paymentRedirectUrl": "https://<your-domain>/payment/result",
  "paymentNotifyUrl": "https://<your-domain>/antom/notify",
  "settlementStrategy": { "settlementCurrency": "<SETTLEMENT_CURRENCY>" },
  "env": { "terminalType": "WEB" }
}
```

- The pay page sample request for FPX uses `"paymentMethodType": "FPX_ONLINEBANKING"` and `terminalType` `"APP/WAP"`, but the payment method enum table, the FPX page, and the FPX response sample use `ONLINEBANKING_FPX`. Treat `ONLINEBANKING_FPX` as the enum and confirm in sandbox.
- To preselect an FPX bank, pass `paymentMethod.paymentMethodMetaData.bankIdentifierCode` (codes in `antom-codes.md`).

## pay (One-time Payments)

`POST /v1/payments/pay`

#### Request fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `merchantRegion` | String | max 2 | The country or region where the merchant operates the business. The parameter is a 2-letter country or region code that follows [ISO 3166 Country Codes](https://www.iso.o... |
| `merchantAccountId` | String | max 32 | A unique ID to identify a specific merchant account. Note: Specify this parameter when you use a single client ID across multiple locations. |
| `userRegion` | String | max 2 | A 2-letter country or region code based on the standard of ISO 3166 Country Codes. This parameter is used to sort digital wallets ac... |
| `env` | Env, REQUIRED |  | Information about the environment where the order is placed, such as the device information. |
| `env.terminalType` | String, REQUIRED | values: WEB,WAP,APP,MINI_APP | Terminal type of which the merchant service applies to. Valid values are: - `WEB`: The client-side terminal type is a website, which is opened via a PC browser. - `WAP`: ... |
| `env.osType` | String | values: IOS,ANDROID | Type of the operation system. Valid values are: - `IOS`: indicates the operation system is Apple's iOS. - `ANDROID`: indicates the operation system is Google's Android. >... |
| `env.browserInfo` | BrowserInfo |  | The information about the browser environment where the order is placed. Note: Specify this parameter when you collect this information during the payment process. |
| `env.browserInfo.acceptHeader` | String | max 2048 | The accept header of the user's browser. |
| `env.browserInfo.javaEnabled` | Boolean |  | Indicates whether the user's browser is able to run Java. Valid values are: - `true`: indicates that the user's browser is able to run Java. - `false`: indicates that the... |
| `env.browserInfo.javaScriptEnabled` | Boolean |  | Indicates whether the user's browser is able to run Java. Valid values are: - `true`: indicates that the user's browser is able to run JavaScript. - `false`: indicates th... |
| `env.browserInfo.language` | String | max 32 | The language of the user's browser. The value is obtained by using the browser's `navigator.language` property (as defined in IETF BCP 47). |
| `env.browserInfo.userAgent` | String | max 2048 | The user agent of the user's browser. Note: Specify this parameter when one of the following conditions is met: - You require risk control. - The value of _pa... |
| `env.colorDepth` | Integer |  | The color depth of the user's browser in bits per pixel. The value is obtained by using the browser's screen.colorDepth property. Valid values are `1`, `4`, `8`, `15`, `1... |
| `env.screenHeight` | Integer |  | The screen height of the user's device in pixels. Note: Specify this parameter if you require risk control. Providing this information helps to increase the accurac... |
| `env.screenWidth` | Integer |  | The screen width of the user's device in pixels. Note: Specify this parameter if you require risk control. Providing this information helps to increase the accuracy... |
| `env.timeZoneOffset` | Integer |  | The time difference between UTC time and the local time of the user's browser, in minutes. The value is obtained by using the `getTimezoneOffset()` property. For example,... |
| `env.deviceBrand` | String | max 64 | The brand of the user's device. Note: Specify this parameter if you require risk control. Providing this information helps to increase the accuracy of anti-money la... |
| `env.deviceModel` | String | max 128 | The model of the user's device. Note: Specify this parameter if you require risk control. Providing this information helps to increase the accuracy of anti-money la... |
| `env.deviceTokenId` | String | max 64 | The token identifier of the device. Note: Specify this parameter if you integrate with the Antom Device Fingerprint client, which is an SDK or JavaScript library th... |
| `env.clientIp` | String | max 64 | The IP address of the client device. Note: Specify this parameter when one of the following conditions is met: - When you require risk control. - When the val... |
| `env.deviceLanguage` | String | max 32 | The language of the device where the user places an order. Note: Specify this parameter if you require risk control. Providing this information helps to identify bl... |
| `env.deviceId` | String | max 64 | The unique ID of the device the user places an order with. Note: Specify this parameter if you require risk control. This parameter can be used to build a trusted r... |
| `env.extendInfo` | String | max 2048 | Extended information. Specify this field if you need to use the extended information. |
| `order` | Order, REQUIRED |  | The order information, such as buyer, merchant, goods, amount, shipping information, and purchase environment. This field is used for different purposes: - During the pay... |
| `order.orderAmount` | Amount, REQUIRED |  | The order amount of the merchant that directly provides services or goods to the customer. This field is used for user consumption records display or payment results page... |
| `order.orderAmount.currency` | String, REQUIRED | max 3 | The transaction currency that is specified in the contract. A 3-letter currency code that follows the ISO 4217 standar... |
| `order.orderAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). Note... |
| `order.referenceOrderId` | String, REQUIRED | max 64 | The unique ID to identify the order on the merchant side, which is assigned by the merchant that provides services or goods directly to the customer. This field is used f... |
| `order.orderDescription` | String, REQUIRED | max 256 | Summary description of the order, which is used for user consumption records display or other further actions. |
| `order.goods` | Array |  | Goods information, including the ID, name, price, and quantity of the goods in the order. Note: Specify this parameter when one of the following conditions is met: ... |
| `order.goods.referenceGoodsId` | String, REQUIRED | max 64 | The unique ID to identify the goods. |
| `order.goods.goodsName` | String, REQUIRED | max 256 | Goods name. |
| `order.goods.goodsCategory` | String | max 64 | The category of the goods. - When the value of _paymentMethodType_ is `SEQURA`, pass in one of the following values: - `product` - `service` - `flight` - `lodging` - This... |
| `order.goods.goodsUnitAmount` | Amount |  | Price of goods. Note: Specify this parameter when one of the following conditions is met: - When you require risk control. - This parameter is required when t... |
| `order.goods.currency` | String, REQUIRED | max 3 | The 3-letter currency code that follows the ISO 4217 standard. |
| `order.goods.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). Note... |
| `order.goods.goodsQuantity` | Integer |  | Quantity of goods. Note: Specify this parameter when one of the following conditions is met: - When you require risk control. - This parameter is required whe... |
| `order.goods.goodsUrl` | String | max 2048 | The URL of the website where the user places an order. Note: Specify this parameter when one of the following conditions is met: - When you require risk control... |
| `order.goods.deliveryMethodType` | String | max 32; values: PHYSICAL,DIGITAL | The delivery method of the goods. Valid values are: - `PHYSICAL`: indicates that the delivery method is physical delivery. - `DIGITAL`: indicates that the delivery method... |
| `order.goods.goodsDiscountAmount` | Amount |  | The merchant-side discount amount for the goods. This parameter is used by the payment method for amount balance validation. The calculation must satisfy the following fo... |
| `order.goods.currency` | String, REQUIRED | max 3 | A 3-letter currency code that follows the ISO 4217 standard. |
| `order.goods.value` | Integer, REQUIRED |  | A positive integer in the smallest currency unit. For example, if the currency is USD and the amount is $1.00, set the value of this parameter to `100`; or if the currenc... |
| `order.goods.goodsEndsOnTime` | Datetime | max 30 | The maximum end date for the service goods. Note: - This parameter is required when the _order.goods.goodsCategory_ parameter is specified as `service` and the ... |
| `order.shipping` | Shipping |  | Shipping information, including information about the recipient (name, phone number, email, and shipping address), and shipping service provider. Note: Specify this... |
| `order.shipping.shippingName` | UserName |  | The name of the recipient. Note: Specify this parameter if you require risk control. Providing this information helps to increase the accuracy of anti-money launder... |
| `order.shipping.firstName` | String, REQUIRED | max 32 | First name. |
| `order.shipping.middleName` | String | max 32 | Middle name |
| `order.shipping.lastName` | String, REQUIRED | max 32 | Last name |
| `order.shipping.fullName` | String | max 128 | Full name |
| `order.shipping.shippingAddress` | Address |  | Shipping address. Notes: Specify this parameter when one of the following conditions is met: - The value of _paymentMethodType_ is `PAGALEVE`. - You require r... |
| `order.shipping.region` | String, REQUIRED | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. |
| `order.shipping.state` | String | max 8 | The state, country, or province name. Note: For card payments, if your business entity is in the United States, and the card issuing country is in Canada, the Unite... |
| `order.shipping.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `order.shipping.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `order.shipping.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `order.shipping.zipCode` | String | max 32 | ZIP or postal code. Note: For card payments, if your business entity is in the United States, specify this parameter according to the following parameter value requ... |
| `order.shipping.shippingCarrier` | String | max 128 | The delivery service provider for shipping a physical product, such as FedEx, UPS, or USPS. Notes: - Specify this parameter if you require risk control. Providi... |
| `order.shipping.shippingPhoneNo` | String | max 16 | The phone number of a recipient (including extension). Note: Specify this parameter when one of the following conditions is met: - You require risk control. -... |
| `order.shipping.shipToEmail` | String | max 64 | The email address where virtual goods are sent. Note: Specify this parameter when one of the following conditions is met: - if you require risk control. - if ... |
| `order.buyer` | Buyer |  | Buyer information, including the ID, name, phone number, and email of the buyer. Note: Specify this parameter when one of the following conditions is met: - You... |
| `order.buyer.referenceBuyerId` | String | max 64 | The unique user ID on the merchant side. Note: Specify this parameter when one of the following conditions is met: - When you require risk control. - When the... |
| `order.buyer.buyerName` | UserName |  | The name of the buyer. Notes: Specify this parameter when one of the following conditions is met: - You require risk control. - The value of _paymentMethodTyp... |
| `order.buyer.firstName` | String | max 32 | First name. Notes: - This parameter is required when the value of _paymentMethodType_ is `SEQURA`. - This parameter is required when the value of _paymentMeth... |
| `order.buyer.middleName` | String | max 32 | Middle name |
| `order.buyer.lastName` | String | max 32 | Last name Notes: - This parameter is required when the value of _paymentMethodType_ is `SEQURA`. - This parameter is required when the value of _paymentMethod... |
| `order.buyer.fullName` | String | max 128 | Full name |
| `order.buyer.buyerPhoneNo` | String | max 24 | The mobile phone number of the buyer. Note: Specify this parameter when one of the following conditions is met: - When you require risk control. - When the va... |
| `order.buyer.buyerEmail` | Email | max 64 | The email of the buyer. Notes: Specify this parameter when one of the following conditions is met: - When you require risk control. - When the value of _payme... |
| `order.buyer.buyerRegistrationTime` | Datetime | max 64 | The time when the buyer registered your account. Specify this parameter if you require risk control. Providing this information helps to increase the accuracy of anti-mon... |
| `order.merchant` | Merchant |  | Information about the secondary merchant. Note: Specify this field if you are an acquirer that has secondary merchants. |
| `order.merchant.referenceMerchantId` | String, REQUIRED | max 32 | The ID to identify the merchant that directly provides services or goods to the customer. |
| `order.merchant.merchantMCC` | String | max 32 | The secondary merchant category code, which is a 4-digit number listed in MCC codes. Note: Specify this field if you ha... |
| `order.merchant.merchantName` | String | max 256 | The merchant name. Note: Specify this field if you have the information. Providing this information helps to increase the accuracy of anti-money laundering and frau... |
| `order.merchant.merchantDisplayName` | String | max 64 | The merchant name to be displayed. Note: Specify this field if you have the information. Providing this information helps to increase the accuracy of anti-money lau... |
| `order.merchant.merchantAddress` | Address |  | The merchant address information. Note: Specify this field if you have the information. Providing this information helps to increase the accuracy of anti-money laun... |
| `order.merchant.region` | String, REQUIRED | max 2 | The two-character ISO 3166 country/region code. |
| `order.merchant.state` | String | max 8 | The state, country, or province. |
| `order.merchant.city` | String | max 32 | The city, district, suburb, town, or village. |
| `order.merchant.address1` | String | max 256 | Address line 1, such as street, PO Box, or company name. |
| `order.merchant.address2` | String | max 256 | Address line 2, such as the apartment, suite, unit, or building information. |
| `order.merchant.zipCode` | String | max 32 | ZIP or postal code. |
| `order.merchant.merchantRegisterDate` | Datetime |  | Merchant registration date. Note: Specify this field if you have the information. Providing this information helps to increase the accuracy of anti-money laundering... |
| `order.extendInfo` | String | max 2048 | Extended information data, including information for special use cases. Note: Specify this field when you need to use the extended information. |
| `order.extendInfo.chinaExtraTransInfo` | ChinaExtraTransInfo |  | Extra information about the transaction. Note: Specify this field for transactions to Alipay. |
| `order.extendInfo.businessType` | String, REQUIRED | max 16; values: 1,2,3,4,5 | The business type. Valid values are: - `1`: Hotel - `2`: Air Flight - `3`: Study Abroad - `4`: Trade - `5`: Other Multiple business types are delimited by "/". |
| `order.extendInfo.flightNumber` | String | max 1000 | The flight number. Multiple flight numbers are delimited by '/'. When _businessType_\=2, this field needs to be specified. |
| `order.extendInfo.departureTime` | Datetime | max 1000 | The flight departure time in the format of yyyy-MM-dd HH:mm. Multiple flight departure times are delimited by '/'. When _businessType_\=2, this field needs to be specifie... |
| `order.extendInfo.hotelName` | String | max 1000 | The hotel name. Multiple hotel names are delimited by '/'. When _businessType_\=1, this field needs to be specified. |
| `order.extendInfo.checkinTime` | Datetime | max 1000 | The hotel check-in time in the format of yyyy-MM-dd and in the UTC+8 timezone. Multiple hotel check-in times are delimited by '/'. When _businessType_\=1, this field need... |
| `order.extendInfo.admissionNoticeUrl` | URL | max 1000 | The school admission notice URL. Mulitple URLs are delimited by '/'. When _businessType_\=3, this field needs to be specified. |
| `order.extendInfo.goodsInfo` | String |  | Goods information that includes SKU names and corresponding quantities, in the format of SKU\_name^quantity. If more than one goods exists, separate values with a vertica... |
| `order.extendInfo.totalQuantity` | Integer |  | Total quantities of all goods in one order. When _businessType_\=4, this field needs to be specified. - Value range: 0 - unlimited |
| `order.transit` | Transit |  | Trip information, including trip modes, legs of trips and passenger information. Note: Specify this field if you require risk control. Providing this information he... |
| `order.transit.transitType` | String | values: FLIGHT,TRAIN,CRUISE,COACH | Trip modes, valid values are: - `FLIGHT`: indicates that the trip mode is by plane. - `TRAIN`: indicates that the trip mode is by train. - `CRUISE`: indicates that the tr... |
| `order.transit.agentCode` | String | max 8 | International Air Transport Association (IATA) code for the travel agency that made the reservation. |
| `order.transit.agentName` | String | max 32 | The name of the travel agency that made the reservation. |
| `order.transit.ticketNumber` | String | max 15 | A unique identifier for the ticket, typically formatted as IATA 3-letter airline code followed by a 10-digit number assigned by the airline. |
| `order.transit.ticketIssuerCode` | String | max 2 | Unique IATA 2-letter airline code identifying the organization or entity that issued the ticket. |
| `order.transit.restrictedTicketIndicator` | String | max 2; values: 0,1 | Indicates whether the ticket is refundable. Valid values include: - `0`: Indicates the ticket is refundable. - `1`: Indicates the ticket is not refundable. |
| `order.transit.legs` | Array |  | Information about sections of the trip, including departure time, arrival time, departure address, arrival address, transportation company name, carrier code and service ... |
| `order.transit.departureTime` | Datetime |  | Time of departure for this leg of the trip. Note: Specify this parameter when the value of _paymentMethodType_ is `SEQURA`, and the value of _order.goods.goodsCateg... |
| `order.transit.arrivalTime` | Datetime |  | Time of arrival for this leg of the trip. Note: Specify this parameter when the value of _paymentMethodType_ is `SEQURA`, and the value of _order.goods.goodsCategor... |
| `order.transit.departureAddress` | Address |  | Departure address for this leg of the trip. |
| `order.transit.region` | String | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. |
| `order.transit.state` | String | max 8 | The state, country, or province. |
| `order.transit.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `order.transit.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `order.transit.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `order.transit.arrivalAddress` | Address |  | Destination address for this leg of the trip. |
| `order.transit.region` | String | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. |
| `order.transit.state` | String | max 8 | The state, country, or province. |
| `order.transit.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `order.transit.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `order.transit.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `order.transit.carrierName` | String | max 128 | Company name of the transportation service provider for this leg of the trip. |
| `order.transit.classType` | String | max 32; values: FIRSTLEVEL,SECONDLEVEL,THIRDLEVEL | Service level for this leg of the trip. Valid values are: - `FIRSTLEVEL`: indicates that the service is the first class. For example, flight first class, cruise luxury se... |
| `order.transit.carrierNo` | String | max 2 | International Air Transport Association (IATA) code is obtained from the Official Airline Guide or its equivalent. Note: Specify this parameter when the value of _p... |
| `order.transit.fareBasis` | String | max 16 | Code used to identify the basic rules of segment fares. This code is defined by the airline to identify ticket attributes, such as cabin class (e.g., business class), far... |
| `order.transit.couponNumber` | String | max 2 | An electronic coupon number used to identify a flight segment in a ticket. Each ticket may contain multiple electronic coupons, each corresponding to an independent fligh... |
| `order.transit.flightNumber` | String | max 5 | The identifier assigned to a specific flight. Typically 1 to 4 numeric digits, optionally followed by a suffix letter. |
| `order.transit.passengers` | Array |  | Information about the passenger of the trip, including the passenger names, passenger email and passenger phone number. - Maximum size: 100 elements |
| `order.transit.passengerName` | UserName |  | The name of the passenger. |
| `order.transit.firstName` | String | max 32 | First name. Note: Specify this parameter when the value of _paymentMethodType_ is `SEQURA`, and the value of _order.goods.goodsCategory_ is `flight`. |
| `order.transit.middleName` | String | max 32 | Middle name. |
| `order.transit.lastName` | String | max 32 | Last name. Note: Specify this parameter when the value of _paymentMethodType_ is `SEQURA`, and the value of _order.goods.goodsCategory_ is `flight`. |
| `order.transit.fullName` | String | max 128 | Full name. |
| `order.transit.passengerEmail` | String | max 64 | The email of the passenger. |
| `order.transit.passengerPhoneNo` | String | max 32 | The mobile phone number of the passenger. |
| `order.transit.passengerId` | String | max 64 | The unique ID that is assigned by merchant to identify a passenger. Notes: - The parameter must be encoded in SHA-256 format. - Note: Specify this paramet... |
| `order.transit.passengerIdType` | String | max 64; values: PASSPORT,NATIONAL_ID_CARD,DRIVER_LICENSE,GREEN_CARD,TRAVEL_DOCUMENT,ALIEN_REGISTRATION_CARD,BIRTH_CERTIFICATE,OTHERS | The type of passenger's ID. Valid values are: - `PASSPORT` : Widely accepted for international and domestic travel. - `NATIONAL_ID_CARD`: National ID cards, which is vali... |
| `order.transit.passengerCode` | String | max 32 | The unique identifier assigned by the air carrier to a passenger for accurately identifying the passenger in the airline system. |
| `order.transit.ancillaryData` | Object |  | Information about an ancillary purchase. |
| `order.transit.connectedTicketNumber` | String | max 15 | The airline ticket number linked to this ancillary purchase. If the ancillary product is associated with a flight ticket (like a baggage fee), enter that flight ticket nu... |
| `order.transit.services` | Array |  | Information about the services included in an ancillary purchase. |
| `order.transit.categoryCode` | String | max 4 | Category code for the ancillary services offered. Retrieve the codes from the International Air Transport Association (IATA). |
| `order.transit.subCategoryCode` | String | max 4 | Subcategory code for the ancillary service category. Retrieve codes from the International Air Transport Association (IATA). |
| `order.lodging` | Lodging |  | Information about lodging service that was purchased, including hotel name, hotel address, check-in date, check-out date, number of booked rooms, number of booked nights ... |
| `order.lodging.hotelName` | String | max 128 | Hotel name. Note: Specify this parameter when the value of _paymentMethodType_ is `SEQURA`, and the value of _order.goods.goodsCategory_ is `lodging`. |
| `order.lodging.hotelAddress` | Address |  | Hotel address. |
| `order.lodging.region` | String | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. Note: Specify this parameter w... |
| `order.lodging.state` | String | max 8 | The state, country, or province. |
| `order.lodging.city` | String | max 32 | The city, district, suburb, town, or village name. Note: Specify this parameter when the value of _paymentMethodType_ is `SEQURA`, and the value of _order.goods.goo... |
| `order.lodging.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `order.lodging.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `order.lodging.checkInDate` | Datetime |  | Date on which the guest checked in. In the case of a no-show or a reservation, the scheduled arrival date. Note: Specify this parameter when the value of _paymentMe... |
| `order.lodging.checkOutDate` | Datetime |  | Date on which the guest checked out. Note: Specify this parameter when the value of _paymentMethodType_ is `SEQURA`, and the value of _order.goods.goodsCategory_ is... |
| `order.lodging.numberOfNights` | Integer |  | Number of nights booked by the payer. - Value range: 1 - unlimited |
| `order.lodging.numberOfRooms` | Integer |  | Number of rooms booked by the payer. - Value range: 1 - unlimited |
| `order.lodging.guestNames` | Array |  | Name of the guest under which the room is reserved. - Maximum size: 100 elements |
| `order.lodging.firstName` | String | max 32 | First name. |
| `order.lodging.middleName` | String | max 32 | Middle name. |
| `order.lodging.lastName` | String | max 32 | Last name. |
| `order.lodging.fullName` | String | max 128 | Full name. |
| `order.gaming` | Gaming |  | Information about gaming top-up, including game name, the topped up user name or ID, user email and user phone number. Note: Specify this parameter if you require r... |
| `order.gaming.gameName` | String | max 128 | Game name. |
| `order.gaming.toppedUpUser` | String | max 128 | Name or ID of the user whose gaming account is topped up. |
| `order.gaming.toppedUpEmail` | String | max 64 | Email of the user whose gaming account is topped up. |
| `order.gaming.toppedUpPhoneNo` | String | max 32 | Phone number of the user whose gaming account is topped up. |
| `order.declaration` | Declaration |  | Foreign exchange settlement declaration. |
| `order.declaration.declarationBizScene` | String | max 64; values: AIRLINE,HOTEL | Industry classification for the foreign exchange settlement declaration. Value valid are: - `AIRLINE`: Indicates that the declared industry classification is the airline ... |
| `order.declaration.declarationBeneficiaryId` | String | max 64 | The unique third-party beneficiary ID used for declaring cumulative amount. Note: This is a required parameter in OTA scenarios, and must be passed together with _decla... |
| `paymentRequestId` | String, REQUIRED | max 64 | The unique ID assigned by a merchant to identify a payment request. Antom uses this field for idempotence control. - This field is an API idempotency field.For payment re... |
| `paymentAmount` | Amount, REQUIRED |  | The payment amount that the merchant requests to receive in the order currency. |
| `paymentAmount.currency` | String, REQUIRED | max 3 | The transaction currency, which is a 3-letter currency code that follows the ISO 4217 standard. |
| `paymentAmount.value` | Integer, REQUIRED |  | The value of the amount as a positive integer in the smallest currency unit. For example, if the currency is USD and the amount is $1.00, set the value of this parameter ... |
| `settlementStrategy` | SettlementStrategy, REQUIRED |  | The settlement strategy for the payment request. |
| `settlementStrategy.settlementCurrency` | String, REQUIRED | max 3 | The ISO currency code of the currency that the merchant wants to be settled against. The field is required if the merchant signed up for multiple currencies to settle. |
| `paymentMethod` | PaymentMethod, REQUIRED |  | The payment method that is used to collect the payment by the merchant or acquirer. |
| `paymentMethod.paymentMethodType` | String, REQUIRED | max 64 | The payment method type that is included in payment method options. By specifying the value of this parameter, you can receive the cashier URL of the specified payment ... |
| `paymentMethod.paymentMethodId` | String | max 128 | The unique ID that is used to identify a payment method. The value of this parameter is that of _cardToken_ obtained from the [notifyPayment](https://docs.antom.com/a... |
| `paymentMethod.paymentMethodMetaData` | PaymentMethodMetaData |  | Additional information required for certain payment methods. Note: Specify this parameter when the value of _paymentMethodType_ is `APPLEPAY`, `BLIK`, `CARD`, `GOOG... |
| `paymentMethod.paymentMethodMetaData.applePayConfiguration` | ApplePayConfiguration |  | Apple Pay payment preferences. |
| `paymentMethod.paymentMethodMetaData.applePayToken` | String | max 4096 | Apple Pay token. Note: Specify this parameter when the merchant does not have PCI qualification. |
| `paymentMethod.paymentMethodMetaData.is3DSAuthentication` | Boolean |  | Indicates whether the transaction authentication type is 3D secure. Valid values are: - `true`: indicates that the transaction authentication type is 3D secure. - `false`... |
| `paymentMethod.paymentMethodMetaData.cardNo` | String | max 32 | The card number. Note: Specify this parameter when the merchant is PCI DSS compliant and has decrypted the _applePayToken_. |
| `paymentMethod.paymentMethodMetaData.cvv` | String | max 4 | The card verification value (CVV), which is also known as a card security code (CSC) or a card veirification code (CVC). Note: Specify this parameter when the merch... |
| `paymentMethod.paymentMethodMetaData.expiryMonth` | String | max 2 | The month the card expires. Pass in two digits representing the month. For example, if the expiry month is February, the value of this parameter is `02`. Note: Spec... |
| `paymentMethod.paymentMethodMetaData.expiryYear` | String | max 2 | The year the card expires. Pass in the last two digits of the year number. For example, if the expiry year is 2025, the value of this parameter is `25`. Note: Speci... |
| `paymentMethod.paymentMethodMetaData.cardholderName` | UserName |  | The cardholder's name. Note: Specify this parameter when the merchant is PCI DSS compliant and has decrypted the _applePayToken_. |
| `paymentMethod.paymentMethodMetaData.firstName` | String | max 32 | The cardholder's first name. |
| `paymentMethod.paymentMethodMetaData.middleName` | String | max 32 | The cardholder's middle name. |
| `paymentMethod.paymentMethodMetaData.lastName` | String | max 32 | The cardholder's last name. |
| `paymentMethod.paymentMethodMetaData.fullName` | String | max 32 | The cardholder's full name. |
| `paymentMethod.paymentMethodMetaData.billingAddress` | Address |  | The billing address that is associated with the cardholder's card account. Providing this information helps to increase the accuracy of anti-money laundering and fraud de... |
| `paymentMethod.paymentMethodMetaData.region` | String, REQUIRED | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. |
| `paymentMethod.paymentMethodMetaData.state` | String | max 8 | The state, country, or province name. |
| `paymentMethod.paymentMethodMetaData.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `paymentMethod.paymentMethodMetaData.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `paymentMethod.paymentMethodMetaData.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `paymentMethod.paymentMethodMetaData.zipCode` | String | max 32 | The ZIP or postal code. |
| `paymentMethod.paymentMethodMetaData.paymentMethodRegion` | String | max 6 | The region code that represents the country or region of the payment method. The value of this parameter is a 2-letter [ISO country code](https://www.iso.org/obp/ui/#sear... |
| `paymentMethod.paymentMethodMetaData.tokenize` |  |  | Deprecated. Use _tokenizeMode_ instead. |
| `paymentMethod.paymentMethodMetaData.tokenizeMode` | String | max 64; values: ENABLED,DISABLED,ASKFORCONSENT | The tokenize mode. Indicates whether card information needs to be stored. When you opt to store card details, the actual information is replaced with a unique and secure ... |
| `paymentMethod.paymentMethodMetaData.isCardOnFile` | Boolean |  | Indicates whether the card used in this transaction has been previously processed and stored by the merchant. Valid values are: - `true`: The card used in the transaction... |
| `paymentMethod.paymentMethodMetaData.recurringType` | String | max 64; values: SCHEDULED,UNSCHEDULED | The recurring type for the payment. Valid values are: - `SCHEDULED`: indicates a regular automated deduction payment. - `UNSCHEDULED`: indicates an irregular automated de... |
| `paymentMethod.paymentMethodMetaData.networkTransactionId` | String | max 100 | This parameter is used for automated deduction payments. The value of this parameter is the value of the same field that is returned by [notifyPayment](https://docs.a... |
| `paymentMethod.paymentMethodMetaData.selectedCardBrand` | String | max 64 | The card brand that the user used to pay. See Card brands to check the valid values. |
| `paymentMethod.paymentMethodMetaData.enableAuthenticationUpgrade` | Boolean |  | Indicates whether you allow non-3D Secure authentication to be upgraded to 3D Secure authentication. Valid values are: - `true`: indicates that you allow non-3D Secure au... |
| `paymentMethod.paymentMethodMetaData.mpiData` | MpiData |  | The MPI (Merchant Plug-In) information. Note: Specify this parameter if you integrate external 3D MPI. |
| `paymentMethod.paymentMethodMetaData.threeDSVersion` | String, REQUIRED | max 16 | The version of 3D Secure protocol. Valid values are: - `1.0.2` - `2.1.0` - `2.2.0` |
| `paymentMethod.paymentMethodMetaData.cavv` | String | max 64 | The cardholder authentication value. The value of this parameter can be Cardholder Authentication Verification Value (CAVV) or Authentication Verification Value (AVV). ... |
| `paymentMethod.paymentMethodMetaData.dsTransactionId` | String | max 64 | The unique transaction identifier assigned by the Directory Server (DS) for 3D Secure authentication. Note: Specify this parameter when 3D Secure authentication is ... |
| `paymentMethod.paymentMethodMetaData.eci` | String, REQUIRED | max 2; values: 02,01,00 | Electronic Commerce Indicator (ECI) that is returned by the card scheme. This parameter is used to indicate the type of cardholder identity authentication. Valid values a... |
| `paymentMethod.paymentMethodMetaData.exemptionType` | String | max 64 | The 3D Secure exemption type specified by the merchant to request Strong Customer Authentication (SCA) exemption from the issuing bank. Once specified, Antom will pass th... |
| `paymentMethod.paymentMethodMetaData.blikCode` | String | max 6 | The unique 6-digit BLIK code that is generated on the user's banking application. The code expires in 2 minutes. Note: Specify this parameter when all the following... |
| `paymentMethod.paymentMethodMetaData.is3DSAuthentication` | Boolean |  | Indicates whether 3D Secure authentication is mandatory for the payment. Valid values are: - `true`: 3D Secure authentication is enforced for the payment. - `false`: 3D S... |
| `paymentMethod.paymentMethodMetaData.is3DSDataOnly` | Boolean |  | Indicates whether the transaction is processed via the 3D Secure data-only flow. Valid values are: - `true`: indicates that the transaction that qualifies will automatica... |
| `paymentMethod.paymentMethodMetaData.cardNo` | String | max 32 | The card number. Note: Specify this parameter when all the following conditions are met: - You have the PCI qualification. - The value of _paymentMethodType_ ... |
| `paymentMethod.paymentMethodMetaData.cvv` | String | max 4 | The card verification value (CVV), which is also known as a card security code (CSC) or a card veirification code (CVC). Note: Specify this parameter when all the f... |
| `paymentMethod.paymentMethodMetaData.expiryYear` | String | max 2 | The year the card expires. Pass in the last two digits of the year number. For example, if the expiry year is 2025, the value of this parameter is `25`. Note: Speci... |
| `paymentMethod.paymentMethodMetaData.expiryMonth` | String | max 2 | The month the card expires. Pass in two digits representing the month. For example, if the expiry month is February, the value of this parameter is `02`. Note: Spec... |
| `paymentMethod.paymentMethodMetaData.cardholderName` | UserName |  | The cardholder's name. Note: Specify this parameter when all the following conditions are met: - You have the PCI qualification. - The value of _paymentMethod... |
| `paymentMethod.paymentMethodMetaData.firstName` | String, REQUIRED | max 32 | The cardholder's first name. |
| `paymentMethod.paymentMethodMetaData.middleName` | String | max 32 | The cardholder's middle name. |
| `paymentMethod.paymentMethodMetaData.lastName` | String, REQUIRED | max 32 | The cardholder's last name. |
| `paymentMethod.paymentMethodMetaData.fullName` | String | max 128 | The cardholder's full name. |
| `paymentMethod.paymentMethodMetaData.billingAddress` | Address |  | The billing address that is associated with the cardholder's card account. Note: Specify this parameter when _paymentMethodType_ is `CARD` and you require risk cont... |
| `paymentMethod.paymentMethodMetaData.region` | String, REQUIRED | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. |
| `paymentMethod.paymentMethodMetaData.state` | String | max 8 | The state, country, or province name. Note: For card payments, if your business entity is in the United States, and the card issuing country is in Canada, the Unite... |
| `paymentMethod.paymentMethodMetaData.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `paymentMethod.paymentMethodMetaData.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `paymentMethod.paymentMethodMetaData.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `paymentMethod.paymentMethodMetaData.zipCode` | String | max 32 | The ZIP or postal code. Note: For card payments, if your business entity is in the United States, specify this parameter according to the following parameter value ... |
| `paymentMethod.paymentMethodMetaData.cpf` | String | max 11 | The Cadastro Pessoal de Pessoa Física (CPF) is the tax ID of the Brazilian individual taxpayer. Note: Specify this parameter when all the following conditions are m... |
| `paymentMethod.paymentMethodMetaData.dateOfBirth` | String | max 10 | The date of birth of the cardholder. The value of this parameter is an 8-digit date of birth in the format of `YYYY-MM-DD` that follows the [ISO 8601](https://www.iso.org... |
| `paymentMethod.paymentMethodMetaData.businessNo` | String | max 10 | The business number of the company that holds the corporate card. The value of this parameter is a 10-digit business number, such as `97XXXXXX11`. Note: Specify thi... |
| `paymentMethod.paymentMethodMetaData.cardPasswordDigest` | String | max 2 | The first two digits of the card payment password. Note: Specify this parameter when all the following conditions are met: - You have the PCI qualification. -... |
| `paymentMethod.paymentMethodMetaData.paymentMethodRegion` | String | max 6 | The region code that represents the country or region of the payment method. The value of this parameter is a 2-letter [ISO country code](https://www.iso.org/obp/ui/#sear... |
| `paymentMethod.paymentMethodMetaData.payerEmail` | Email | max 64 | The email address of the payer. Note: Specify this parameter when all the following conditions are met: - _paymentMethodType_ is `CARD`. - _paymentMethodRegio... |
| `paymentMethod.paymentMethodMetaData.tokenizeMode` | String | max 64; values: ENABLED,DISABLED,ASKFORCONSENT | The tokenize mode. Indicates whether card information needs to be stored. When you opt to store card details, the actual information is replaced with a unique and secure ... |
| `paymentMethod.paymentMethodMetaData.isCardOnFile` | Boolean |  | Indicates whether the card used in this transaction has been previously processed and stored by the merchant. Valid values are: - `true`: The card used in the transaction... |
| `paymentMethod.paymentMethodMetaData.recurringType` | String | max 64; values: SCHEDULED,UNSCHEDULED | The recurring type for the payment. Valid values are: - `SCHEDULED`: indicates a regular automated deduction payment. - `UNSCHEDULED`: indicates an irregular automated de... |
| `paymentMethod.paymentMethodMetaData.networkTransactionId` | String | max 100 | The unique ID assigned by the card scheme to identify a transaction. This parameter is used for automated deduction payments. The value of this parameter is the value of ... |
| `paymentMethod.paymentMethodMetaData.selectedCardBrand` | String | max 64 | The card brand selected by the user for payment. See Card brands to check the valid values. Note: - ... |
| `paymentMethod.paymentMethodMetaData.enableAuthenticationUpgrade` | Boolean |  | Indicates whether you allow non-3D Secure authentication to be upgraded to 3D Secure authentication. Valid values are: - `true`: indicates that you allow non-3D Secure au... |
| `paymentMethod.paymentMethodMetaData.mpiData` | MpiData |  | The MPI (Merchant Plug-In) information. Specify this parameter if you integrate external 3D MPI. |
| `paymentMethod.paymentMethodMetaData.threeDSVersion` | String, REQUIRED | max 16 | The version of 3D Secure protocol. Valid values are: - `1.0.2` - `2.1.0` - `2.2.0` |
| `paymentMethod.paymentMethodMetaData.cavv` | String | max 64 | The cardholder authentication value. The value of this parameter can be Cardholder Authentication Verification Value (CAVV) or Authentication Verification Value (AVV). ... |
| `paymentMethod.paymentMethodMetaData.dsTransactionId` | String | max 64 | The unique transaction identifier assigned by the Directory Server (DS) for 3D Secure authentication. Note: Specify this parameter when 3D Secure authentication is ... |
| `paymentMethod.paymentMethodMetaData.eci` | String, REQUIRED | max 2; values: 02,01,00 | Electronic Commerce Indicator (ECI) that is returned by the card scheme. This parameter is used to indicate the type of cardholder identity authentication. Valid values a... |
| `paymentMethod.paymentMethodMetaData.credentialType` | String | max 16; values: PAN,NETWORK_TOKEN | Indicates whether the Merchant Plug-In (MPI) information was obtained via the primary account number (PAN) or the network token. Valid values are: - `PAN`: Indicates that... |
| `paymentMethod.paymentMethodMetaData.requireIssuerAuthentication` | Boolean |  | Indicates whether Korean card transactions require authentication from the issuing bank. Valid values are: - `true`: indicates that Korean card transactions require authe... |
| `paymentMethod.paymentMethodMetaData.invoiceId` | String | max 20 | The unique ID to identify invoice files. Note: Please specify this parameter when the payment method is UPI, ONLINEBANKING\_NET BANKING, or Indian cards. |
| `paymentMethod.paymentMethodMetaData.requireIndianAuthentication` | Boolean |  | Indicates whether user information authorization and verification through the local Indian agency's page is required for payments using Indian cards. Valid values are: - ... |
| `paymentMethod.paymentMethodMetaData.funding` | String | max 32; values: CREDIT,DEBIT | The funding type of the card designated by the merchant for fund deductions. Valid values are: - `CREDIT`: Indicates a credit card. - `DEBIT`: Indicates a debit card. *... |
| `paymentMethod.paymentMethodMetaData.networktoken` | NetworkToken |  | The network token. |
| `paymentMethod.paymentMethodMetaData.networkTokenNo` | String | max 64 | The token number issued to the merchant by the card scheme. |
| `paymentMethod.paymentMethodMetaData.cryptogram` | String | max 128 | The cryptogram of the network token issued to the merchant by the card scheme. |
| `paymentMethod.paymentMethodMetaData.eci` | String | max 2 | Electronic Commerce Indicator (ECI) returned by the card scheme. |
| `paymentMethod.paymentMethodMetaData.expiryMonth` | String | max 2 | The expiration month of the network token that the merchant obtained from the card scheme. |
| `paymentMethod.paymentMethodMetaData.expiryYear` | String | max 2 | The expiration year of the network token that the merchant obtained from the card scheme. |
| `paymentMethod.paymentMethodMetaData.exemptionType` | String | max 64 | The 3D Secure exemption type specified by the merchant to request Strong Customer Authentication (SCA) exemption from the issuing bank. Once specified, Antom will pass th... |
| `paymentMethod.paymentMethodMetaData.requestIncrementalAuthorization` | Boolean |  | Indicates whether this pre-authorization request supports for incremental authorization. Valid values are: - `true`: This pre-authorization request supports for increment... |
| `paymentMethod.paymentMethodMetaData.requestExtendedAuthorization` | Boolean |  | Indicates whether this pre-authorization request supports extended authorization. Valid values are: - `true`: Indicates that this pre-authorization request supports exten... |
| `paymentMethod.paymentMethodMetaData.paymentType` | String | max 32; values: MOTO | Payment scenario type. Valid value is: - `MOTO`: Indicates a mail order or telephone order scenario. Notes: - This parameter is required in the MOTO scenario. >... |
| `paymentMethod.paymentMethodMetaData.bankIdentifierCode` | String |  | The unique code of the bank. See Bank list to check the valid values. Note: Specify this parameter when ... |
| `paymentMethod.paymentMethodMetaData.googlePayConfiguration` | GooglePayConfiguration |  | Google Pay payment preferences. |
| `paymentMethod.paymentMethodMetaData.googlePayToken` | String | max 4096 | Google pay token. Note: Specify this parameter when the merchant does not have PCI qualification. |
| `paymentMethod.paymentMethodMetaData.is3DSAuthentication` | Boolean |  | Indicates whether the transaction authentication type is 3D secure. Valid values are: - `true`: indicates that the transaction authentication type is 3D secure. - `false`... |
| `paymentMethod.paymentMethodMetaData.cardNo` | String | max 32 | The card number. Note: Specify this parameter when the merchant is PCI DSS compliant and has decrypted the googlePayToken. |
| `paymentMethod.paymentMethodMetaData.cvv` | String | max 3 | The card verification value (CVV), which is also known as a card security code (CSC) or a card veirification code (CVC). Note: Specify this parameter when the merch... |
| `paymentMethod.paymentMethodMetaData.expiryMonth` | String | max 2 | The month the card expires. Pass in two digits representing the month. For example, if the expiry month is February, the value of this parameter is `02`. Note: Spec... |
| `paymentMethod.paymentMethodMetaData.expiryYear` | String | max 2 | The year the card expires. Pass in the last two digits of the year number. For example, if the expiry year is 2025, the value of this parameter is `25`. Note: Speci... |
| `paymentMethod.paymentMethodMetaData.cardholderName` | UserName |  | The cardholder's name. Note: Specify this parameter when the merchant is PCI DSS compliant and has decrypted the googlePayToken. |
| `paymentMethod.paymentMethodMetaData.firstName` | String, REQUIRED | max 32 | The cardholder's first name. |
| `paymentMethod.paymentMethodMetaData.middleName` | String | max 32 | The cardholder's middle name. |
| `paymentMethod.paymentMethodMetaData.lastName` | String, REQUIRED | max 32 | The cardholder's last name. |
| `paymentMethod.paymentMethodMetaData.fullName` | String | max 32 | The cardholder's full name. |
| `paymentMethod.paymentMethodMetaData.billingAddress` | Address |  | The billing address that is associated with the cardholder's card account. Providing this information helps to increase the accuracy of anti-money laundering and fraud de... |
| `paymentMethod.paymentMethodMetaData.region` | String, REQUIRED | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. |
| `paymentMethod.paymentMethodMetaData.state` | String | max 8 | The state, country, or province name. |
| `paymentMethod.paymentMethodMetaData.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `paymentMethod.paymentMethodMetaData.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `paymentMethod.paymentMethodMetaData.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `paymentMethod.paymentMethodMetaData.zipCode` | String | max 32 | The ZIP or postal code. |
| `paymentMethod.paymentMethodMetaData.paymentMethodRegion` | String | max 6 | The region code that represents the country or region of the payment method. The value of this parameter is a 2-letter [ISO country code](https://www.iso.org/obp/ui/#sear... |
| `paymentMethod.paymentMethodMetaData.tokenizeMode` | String | max 64; values: ENABLED,DISABLED,ASKFORCONSENT | The tokenize mode. Indicates whether card information needs to be stored. When you opt to store card details, the actual information is replaced with a unique and secure ... |
| `paymentMethod.paymentMethodMetaData.isCardOnFile` | Boolean |  | Indicates whether the card used in this transaction has been previously processed and stored by the merchant. Valid values are: - `true`: The card used in the transaction... |
| `paymentMethod.paymentMethodMetaData.recurringType` | String | max 64; values: SCHEDULED,UNSCHEDULED | The recurring type for the payment. Valid values are: - `SCHEDULED`: indicates a regular automated deduction payment. - `UNSCHEDULED`: indicates an irregular automated de... |
| `paymentMethod.paymentMethodMetaData.networkTransactionId` | String | max 100 | This parameter is used for automated deduction payments. The value of this parameter is the value of the same field that is returned by [notifyPayment](https://docs.a... |
| `paymentMethod.paymentMethodMetaData.selectedCardBrand` | String | max 64 | The card brand that the user used to pay. See Card brands to check the valid values. |
| `paymentMethod.paymentMethodMetaData.enableAuthenticationUpgrade` | Boolean |  | Indicates whether you allow non-3D Secure authentication to be upgraded to 3D Secure authentication. Valid values are: - `true`: indicates that you allow non-3D Secure au... |
| `paymentMethod.paymentMethodMetaData.mpiData` | MpiData |  | The MPI (Merchant Plug-In) information. Note: Specify this parameter if you integrate external 3D MPI. |
| `paymentMethod.paymentMethodMetaData.threeDSVersion` | String, REQUIRED | max 16 | The version of 3D Secure protocol. Valid values are: - `1.0.2` - `2.1.0` - `2.2.0` |
| `paymentMethod.paymentMethodMetaData.cavv` | String | max 64 | The cardholder authentication value. The value of this parameter can be Cardholder Authentication Verification Value (CAVV) or Authentication Verification Value (AVV). ... |
| `paymentMethod.paymentMethodMetaData.dsTransactionId` | String | max 64 | The unique transaction identifier assigned by the Directory Server (DS) for 3D Secure authentication. Note: Specify this parameter when 3D Secure authentication is ... |
| `paymentMethod.paymentMethodMetaData.eci` | String, REQUIRED | max 2; values: 02,01,00 | Electronic Commerce Indicator (ECI) that is returned by the card scheme. This parameter is used to indicate the type of cardholder identity authentication. Valid values a... |
| `paymentMethod.paymentMethodMetaData.exemptionType` | String | max 64 | The 3D Secure exemption type specified by the merchant to request Strong Customer Authentication (SCA) exemption from the issuing bank. Once specified, Antom will pass th... |
| `paymentMethod.paymentMethodMetaData.interactionType` | String | max 11; values: QR,REDIRECT | Indicates the type of payment interaction flow used by the user. Valid values are: - `QR`: The user pays by scanning a QR code. - `REDIRECT`: The user pays via a redirect... |
| `paymentMethod.paymentMethodMetaData.storeName` | String | values: LAWSON,MINISTOP,SEICOMART,FAMILYMART | When the value of _paymentMethodType_ is `KONBINI`, specify this parameter to obtain a convenience store barcode, reducing or eliminating the need for barcode exchange at... |
| `paymentMethod.paymentMethodMetaData.payerPhoneNo` | String | max 24 | The user's phone number in E.164 format. For `MB_WAY`, the country code is +351, e.g., +351 12345678. Note: If this parameter is not specified, Antom will launch an... |
| `paymentMethod.paymentMethodMetaData.payerEmail` | Email | max 64 | The email address of the payer. Specify this parameter when all the following conditions are met: - The value of _paymentMethodType_ is `MERCADOPAGO_BR`, `MERCADOPAGO_MX`... |
| `paymentMethod.paymentMethodMetaData.cpf` | String | max 11 | Cadastro de Pessoas Físicas (CPF). The registration number of the Brazilian individual taxpayer. Specify this parameter when all the following conditions are met: -... |
| `paymentMethod.paymentMethodMetaData.cpf` | String | max 11 | Cadastro de Pessoas Físicas (CPF). The registration number of the Brazilian individual taxpayer. Specify this parameter when the value of _paymentMethodType_ is `PAGALE... |
| `paymentMethod.paymentMethodMetaData.payerEmail` | Email | max 64 | The email address of the payer. Specify this parameter when the value of _paymentMethodType_ is `PAGALEVE`. |
| `paymentMethod.paymentMethodMetaData.payerPhoneNo` | String | max 24 | The phone number of the payer. Specify this parameter when the value of _paymentMethodType_ is `PAGALEVE`. |
| `paymentMethod.paymentMethodMetaData.billingAddress` | Address |  | The billing address that is associated with the cardholder's card account. Specify this parameter when _paymentMethodType_ is `PAGALEVE`. Providing this information helps... |
| `paymentMethod.paymentMethodMetaData.region` | String, REQUIRED | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. |
| `paymentMethod.paymentMethodMetaData.state` | String | max 8 | The state, country, or province name. |
| `paymentMethod.paymentMethodMetaData.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `paymentMethod.paymentMethodMetaData.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `paymentMethod.paymentMethodMetaData.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `paymentMethod.paymentMethodMetaData.zipCode` | String | max 32 | The ZIP or postal code. |
| `paymentMethod.paymentMethodMetaData.payerName` | UserName |  | The payer's name. Specify this parameter when the value of _paymentMethodType_ is `PAGALEVE`. |
| `paymentMethod.paymentMethodMetaData.firstName` | String, REQUIRED | max 32 | The payer's first name. |
| `paymentMethod.paymentMethodMetaData.lastName` | String, REQUIRED | max 32 | The payer's last name. Scenario: Pay by Bank (Pay by Bank) |
| `paymentMethod.paymentMethodMetaData.bankIdentifierCode` | String |  | The unique code of the bank. See Bank list to check the valid values. Note: Specify this parameter when ... |
| `paymentMethod.paymentMethodMetaData.smartPaymentEnabled` | Boolean |  | Indicates whether integrated with PayPay Smart Payment. Valid values are: - `true`: Indicates that you have integrated with PayPay Smart Payment. - `false`: Indicates tha... |
| `paymentMethod.paymentMethodMetaData.bankIdentifierCode` | String | max 36 | The unique code of the bank. See Bank list to check the valid values. If this parameter is not specified, Anto... |
| `paymentMethod.paymentMethodMetaData.requirePixCode` | Boolean |  | Indicates whether a Pix QR code payment is required. Valid values are: - `true`: Set to `true` when you need to display a Pix code for the user to make a payment. - `fals... |
| `paymentMethod.paymentMethodMetaData.cpf` | String | max 11 | Cadastro de Pessoas Físicas (CPF). The registration number of the Brazilian individual taxpayer. Scenario: Przelewy24 (Przelewy24) |
| `paymentMethod.paymentMethodMetaData.payerEmail` | Email | max 64 | The email address of the payer. Specify this parameter when all the following conditions are met: - The value of _paymentMethodType_ is `P24`. - You collect this ... |
| `paymentMethod.paymentMethodMetaData.bankIdentifierCode` | String |  | The unique code of the bank. See Bank list to check the valid values. Specify this parameter when all t... |
| `paymentMethod.paymentMethodMetaData.channelOptionBill` | ChannelOptionBill |  | Configuration object for optional billing settings supported by the payment method. |
| `paymentMethod.paymentMethodMetaData.channelPromotionCode` | String | max 2080 | One or more promotion codes agreed between the merchant and seQura to identify promotional campaigns, such as interest-free installments. Multiple codes must be comma-sep... |
| `paymentMethod.paymentMethodMetaData.interactionType` | String | max 11; values: QR,REDIRECT | Indicates the type of payment interaction flow used by the user. Valid values are: - `QR`: The user pays by scanning a QR code. - `REDIRECT`: The user pays via a redirect... |
| `paymentMethod.customerId` | String | max 32 | The unique ID to identify a buyer. Note: Specify this parameter if you have the information. Providing this information helps to increase the accuracy of anti-money... |
| `paymentMethod.extendInfo` | String | max 2048 | Extended information. Note: Specify this parameter if you need to use the extended information. |
| `paymentFactor` | PaymentFactor |  | Factors that impact the payment. This parameter is used to indicate the payment scenario. Note: Specify this parameter when the value of _paymentMethodType_ is `CAR... |
| `paymentFactor.isAuthorization` | Boolean |  | Indicates whether the payment scenario requires authorization. Valid values are: - `false`: Default value. Indicates that the payment scenario is a regular payment withou... |
| `paymentFactor.captureMode` | String | max 64; values: AUTOMATIC,MANUAL | Indicates the method for capturing funds after the user authorizes the payment. Valid values are: - `AUTOMATIC`: indicates that Antom automatically captures the funds aft... |
| `creditPayPlan` | CreditPayPlan |  | The credit payment plan information for this payment. Note: Specify this field if you want to support installment payment and contact Antom technical support for de... |
| `creditPayPlan.installmentNum` | String, REQUIRED | max 8 | The number of installments. |
| `creditPayPlan.creditPayFeeType` | String |  | The value of this field is `PERCENTAGE`, which indicates that the fee is described in percentage with 1 - 3 digits. For example, 5 means 5%, 100 means 100%. Note: S... |
| `creditPayPlan.feePercentage` | Integer |  | This field is determined by the sellers' liability. For example, 0 indicates the seller is not liable for any fee. 100 indicates the seller pays 100% of the fee. Note... |
| `appId` | String | max 32 | The unique ID that is assigned by Antom to identify the mini program. Note: This field is required when _terminalType_ is `MINI_APP`. |
| `paymentExpiryTime` | Datetime |  | The payment expiration time is a specific time after which the payment will expire and the acquirer or merchant must terminate the order processing. Notes: - Fo... |
| `paymentNotifyUrl` | URL | max 2048 | The URL that is used to receive the result notification. Notes: - Payment result notification: Specify this parameter if you want to receive an asynchronous not... |
| `paymentRedirectUrl` | URL, REQUIRED | max 2048 | The merchant page URL that the user is redirected to after the payment is completed. Note: The maximum length of this parameter is 255 when the value of _paymentMth... |
| `productCode` | String, REQUIRED |  | Represents the payment product that is being used, which is stipulated in the contract. For One-time Payments, the value is fixed as `CASHIER_PAYMENT`. |
| `subscriptionInfo` | SubscriptionInfo |  | Subscription Information. Note: This parameter is required only for the first payment during subscription creation. |
| `subscriptionInfo.subscriptionDescription` | String, REQUIRED | max 256 | The description of the subscription, used for displaying user consumption records and other actions. |
| `subscriptionInfo.subscriptionStartTime` | Datetime, REQUIRED |  | The date and time when the subscription becomes active. - The value follows the ISO 8601 standard format. For ex... |
| `subscriptionInfo.subscriptionEndTime` | Datetime |  | The date and time when the subscription ends. Note: Specify this parameter when you want to designate the subscription end time. - The value follows the [ISO 8601](... |
| `subscriptionInfo.periodRule` | PeriodRule, REQUIRED |  | The subscription period rule, used to define a subscription's billing period. |
| `subscriptionInfo.periodRule.periodType` | String, REQUIRED | max 20; values: YEAR,MONTH,WEEK,DAY | The subscription period type. Valid values are: - `YEAR`: indicates that the subscription period is measured in years. - `MONTH`: indicates that the subscription period i... |
| `subscriptionInfo.periodRule.periodCount` | String, REQUIRED |  | The number of period types within one subscription period. If the value of _periodType_ is `MONTH` and the value of _periodCount_ is `2`, it means that the subscription p... |
| `subscriptionInfo.trials` | Array |  | List of subscription trial information. Note: Specify this parameter if the subscription includes any trial periods. |
| `subscriptionInfo.trials.trialStartPeriod` | String, REQUIRED |  | The start subscription period of the trial. For example, if the trial starts from the first subscription period, specify this parameter as `1`. - Value range: 1 - unlimit... |
| `subscriptionInfo.trials.trialAmount` | Amount, REQUIRED |  | The discounted amount for each subscription trial period. The value of this parameter must be a positive number. |
| `subscriptionInfo.trials.currency` | String, REQUIRED | max 3 | The transaction currency, which is a 3-letter currency code that follows the ISO 4217 standard. |
| `subscriptionInfo.trials.value` | String, REQUIRED | max 16 | The value of the amount as a positive integer in the smallest currency unit. If the currency is USD and the amount is $1.00, set the value of this parameter to `100`; or ... |
| `subscriptionInfo.subscriptionNotifyUrl` | String, REQUIRED | max 2048 | The URL that is used to receive the subscription result notification. You can also configure the subscription notification URL in Antom Dashboard. If you specify this URL... |
| `metadata` | String | max 2048 | Use this parameter to submit metadata, including information for special use cases. Specify this parameter when you need to add metadata information. |
| `locale` | String | max 8 | Language tag specified for the checkout page. Valid values are: - `en_US`: Default value. English - `pt_BR`: Portuguese (Brazil) - `pt_PT`: Portuguese - `es_ES`: Spanish ... |

#### Response fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `result` | Result, REQUIRED |  | The result of the API call. If this API is successfully called, the payment is successful. |
| `result.resultCode` | String, REQUIRED | max 64 | Result code. The result code that might be returned are listed in the Result/Error codes table on this page. |
| `result.resultStatus` | String, REQUIRED | values: S,F,U | Result status. Valid values are: - `S`: Indicates that the API call succeeds. - `F`: Indicates that the API call fails. - `U`: Indicates that the API call might be succes... |
| `result.resultMessage` | String, REQUIRED | max 256 | Result message that explains the result code. |
| `paymentRequestId` | String | max 64 | The unique ID that is assigned by a merchant to identify a payment request. Note: This field is returned when _resultCode_ is `PAYMENT_IN_PROCESS`. |
| `paymentId` | String | max 64 | The unique ID that is assigned by Antom to identify a payment. Note: This field is returned when _resultCode_ is `PAYMENT_IN_PROCESS`. |
| `paymentAmount` | Amount |  | The payment amount that the merchant requests to receive in the order currency. Note: This field is returned when _resultCode_ is `PAYMENT_IN_PROCESS`. |
| `paymentAmount.currency` | String, REQUIRED | max 3 | The 3-letter currency code that follows the ISO 4217 standard. |
| `paymentAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). Notes*... |
| `paymentData` | String | max 20000 | Used by the Antom client SDK to render the checkout page. This parameter is returned if the merchant app has integrated Antom client SDK. After receiving the parameter, y... |
| `paymentCreateTime` | Datetime |  | The date and time when the payment is created. Note: This field is returned when _resultCode_ is `PAYMENT_IN_PROCESS`. - The value follows the [ISO 8601](https://ww... |
| `pspCustomerInfo` | PspCustomerInfo |  | The customer information of digital wallets. Note: This field is returned when the digital wallets can provide the related information. |
| `pspCustomerInfo.pspName` | String | max 64 | The name of the digital wallets. Note: This field is returned when the digital wallets supports providing the related information. |
| `pspCustomerInfo.pspCustomerId` | String | max 64 | The customer ID of the digital wallets. Note: This field is returned when the digital wallets support providing the related information. |
| `pspCustomerInfo.displayCustomerId` | String | max 64 | The customer ID used for display. For example, _loginId_. Note: This field is returned when the digital wallets supports providing the related information. |
| `orderCodeForm` | OrderCodeForm |  | Information about the order code. Note: This parameter is returned when the payment method supports providing the related information. |
| `orderCodeForm.expireTime` | Datetime, REQUIRED |  | Expiry time of the order code information. - The value follows the ISO 8601 standard format. For example, "2019-... |
| `orderCodeForm.codeDetails` | Array, REQUIRED |  | Details about the code. - Maximum size: 4 elements |
| `orderCodeForm.codeDetails.codeValue` | String, REQUIRED | max 2048 | The value of the code. - If the value of _displayType_ is set to`SMALLIMAGE`, `MIDDLEIMAGE`, or `BIGIMAGE`, this parameter indicates a plain or Base64-encoded image URL. ... |
| `orderCodeForm.codeDetails.displayType` | String, REQUIRED | values: TEXT,MIDDLEIMAGE,SMALLIMAGE,BIGIMAGE | Indicates the way to display the code on the UI. Valid values are: - `TEXT`: Indicates that the code is directly displayed on the UI as text or a label. - `MIDDLEIMAGE`: ... |
| `orderCodeForm.extendInfo` | String | max 2048 | Extended information. Note: This field is returned when extended information exists. |
| `grossSettlementAmount` | Amount |  | The value of this field equals to transaction amount multiplied by the value of _settlementQuote_. This field is returned when the currency exchange is predetermined and ... |
| `grossSettlementAmount.currency` | String, REQUIRED | max 3 | The 3-letter currency code that follows the ISO 4217 standard. |
| `grossSettlementAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). Notes*... |
| `settlementQuote` | Quote |  | The exchange rate between the settlement currency and transaction currency. Note: This field is returned when _grossSettlementAmount_ is returned. |
| `settlementQuote.guaranteed` | Boolean |  | Guaranteed exchange rate available for payment. |
| `settlementQuote.quoteId` | String | max 64 | The unique ID that is assigned by Antom to identify an exchange rate. |
| `settlementQuote.quoteCurrencyPair` | String, REQUIRED | max 16 | A currency pair of settlement currency and transaction currency. The two currencies are separated with a slash and use the 3-letter [ISO-4217](https://www.iso.org/iso-421... |
| `settlementQuote.quotePrice` | Decimal, REQUIRED |  | The exchange rate used when a currency conversion between settlement currency and transaction currency occurs. - Value range: 0 - unlimited |
| `settlementQuote.quoteStartTime` | Datetime |  | The date and time when _quotePrice_ is effective. - The value follows the ISO 8601 standard format. For example,... |
| `settlementQuote.quoteExpiryTime` | Datetime |  | The date and time when _quotePrice_ expires. - The value follows the ISO 8601 standard format. For example, "201... |
| `appIdentifier` | String | max 128 | Android package name, which is used by Android apps to open a cashier page. Note: This field is returned when _resultCode_ is `PAYMENT_IN_PROCESS` and _terminalType... |
| `applinkUrl` | URL | max 2048 | The URL that redirects users to open an app when the target app is installed, or to open a WAP page when the target app is not installed. For Android, the URL is a Native... |
| `normalUrl` | URL | max 2048 | The URL that redirects users to a WAP or WEB page in the default browser or the embedded WebView. Note: When the value of _resultCode_ is `PAYMENT_IN_PROCESS`, at l... |
| `schemeUrl` | URL | max 2048 | The URL scheme that redirects users to open an app in an Android or iOS system when the target app is installed. Note: When the value of _resultCode_ is `PAYMENT_IN... |
| `paymentResultInfo` | PaymentResultInfo |  | The payment result information. Note: This parameter may be returned when the value of _paymentMethodType_ is `CARD`. |
| `paymentResultInfo.avsResultRaw` | String | max 128 | The raw AVS result. See AVS result codes to check the valid values. Note: This parameter may be returned whe... |
| `paymentResultInfo.cvvResultRaw` | String | max 128 | The raw Card Verification Value (CVV), Card Security Code (CSC), or Card Verification Code (CVC) result. See [CVV result codes](https://docs.antom.com/ac/card/risk_method... |
| `paymentResultInfo.networkTransactionId` | String | max 128 | The unique ID assigned by the card scheme to identify a transaction. The value of this parameter is used by the same parameter of [pay (One-time Payments)](https://do... |
| `paymentResultInfo.cardCategory` | string | max 32; values: CONSUMER,COMMERCIAL | The payment card category. Valid vaules are: - `CONSUMER`: the personal card - `COMMERCIAL`: the business card Note: This parameter is returned when all the followi... |
| `paymentResultInfo.cardholderName` | String | max 64 | The cardholder's name. Note: This parameter is returned when the value of _paymentMethodType_ in the [pay (One-time Payments)](https://docs.antom.com/ac/ams/pay... |
| `paymentResultInfo.cardBin` | String | max 8 | The first six digits of the bank card number, used to identify the issuing bank and card type of the bank card. Note: This parameter is returned when the value of _... |
| `paymentResultInfo.lastFour` | String | max 4 | Last 4 digits of the card number. Note: This parameter is returned when the value of _paymentMethodType_ in the [pay (One-time Payments)](https://docs.antom.com... |
| `paymentResultInfo.expiryMonth` | Datetime | max 2 | The month the card expires. Pass in two digits representing the month. For example, if the expiry month is February, the value of this parameter is `02`. Note: This... |
| `paymentResultInfo.expiryYear` | Datetime | max 2 | The year the card expires. Pass in the last two digits of the year number. For example, if the expiry year is 2025, the value of this parameter is `25`. Note: This ... |
| `paymentResultInfo.funding` | String | max 32; values: CREDIT,DEBIT,PREPAID,CHARGE,DEFERRED_DEBIT | The funding type of the card. Valid values are: - `CREDIT`: indicates a credit card. - `DEBIT`: indicates a debit card. - `PREPAID`: indicates a prepaid card. - `CHARGE`:... |
| `paymentResultInfo.refusalCodeRaw` | String | max 16 | The response code indicates an issuer’s reason for declining a transaction. Valid values see ISO codes. This parameter is re... |
| `paymentResultInfo.refusalReasonRaw` | String | max 256 | The error message that explains the response code. This parameter is returned when the following conditions are met: - The transaction fails. - The value of _paym... |
| `paymentResultInfo.merchantAdviceCode` | String | max 128 | The Merchant Advice Code (MAC) provides additional information about the type of Mastercard transaction or the reason why the payment failed. If the payment failed, the M... |
| `paymentResultInfo.exemptionRequested` | String | max 64 | The exemption type that Antom requested for in the payment authorization. Valid values are: - `lowValue`: Transactions below €30 may be exempt from Strong Customer Authen... |
| `paymentResultInfo.incrementalAuthorizationAvailable` | Boolean |  | Indicates whether incremental authorization is supported for this pre-authorization. Valid values are: - `true`: Indicates that incremental authorization is supported for... |
| `paymentResultInfo.extendedAuthorizationAvailable` | Boolean |  | Indicates whether extended authorization is supported for this pre-authorization. Valid values are: - `true`: Indicates that extended authorization is supported for this ... |
| `paymentResultInfo.refundOnAuthorizationAvailable` | Boolean |  | Indicates whether a refund can be initiated using _paymentId_ after the pre-authorization succeeds. Valid values are: - `true`: Indicates that subsequent refunds can be i... |
| `paymentResultInfo.fingerprint` | String | max 64 | Card fingerprint. A read-only parameter generated by the Antom system to uniquely identify the same card. The same bank card always generates the same _fingerprint_; a ca... |
| `promotionResult` | Array |  | Promotion result. Note: This parameter is generated when the buyer applies a promotion during the process of placing an order. |
| `promotionResult.promotionType` | String, REQUIRED | max 32 | Promotion type. Valid values are: `DISCOUNT`: a deduction from the regular price. `INTEREST_FREE`: payment in interest-free installments. |
| `promotionResult.discount` | Discount |  | Discount information. Note: This parameter is generated when the buyer applies a promotion during the process of placing an order. |
| `promotionResult.discount.discountName` | String | max 128 | The discount name displayed in the default language. |
| `promotionResult.discount.savingsAmount` | Amount |  | Discount amount. |
| `promotionResult.discount.value` | Integer, REQUIRED |  | The discount amount as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). Note:... |
| `promotionResult.discount.currency` | String, REQUIRED |  | A 3-letter currency code that follows the ISO 4217 standard. |
| `authExpiryTime` | Datetime |  | Pre-authorization expiration time. Notes: - This parameter is returned when the value of _result.resultStatus_ is `S`. - The capture must be initiated before ... |

## createPaymentSession (One-time Payments)

`POST /v1/payments/createPaymentSession`

On `U` or a timeout, change `paymentRequestId` and call again.

#### Request fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `merchantRegion` | String | max 2 | The country or region where the merchant operates the business. The parameter is a 2-letter country or region code that follows [ISO 3166 Country Codes](https://www.iso.o... |
| `env` | Env |  | Information about the environment where the order is placed. Note: Specify this parameter when you require risk control. Providing this information helps to increas... |
| `env.terminalType` | String | values: WEB,WAP,APP | Terminal type of which the merchant service applies to. Valid values are: - `WEB`: The client-side terminal type is a website, which is opened via a PC browser. - `WAP`: ... |
| `env.osType` | String | values: IOS,ANDROID | Type of the operation system. Valid values are: - `IOS`: indicates the operation system is Apple's iOS. - `ANDROID`: indicates the operation system is Google's Android. >... |
| `env.clientIp` | String | max 64 | The IP address of the client device. Note: Specify this parameter when you require risk control. Providing this information helps to increase the accuracy of anti-m... |
| `order` | Order, REQUIRED |  | The order information, such as buyer, merchant, goods, amount, shipping information, and purchase environment. This parameter is used for different purposes: - During the... |
| `order.orderAmount` | Amount, REQUIRED |  | The order amount of the merchant that directly provides services or goods to the customer. This parameter is used for user consumption records display or payment results ... |
| `order.orderAmount.currency` | String, REQUIRED | max 3 | The transaction currency that is specified in the contract. A 3-letter currency code that follows the ISO 4217 standar... |
| `order.orderAmount.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. If the currency is `USD` and the amount is $1.00, then set the value of this parameter to `100`.... |
| `order.referenceOrderId` | String, REQUIRED | max 64 | The unique ID to identify the order on the merchant side, which is assigned by the merchant that provides services or goods directly to the customer. This parameter is us... |
| `order.orderDescription` | String, REQUIRED | max 256 | Summary description of the order, which is used for user consumption records display or other further actions. |
| `order.goods` | Array |  | Goods information, including the ID, name, price, and quantity of the goods in the order. Note: Specify this parameter when one of the following conditions is met: ... |
| `order.goods.referenceGoodsId` | String, REQUIRED | max 64 | The unique ID to identify the goods. |
| `order.goods.goodsName` | String, REQUIRED | max 256 | Goods name. |
| `order.goods.goodsCategory` | String | max 64 | The category of the goods. If the goods have multiple layers for categorization, use slashes between different categories and write the parent category before the subcate... |
| `order.goods.goodsUnitAmount` | Amount |  | Price of goods. Note: Specify this parameter when one of the following conditions is met: - When you require risk control. - This parameter is required when t... |
| `order.goods.currency` | String, REQUIRED | max 3 | The 3-letter currency code that follows the ISO 4217 standard. |
| `order.goods.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). Note: Fo... |
| `order.goods.goodsQuantity` | Integer |  | Quantity of goods. Note: Specify this parameter when one of the following conditions is met: - When you require risk control. - This parameter is required whe... |
| `order.goods.goodsUrl` | String | max 2048 | The URL of the website where the user places an order. Note: Specify this parameter when one of the following conditions is met: - When you require risk control... |
| `order.goods.deliveryMethodType` | String | max 32; values: PHYSICAL,DIGITAL | The delivery method of the goods. Valid values are: - `PHYSICAL`: indicates that the delivery method is physical delivery. - `DIGITAL`: indicates that the delivery method... |
| `order.goods.priceId` | String | max 64 | The ID of the Price object. One of `priceId` or `referenceGoodsId` is required. |
| `order.goods.adjustableQuantity` | Object |  | The item’s quantity to be adjusted by the user during Checkout page. Note：Please specify this field when `priceId` has vaule. |
| `order.goods.maximum` | Integer |  | The maximum quantity the user can adjust during the Checkout page. - Value range: [0, 99999] |
| `order.goods.minimum` | Integer |  | The minimum quantity the user can adjust during the Checkout page. - Value range: [0, 1] |
| `order.shipping` | Shipping |  | Shipping information, including information about the recipient (name, phone number, email, and shipping address), and shipping service provider. Specify this parameter w... |
| `order.shipping.shippingName` | UserName |  | The name of the recipient. Specify this parameter when you require risk control. Providing this information helps to increase the accuracy of anti-money laundering and fr... |
| `order.shipping.firstName` | String, REQUIRED | max 32 | First name. |
| `order.shipping.middleName` | String | max 32 | Middle name |
| `order.shipping.lastName` | String, REQUIRED | max 32 | Last name |
| `order.shipping.fullName` | String | max 128 | Full name |
| `order.shipping.shippingAddress` | Address |  | Shipping address. Specify this parameter if you require risk control. Providing this information helps to increase the accuracy of anti-money laundering and fraud detecti... |
| `order.shipping.region` | String, REQUIRED | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. |
| `order.shipping.state` | String | max 8 | The state, country, or province name. Note: For card payments, if your business entity is in the United States, and the card issuing country is in Canada, the United St... |
| `order.shipping.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `order.shipping.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `order.shipping.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `order.shipping.zipCode` | String | max 32 | ZIP or postal code. For card payments, if your business entity is in the United States, specify this parameter according to the following parameter value requirements: ... |
| `order.shipping.shippingCarrier` | String | max 128 | The delivery service provider for shipping a physical product, such as FedEx, UPS, or USPS. Specify this parameter if you require risk control. Providing this information... |
| `order.shipping.shippingPhoneNo` | String | max 16 | The phone number of a recipient (including extension). Specify this parameter when you require risk control. Providing this information helps to increase the accuracy of ... |
| `order.shipping.shipToEmail` | String | max 64 | The email address where virtual goods are sent. Specify this parameter when one of the following conditions is met: - if you require risk control. - if you are a digital ... |
| `order.shipping.shippingFeeId` | String | max 64 | The ID of the shipping fee used for identifying the shipping option. |
| `order.shipping.shippingFee` | Object |  | The shipping fee. |
| `order.shipping.currency` | String, REQUIRED | max 3 | The 3-letter currency code that follows the ISO 4217 standard. |
| `order.shipping.value` | Integer, REQUIRED |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). - Value ra... |
| `order.shipping.shippingDescription` | String | max 64 | Extended information about logistics-related services, including shipping time, logistics companies, etc. |
| `order.shipping.deliveryEstimate` | Object |  | Estimated shipping time range. |
| `order.shipping.maximum` | Object |  | Estimated longest shipping time for logistics services. |
| `order.shipping.unit` | String, REQUIRED | max 16; values: YEAR,MONTH,DAY,HOUR,MINUTE,SECOND | Units for the longest shipping time of logistics services. The valid values include: - `YEAR`: Indicates that the shortest shipping time unit for logistics services is in... |
| `order.shipping.value` | Integer, REQUIRED |  | Estimated value for the longest shipping time of logistics services. - Value range: [0, +∞) |
| `order.shipping.minimum` | Object |  | Estimated shortest shipping time for logistics services. |
| `order.shipping.unit` | String, REQUIRED | max 16; values: YEAR,MONTH,DAY,HOUR,MINUTE,SECOND | Units for the shortest shipping time of logistics services. The valid values include: - `YEAR`: Indicates that the shortest shipping time unit for logistics services is i... |
| `order.shipping.value` | Integer |  | Estimated value for the shortest shipping time of logistics services. - Value range: [0, +∞) |
| `order.buyer` | Buyer |  | Buyer information, including the ID, name, phone number, and email of the buyer. Specify this parameter when one of the following conditions is met: - You require risk co... |
| `order.buyer.referenceBuyerId` | String | max 64 | The unique ID to identify the buyer. Specify this parameter: - When you require risk control. - When the value of _paymentMethodType_ is `CARD`. Providing this informatio... |
| `order.buyer.buyerName` | UserName |  | The name of the buyer. Specify this parameter when you require risk control. Providing this information helps to increase the accuracy of anti-money laundering and fraud ... |
| `order.buyer.firstName` | String, REQUIRED | max 32 | First name. |
| `order.buyer.middleName` | String | max 32 | Middle name |
| `order.buyer.lastName` | String, REQUIRED | max 32 | Last name |
| `order.buyer.fullName` | String | max 128 | Full name |
| `order.buyer.buyerPhoneNo` | String | max 24 | The mobile phone number of the buyer. Specify this parameter when one of the following conditions is met: - You require risk control. - The value of _paymentMethodType_ i... |
| `order.buyer.buyerEmail` | Email | max 64 | The email of the buyer. Specify this parameter: - When you require risk control. - When the value of _paymentMethodType_ is `CARD`. Providing this information helps to in... |
| `order.buyer.buyerRegistrationTime` | Datetime | max 64 | The time when the buyer registered your account. Specify this parameter if you require risk control. Providing this information helps to increase the accuracy of anti-mon... |
| `order.transit` | Transit |  | Trip information, including trip modes, legs of trips and passenger information. Specify this field if you require risk control. Providing this information helps to incre... |
| `order.transit.transitType` | String | values: FLIGHT,TRAIN,CRUISE,COACH | Trip modes, valid values are: - `FLIGHT`: indicates that the trip mode is by plane. - `TRAIN`: indicates that the trip mode is by train. - `CRUISE`: indicates that the tr... |
| `order.transit.agentCode` | String | max 8 | International Air Transport Association (IATA) code for the travel agency that made the reservation. |
| `order.transit.agentName` | String | max 32 | The name of the travel agency that made the reservation. |
| `order.transit.ticketNumber` | String | max 15 | A unique identifier for the ticket, typically formatted as IATA 3-letter airline code followed by a 10-digit number assigned by the airline. |
| `order.transit.ticketIssuerCode` | String | max 2 | Unique IATA 2-letter airline code identifying the organization or entity that issued the ticket. |
| `order.transit.restrictedTicketIndicator` | String | max 2; values: 0,1 | Indicates whether the ticket is refundable. Valid values include: - `0`: Indicates the ticket is refundable. - `1`: Indicates the ticket is not refundable. |
| `order.transit.legs` | Array |  | Information about sections of the trip, including departure time, arrival time, departure address, arrival address, transportation company name, carrier code and service ... |
| `order.transit.departureTime` | Datetime |  | Time of departure for this leg of the trip. - The value follows the ISO 8601 standard format. For example, "2019... |
| `order.transit.arrivalTime` | Datetime |  | Time of arrival for this leg of the trip. - The value follows the ISO 8601 standard format. For example, "2019-1... |
| `order.transit.departureAddress` | Address |  | Departure address for this leg of the trip. |
| `order.transit.region` | String | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Contry Codes standard. |
| `order.transit.state` | String | max 8 | The state, country, or province. |
| `order.transit.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `order.transit.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `order.transit.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `order.transit.arrivalAddress` | Address |  |  |
| `order.transit.region` | String | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. |
| `order.transit.state` | String | max 8 | The state, country, or province. |
| `order.transit.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `order.transit.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `order.transit.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `order.transit.carrierName` | String | max 128 | Company name of the transportation service provider for this leg of the trip. |
| `order.transit.classType` | String | max 32; values: FIRSTLEVEL,SECONDLEVEL,THIRDLEVEL | Service level for this leg of the trip. Valid values are: - `FIRSTLEVEL`: indicates that the service is the first class. For example, flight first class, cruise luxury se... |
| `order.transit.departureAirportCode` | String | max 3 | IATA code for the originating airport for this leg of the trip. |
| `order.transit.carrierNo` | String | max 2 | International Air Transport Association (IATA) code is obtained from the Official Airline Guide or its equivalent. |
| `order.transit.fareBasis` | String | max 16 | Code used to identify the basic rules of segment fares. This code is defined by the airline to identify ticket attributes, such as cabin class (e.g., business class), far... |
| `order.transit.couponNumber` | String | max 2 | An electronic coupon number used to identify a flight segment in a ticket. Each ticket may contain multiple electronic coupons, each corresponding to an independent fligh... |
| `order.transit.flightNumber` | String | max 5 | The identifier assigned to a specific flight. Typically 1 to 4 numeric digits, optionally followed by a suffix letter. |
| `order.transit.passengers` | Array |  | Information about the passenger of the trip, including the passenger names, passenger email and phone number. - Maximum size: 100 elements |
| `order.transit.passengerName` | Object |  | The name of the passenger. |
| `order.transit.firstName` | String | max 32 | First name. |
| `order.transit.middleName` | String | max 32 | Middle name. |
| `order.transit.lastName` | String | max 32 | Last name. |
| `order.transit.fullName` | String | max 128 | Full name. |
| `order.transit.passengerId` | String | max 64 | The unique ID that is assigned by merchant to identify a passenger. |
| `order.transit.passengerIdType` | String | max 64; values: PASSPORT,NATIONAL_ID_CARD,DRIVER_LICENSE,GREEN_CARD,TRAVEL_DOCUMENT,ALIEN_REGISTRATION_CARD,BIRTH_CERTIFICATE,OTHERS | The type of passenger's ID. Valid values are: - `PASSPORT` : Widely accepted for international and domestic travel. - `NATIONAL_ID_CARD`: National ID cards, which is vali... |
| `order.transit.passengerCode` | String | max 32 | The unique identifier assigned by the air carrier to a passenger for accurately identifying the passenger in the airline system. |
| `order.transit.ancillaryData` | Object |  | Information about an ancillary purchase. |
| `order.transit.connectedTicketNumber` | String | max 15 | The airline ticket number linked to this ancillary purchase. If the ancillary product is associated with a flight ticket (like a baggage fee), enter that flight ticket nu... |
| `order.transit.services` | Array |  | Information about the services included in an ancillary purchase. |
| `order.transit.categoryCode` | String | max 4 | Category code for the ancillary services offered. Retrieve the codes from the International Air Transport Association (IATA). |
| `order.transit.subCategoryCode` | String | max 4 | Subcategory code for the ancillary service category. Retrieve codes from the International Air Transport Association (IATA). |
| `order.lodging` | Lodging |  | Information about lodging service that was purchased, including hotel name, hotel address, check-in date, check-out date, number of booked rooms, number of booked nights ... |
| `order.lodging.hotelName` | String | max 128 | Hotel name. |
| `order.lodging.hotelAddress` | Address |  | Hotel address. |
| `order.lodging.region` | String | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Contry Codes standard. |
| `order.lodging.state` | String | max 8 | The state, country, or province. |
| `order.lodging.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `order.lodging.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `order.lodging.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `order.lodging.checkInDate` | Datetime |  | Date on which the guest checked in. In the case of a no-show or a reservation, the scheduled arrival date. - The value follows the [ISO 8601](https://www.iso.org/iso-8601... |
| `order.lodging.checkOutDate` | Datetime |  | Date on which the guest checked out. - The value follows the ISO 8601 standard format. For example, "2019-11-27T... |
| `order.lodging.numberOfNights` | Integer |  | Number of rooms booked by the payer. - Value range: 1 - unlimited |
| `order.lodging.numberOfRooms` | Integer |  | Number of nights booked by the payer. - Value range: 1 - unlimited |
| `order.lodging.guestName` | Array |  | Name of the guest under which the room is reserved. - Maximum size: 100 elements |
| `order.lodging.firstName` | String | max 32 | First name. |
| `order.lodging.middleName` | String | max 32 | Middle name. |
| `order.lodging.lastName` | String | max 32 | Last name. |
| `order.lodging.fullName` | String | max 128 | Full name. |
| `order.gaming` | Gaming |  | Information about gaming top-up, including game name, the topped up user name or ID, user email and user phone number. Specify this parameter if you require risk control.... |
| `order.gaming.gameName` | String | max 128 | Game name. |
| `order.gaming.toppedUpUser` | String | max 128 | Name or ID of the user whose gaming account is topped up. |
| `order.gaming.toppedUpEmail` | String | max 64 |  |
| `order.gaming.toppedUpPhoneNo` | String | max 32 |  |
| `order.declaration` | Declaration |  | Foreign exchange settlement declaration. |
| `order.declaration.declarationBizScene` | String | max 64; values: AIRLINE,HOTEL | Industry classification for the foreign exchange settlement declaration. Value valid are: - `AIRLINE`: Indicates that the declared industry classification is the airline ... |
| `order.declaration.declarationBeneficiaryId` | String | max 64 | The unique third-party beneficiary ID used for declaring cumulative amount. Note: This is a required parameter in OTA scenarios, and must be passed together with _decla... |
| `paymentRequestId` | String, REQUIRED | max 64 | The unique ID assigned by a merchant to identify a payment request. |
| `paymentAmount` | Amount, REQUIRED |  | The payment amount that the merchant requests to receive in the order currency. |
| `paymentAmount.currency` | String, REQUIRED | max 3 | The transaction currency, which is a 3-letter currency code that follows the ISO 4217 standard. |
| `paymentAmount.value` | Integer, REQUIRED |  | The value of the amount as a positive integer in the smallest currency unit. For example, if the currency is USD and the amount is $1.00, set the value of this parameter ... |
| `settlementStrategy` | SettlementStrategy, REQUIRED |  | The settlement strategy for the payment request. |
| `settlementStrategy.settlementCurrency` | String | max 3 | The ISO currency code of the currency that the merchant wants to be settled against. The field is required if the merchant signed up for multiple currencies to settle. |
| `paymentMethod` | PaymentMethod |  | The payment method that is used to collect the payment by the merchant or acquirer. |
| `paymentMethod.paymentMethodType` | String | max 64 | The payment method type that is included in payment method options. By specifying the value of this parameter, you can receive the cashier URL of the specified payment me... |
| `paymentMethod.paymentMethodId` | String | max 128 | The unique ID that is used to identify a payment method. The value of this parameter is that of _cardToken_ obtained from the [notifyPayment](https://docs.antom.com/a... |
| `paymentMethod.paymentMethodMetaData` | PaymentMethodMetaData |  | Additional information required for some specific payment methods. Note: Specify this parameter when the value of _paymentMethodType_ is `APPLEPAY`, `CARD`, `GOOGLE... |
| `paymentMethod.paymentMethodMetaData.applePayConfiguration` | ApplePayConfiguration |  | Apple Pay payment preferences. |
| `paymentMethod.paymentMethodMetaData.requiredBillingContactFields` | Enum |  | Billing information fields that you require from the user to fulfill the order. The valid values are: - `name` - `postalAddress` |
| `paymentMethod.paymentMethodMetaData.requiredShippingContactFields` | Enum |  | Shipping information fields that you require from the user to fulfill the order. Valid values are: - `email` - `name` - `phone` - `postalAddress` |
| `paymentMethod.paymentMethodMetaData.buttonsBundled` | Boolean |  | Indicate whether the Apple Pay button is combined with other payment methods. Valid values are: - `true`: Indicates that Apple Pay is bundled with other payment methods; ... |
| `paymentMethod.paymentMethodMetaData.is3DSAuthentication` | Boolean |  | Indicates whether the transaction authentication type is 3D secure. Valid values are: - `true`: indicates that the transaction authentication type is 3D secure. - `false`... |
| `paymentMethod.paymentMethodMetaData.billingAddress` | Address |  | The billing address that is associated with the cardholder's card account. |
| `paymentMethod.paymentMethodMetaData.region` | String, REQUIRED | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. |
| `paymentMethod.paymentMethodMetaData.state` | String | max 8 | The state, country, or province name. |
| `paymentMethod.paymentMethodMetaData.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `paymentMethod.paymentMethodMetaData.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `paymentMethod.paymentMethodMetaData.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `paymentMethod.paymentMethodMetaData.zipCode` | String | max 32 | ZIP or postal code. |
| `paymentMethod.paymentMethodMetaData.paymentMethodRegion` | String | max 6 | The region code that represents the country or region of the payment method. The value of this parameter is a 2-letter [ISO country code](https://www.iso.org/obp/ui/#sear... |
| `paymentMethod.paymentMethodMetaData.tokenize` |  |  | Deprecated. Use _tokenizeMode_ instead. |
| `paymentMethod.paymentMethodMetaData.tokenizeMode` | String | max 64; values: ENABLED,DISABLED,ASKFORCONSENT | The tokenize mode. Indicates whether card information needs to be stored. When you opt to store card details, the actual information is replaced with a unique and secure ... |
| `paymentMethod.paymentMethodMetaData.isCardOnFile` | Boolean |  | Indicates whether the card used in this transaction has been previously processed and stored by the merchant. Valid values are: - `true`: The card used in the transaction... |
| `paymentMethod.paymentMethodMetaData.recurringType` | String | max 64; values: SCHEDULED,UNSCHEDULED | The recurring type for the payment. Valid values are: - `SCHEDULED`: indicates a regular automated deduction payment. - `UNSCHEDULED`: indicates an irregular automated de... |
| `paymentMethod.paymentMethodMetaData.networkTransactionId` | String | max 100 | This parameter is used for automated deduction payments. The value of this parameter is the value of the same field that is returned by [notifyPayment](https://docs.a... |
| `paymentMethod.paymentMethodMetaData.selectedCardBrand` | String | max 64 | The card brand that the user used to pay. |
| `paymentMethod.paymentMethodMetaData.enableAuthenticationUpgrade` | Boolean |  | Indicates whether you allow non-3D Secure authentication to be upgraded to 3D Secure authentication. Valid values are: - `true`: indicates that you allow non-3D Secure au... |
| `paymentMethod.paymentMethodMetaData.mpiData` | MpiData |  | The MPI (Merchant Plug-In) information. Note: Specify this parameter if you integrate external 3D MPI. |
| `paymentMethod.paymentMethodMetaData.threeDSVersion` | String, REQUIRED | max 16 | The version of 3D Secure protocol. Valid values are: - `1.0.2` - `2.1.0` - `2.2.0` |
| `paymentMethod.paymentMethodMetaData.cavv` | String | max 64 | The cardholder authentication value. The value of this parameter can be Cardholder Authentication Verification Value (CAVV) or Authentication Verification Value (AVV). ... |
| `paymentMethod.paymentMethodMetaData.dsTransactionId` | String | max 64 | The unique transaction identifier assigned by the Directory Server (DS) for 3D Secure authentication. Note: Specify this parameter when 3D Secure authentication is ... |
| `paymentMethod.paymentMethodMetaData.eci` | String, REQUIRED | max 2; values: 02,01,00 | Electronic Commerce Indicator (ECI) that is returned by the card scheme. This parameter is used to indicate the type of cardholder identity authentication. Valid values a... |
| `paymentMethod.paymentMethodMetaData.is3DSAuthentication` | Boolean |  | Indicates whether the transaction authentication type is 3D secure. Valid values are: - `true`: indicates that the transaction authentication type is 3D secure. - `false`... |
| `paymentMethod.paymentMethodMetaData.cardNo` | String | max 32 | The card number. Specify this parameter when all the following conditions are met: - You have the PCI qualification. - The value of _paymentMethodType_ is `CARD`. - You c... |
| `paymentMethod.paymentMethodMetaData.cvv` | String | max 3 | The card verification value (CVV), which is also known as a card security code (CSC) or a card veirification code (CVC). Specify this parameter when all the following con... |
| `paymentMethod.paymentMethodMetaData.expiryYear` | String | max 2 | The year the card expires. Pass in the last two digits of the year number. For example, if the expiry year is 2025, the value of this parameter is `25`. Specify this para... |
| `paymentMethod.paymentMethodMetaData.expiryMonth` | String | max 2 | The month the card expires. Pass in two digits representing the month. For example, if the expiry month is February, the value of this parameter is `02`. Specify this par... |
| `paymentMethod.paymentMethodMetaData.cardholderName` | UserName |  | The cardholder's name. Note: Specify this parameter when all the following conditions are met: - You have the PCI qualification. - The value of _paymentMe... |
| `paymentMethod.paymentMethodMetaData.firstName` | String, REQUIRED | max 32 | The cardholder's first name. |
| `paymentMethod.paymentMethodMetaData.middleName` | String | max 32 | The cardholder's middle name. |
| `paymentMethod.paymentMethodMetaData.lastName` | String, REQUIRED | max 32 | The cardholder's last name. |
| `paymentMethod.paymentMethodMetaData.fullName` | String | max 128 | The cardholder's full name. |
| `paymentMethod.paymentMethodMetaData.billingAddress` | Address |  | The billing address that is associated with the cardholder's card account. Note: Specify this parameter when _paymentMethodType_ is `CARD` and you require risk cont... |
| `paymentMethod.paymentMethodMetaData.region` | String, REQUIRED | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. For card payments, if your busines... |
| `paymentMethod.paymentMethodMetaData.state` | String, REQUIRED | max 8 | The state, country, or province name. For card payments, if your business entity is in the United States, and the card issuing country is in Canada, the United States, ... |
| `paymentMethod.paymentMethodMetaData.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `paymentMethod.paymentMethodMetaData.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `paymentMethod.paymentMethodMetaData.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `paymentMethod.paymentMethodMetaData.zipCode` | String | max 32 | The ZIP or postal code. For card payments, if your business entity is in the United States, specify this parameter according to the following parameter value requiremen... |
| `paymentMethod.paymentMethodMetaData.cpf` | String | max 11 | The Cadastro Pessoal de Pessoa Física (CPF) is the tax ID of the Brazilian individual taxpayer. Note: Specify this parameter when all the following conditions a... |
| `paymentMethod.paymentMethodMetaData.dateOfBirth` | String | max 8 | The date of birth of the cardholder. The value of this parameter is an 8-digit date of birth in the format of `YYYY-MM-DD` that follows the [ISO 8601](https://www.iso.org... |
| `paymentMethod.paymentMethodMetaData.businessNo` | String | max 10 | The business number of the company that holds the corporate card. The value of this parameter is a 10-digit business number, such as `97XXXXXX11`. Note: Specify... |
| `paymentMethod.paymentMethodMetaData.cardPasswordDigest` | String | max 2 | The first two digits of the card payment password. Note: Specify this parameter when all the following conditions are met: - You have the PCI qualification.... |
| `paymentMethod.paymentMethodMetaData.paymentMethodRegion` | String | max 6 | The region code that represents the country or region of the payment method. The value of this parameter is a 2-letter [ISO country code](https://www.iso.org/obp/ui/#sear... |
| `paymentMethod.paymentMethodMetaData.payerEmail` | Email | max 64 | The email address of the payer. Note: Specify this parameter when all the following conditions are met: - _paymentMethodType_ is `CARD`. - _paymentMethodR... |
| `paymentMethod.paymentMethodMetaData.tokenizeMode` | String | max 64; values: ENABLED,DISABLED,ASKFORCONSENT | The tokenize mode. Indicates whether card information needs to be stored. When you opt to store card details, the actual information is replaced with a unique and secure ... |
| `paymentMethod.paymentMethodMetaData.isCardOnFile` | Boolean |  | Indicates whether the card used in this transaction has been previously processed and stored by the merchant. Valid values are: - `true`: The card used in the transaction... |
| `paymentMethod.paymentMethodMetaData.recurringType` | String | max 64; values: SCHEDULED,UNSCHEDULED | The recurring type for the payment. Valid values are: - `SCHEDULED`: indicates a regular automated deduction payment. - `UNSCHEDULED`: indicates an irregular automated de... |
| `paymentMethod.paymentMethodMetaData.networkTransactionId` | String | max 100 | This parameter is used for automated deduction payments. The value of this parameter is the value of the same field that is returned by [notifyPayment](https://docs.a... |
| `paymentMethod.paymentMethodMetaData.selectedCardBrand` | String | max 64 | The card brand that the user used to pay. See Card brands to check the valid values. Note: Specify this ... |
| `paymentMethod.paymentMethodMetaData.enableAuthenticationUpgrade` | Boolean |  | Indicates whether you allow non-3D Secure authentication to be upgraded to 3D Secure authentication. Valid values are: - `true`: indicates that you allow non-3D Secure au... |
| `paymentMethod.paymentMethodMetaData.mpiData` | MpiData |  | The MPI (Merchant Plug-In) information. Note: Specify this parameter if you integrate external 3D MPI. |
| `paymentMethod.paymentMethodMetaData.threeDSVersion` | String, REQUIRED | max 16 | The version of 3D Secure protocol. Valid values are: - `1.0.2` - `2.1.0` - `2.2.0` |
| `paymentMethod.paymentMethodMetaData.cavv` | String | max 64 | The cardholder authentication value. The value of this parameter can be Cardholder Authentication Verification Value (CAVV) or Authentication Verification Value (AVV). ... |
| `paymentMethod.paymentMethodMetaData.dsTransactionId` | String | max 64 | The unique transaction identifier assigned by the Directory Server (DS) for 3D Secure authentication. Note: Specify this parameter when 3D Secure authentication is ... |
| `paymentMethod.paymentMethodMetaData.eci` | String, REQUIRED | max 2; values: 02,01,00 | Electronic Commerce Indicator (ECI) that is returned by the card scheme. This parameter is used to indicate the type of cardholder identity authentication. Valid values a... |
| `paymentMethod.paymentMethodMetaData.requestIncrementalAuthorization` | Boolean |  | Indicates whether this pre-authorization request supports for incremental authorization. Valid values are: - `true`: This pre-authorization request supports for increment... |
| `paymentMethod.paymentMethodMetaData.requestExtendedAuthorization` | Boolean |  | Indicates whether this pre-authorization request supports extended authorization. Valid values are: - `true`: Indicates that this pre-authorization request supports exten... |
| `paymentMethod.paymentMethodMetaData.paymentType` | String | max 32; values: MOTO | Payment scenario type. Valid value is: - `MOTO`: Indicates a mail order or telephone order scenario. Notes: - This parameter is required in the MOTO scenario. >... |
| `paymentMethod.paymentMethodMetaData.googlePayConfiguration` | GooglePayConfiguration |  | Google Pay payment preferences. |
| `paymentMethod.paymentMethodMetaData.configuration` | Configuration |  | Basic merchant information. |
| `paymentMethod.paymentMethodMetaData.merchantName` | String | max 128 | The merchant name you want displayed in the payment sheet. |
| `paymentMethod.paymentMethodMetaData.merchantId` | String, REQUIRED | max 32 | The Google Pay ID that the merchant registers and activates on Google. When using the Antom payment sheet, the merchant does not need to register this ID separately. ... |
| `paymentMethod.paymentMethodMetaData.emailRequired` | Boolean |  | Indicates whether to collect the user's email information reserved in Google Pay. The default value is `false`. |
| `paymentMethod.paymentMethodMetaData.shippingAddressRequired` | Boolean |  | Indicates whether to collect the user's complete shipping address reserved in Google Pay. The default value is `false`. |
| `paymentMethod.paymentMethodMetaData.shippingAddressParameters` | ShippingAddressParameters |  | The additional information related to shipping. Note: When the value of _shippingAddressRequired_ is `true`, please specify this parameter. |
| `paymentMethod.paymentMethodMetaData.allowedCountryCodes` | String | max 100 | Logistics country code in accordance with ISO 3166-1 standard. If this field is not specified, all transportation address countries will be effective by default. |
| `paymentMethod.paymentMethodMetaData.phoneNumberRequired` | Boolean |  | Indicates whether the specified logistics shipping address requires collecting the user's phone number. The default value is `false`. |
| `paymentMethod.paymentMethodMetaData.allowedAuthMethods` | String | values: PAN_ONLY,CRYPTOGRAM_3DS | Specify the supported card transaction authenticate methods. Valid values are: - `PAN_ONLY`: This authentication method is associated with payment cards stored on file wi... |
| `paymentMethod.paymentMethodMetaData.billingAddressRequired` | Boolean |  | Indicate whether it is necessary to collect the billing address. The default value is `false`. Note: Additional data requests may lead to a decrease in payment conv... |
| `paymentMethod.paymentMethodMetaData.billingAddressParameters` | BillingAddressParameters |  | The additional information related to billing adress. |
| `paymentMethod.paymentMethodMetaData.format` | String | values: MIN,FULL | Billing address format required to complete the transaction. - `MIN`: Name, country code, and postal code (default). - `FULL`: Name, street address, locality, region, cou... |
| `paymentMethod.paymentMethodMetaData.buttonsBundled` | Boolean |  | Indicate whether the Google Pay button is combined with other payment methods. Valid values are: - `true`: Indicates that Google Pay is bundled with other payment methods... |
| `paymentMethod.paymentMethodMetaData.is3DSAuthentication` | Boolean |  | Indicates whether the transaction authentication type is 3D secure. Valid values are: - `true`: indicates that the transaction authentication type is 3D secure. - `false`... |
| `paymentMethod.paymentMethodMetaData.cardNo` | String | max 32 | The card number. Note: Specify this parameter when the merchant is PCI DSS compliant and has decrypted the googlePayToken. |
| `paymentMethod.paymentMethodMetaData.cvv` | String | max 3 | The card verification value (CVV), which is also known as a card security code (CSC) or a card veirification code (CVC). Note: Specify this parameter when the merch... |
| `paymentMethod.paymentMethodMetaData.expiryMonth` | String | max 2 | The month the card expires. Pass in two digits representing the month. For example, if the expiry month is February, the value of this parameter is `02`. Note: Spec... |
| `paymentMethod.paymentMethodMetaData.expiryYear` | String | max 2 | The year the card expires. Pass in the last two digits of the year number. For example, if the expiry year is 2025, the value of this parameter is `25`. Note: Speci... |
| `paymentMethod.paymentMethodMetaData.cardholderName` | UserName |  | The cardholder's name. Note: Specify this parameter when the merchant is PCI DSS compliant and has decrypted the googlePayToken. |
| `paymentMethod.paymentMethodMetaData.firstName` | String, REQUIRED | max 32 | The cardholder's first name. |
| `paymentMethod.paymentMethodMetaData.middleName` | String | max 32 | The cardholder's middle name. |
| `paymentMethod.paymentMethodMetaData.lastName` | String, REQUIRED | max 32 | The cardholder's last name. |
| `paymentMethod.paymentMethodMetaData.fullName` | String | max 128 | The cardholder's full name. |
| `paymentMethod.paymentMethodMetaData.billingAddress` | Address |  | The billing address that is associated with the cardholder's card account. |
| `paymentMethod.paymentMethodMetaData.region` | String, REQUIRED | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. |
| `paymentMethod.paymentMethodMetaData.state` | String | max 8 | The state, country, or province name. |
| `paymentMethod.paymentMethodMetaData.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `paymentMethod.paymentMethodMetaData.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `paymentMethod.paymentMethodMetaData.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `paymentMethod.paymentMethodMetaData.paymentMethodRegion` | String | max 6 | The region code that represents the country or region of the payment method. The value of this parameter is a 2-letter [ISO country code](https://www.iso.org/obp/ui/#sear... |
| `paymentMethod.paymentMethodMetaData.tokenizeMode` | String | max 64; values: ENABLED,DISABLED,ASKFORCONSENT | The tokenize mode. Indicates whether card information needs to be stored. When you opt to store card details, the actual information is replaced with a unique and secure ... |
| `paymentMethod.paymentMethodMetaData.isCardOnFile` | Boolean |  | Indicates whether the card used in this transaction has been previously processed and stored by the merchant. Valid values are: - `true`: The card used in the transaction... |
| `paymentMethod.paymentMethodMetaData.recurringType` | String | max 64; values: SCHEDULED,UNSCHEDULED | The recurring type for the payment. Valid values are: - `SCHEDULED`: indicates a regular automated deduction payment. - `UNSCHEDULED`: indicates an irregular automated de... |
| `paymentMethod.paymentMethodMetaData.networkTransactionId` | String | max 100 | This parameter is used for automated deduction payments. The value of this parameter is the value of the same field that is returned by [notifyPayment](https://docs.a... |
| `paymentMethod.paymentMethodMetaData.selectedCardBrand` | String | max 64 | The card brand that the user used to pay. |
| `paymentMethod.paymentMethodMetaData.enableAuthenticationUpgrade` | Boolean |  | Indicates whether you allow non-3D Secure authentication to be upgraded to 3D Secure authentication. Valid values are: - `true`: indicates that you allow non-3D Secure au... |
| `paymentMethod.paymentMethodMetaData.mpiData` | MpiData |  | The MPI (Merchant Plug-In) information. Note: Specify this parameter if you integrate external 3D MPI. |
| `paymentMethod.paymentMethodMetaData.threeDSVersion` | String, REQUIRED | max 16 | The version of 3D Secure protocol. Valid values are: - `1.0.2` - `2.1.0` - `2.2.0` |
| `paymentMethod.paymentMethodMetaData.cavv` | String | max 64 | The cardholder authentication value. The value of this parameter can be Cardholder Authentication Verification Value (CAVV) or Authentication Verification Value (AVV). ... |
| `paymentMethod.paymentMethodMetaData.dsTransactionId` | String | max 64 | The unique transaction identifier assigned by the Directory Server (DS) for 3D Secure authentication. Note: Specify this parameter when 3D Secure authentication is ... |
| `paymentMethod.paymentMethodMetaData.eci` | String, REQUIRED | max 2; values: 02,01,00 | Electronic Commerce Indicator (ECI) that is returned by the card scheme. This parameter is used to indicate the type of cardholder identity authentication. Valid values a... |
| `paymentMethod.paymentMethodMetaData.interactionType` | String | max 11; values: QR,REDIRECT | Indicates the type of payment interaction flow used by the user. Valid values are: - `QR`: The user pays by scanning a QR code. - `REDIRECT`: The user pays via a redirect... |
| `paymentMethod.paymentMethodMetaData.smartPaymentEnabled` | Boolean |  | Indicates whether integrated with PayPay Smart Payment. Valid values are: - `true`: Indicates that you have integrated with PayPay Smart Payment. - `false`: Indicates tha... |
| `paymentMethod.paymentMethodMetaData.interactionType` | String | max 11; values: QR,REDIRECT | Indicates the type of payment interaction flow used by the user. Valid values are: - `QR`: The user pays by scanning a QR code. - `REDIRECT`: The user pays via a redirect... |
| `savedPaymentMethods` | Array |  | Payment information stored by the user in the merchant system. |
| `savedPaymentMethods.paymentMethodType` | String | max 64 | The payment method type that is included in payment method options. |
| `savedPaymentMethods.paymentMethodId` | String | max 128 | The unique ID that is used to identify a payment method. |
| `paymentFactor` | PaymentFactor |  | Factors that impact the payment. This field is used to indicate the payment scenario. Specify this parameter when the value of _paymentMethodType_ is `CARD` and you integ... |
| `paymentFactor.isAuthorization` | Boolean |  | Indicates whether the payment scenario requires authorization. Valid values are: - `false`: Default value. Indicates that the payment scenario is a regular payment withou... |
| `paymentFactor.captureMode` | String | max 64; values: AUTOMATIC,MANUAL | Indicates the method for capturing funds after the user authorizes the payment. Valid values are: - `AUTOMATIC`: indicates that Antom automatically captures the funds aft... |
| `enableInstallmentCollection` | Boolean |  | Indicates whether Antom collects the installment information for the payment. Specify this parameter if you need Antom to collect the installment information. Valid value... |
| `creditPayPlan` | CreditPayPlan |  | The credit payment plan information for this payment. Specify this parameter when the payment supports installments and you specify the card-related information in _payme... |
| `creditPayPlan.installmentNum` | String, REQUIRED | max 8 | The number of installments. |
| `creditPayPlan.creditPayFeeType` | String |  | The value of this field is `PERCENTAGE`, which indicates that the fee is described in percentage with 1 - 3 digits. For example, 5 means 5%, 100 means 100%. Note: Specify... |
| `creditPayPlan.feePercentage` | Integer |  | This field is determined by the sellers' liability. For example, 0 indicates the seller is not liable for any fee. 100 indicates the seller pays 100% of the fee. Note: Sp... |
| `creditPayPlan.filteredInstallmentPeriods` | Array |  | The list of installment periods that the merchant allows the user to select. This parameter is used to filter the available installment payment plans displayed to the use... |
| `paymentSessionExpiryTime` | Datetime |  | The specific date and time after which the payment session will expire. The default expiration time is 1 hour after the session creation. For example, if the session is c... |
| `paymentNotifyUrl` | URL | max 2048 | The URL that is used to receive the payment result notification. Specify this parameter if you want to receive an asynchronous notification of the payment result. You can... |
| `paymentRedirectUrl` | URL, REQUIRED | max 2048 | The merchant page URL that the user is redirected to after the payment is completed. Note: The maximum length of this parameter is 255 when the value of _paymentMth... |
| `productCode` | String, REQUIRED |  | Represents the payment product that is being used, which is stipulated in the contract. For One-time Payments, the value is fixed as `CASHIER_PAYMENT`. |
| `locale` | String | max 8 | Language tag specified for the checkout page. Valid values are: - `en_US`: Default value. English - `pt_BR`: Portuguese (Brazil) - `pt_PT`: Portuguese - `es_ES`: Spanish ... |
| `availablePaymentMethod` | AvailablePaymentMethod |  | Information on payment methods available to the user. |
| `availablePaymentMethod.paymentMethodTypeList` | Array |  | List of available payment methods for the user. |
| `availablePaymentMethod.paymentMethodTypeList.paymentMethodType` | String |  | The payment method type that is included in payment method options. See Payment methods to check the valid values. -... |
| `availablePaymentMethod.paymentMethodTypeList.paymentMethodOrder` | Integer |  | The priority order of the payment methods configured by the user is indicated by numerical values, with smaller numbers representing higher priority. If the user does not... |
| `availablePaymentMethod.paymentMethodTypeList.expressCheckout` | Boolean |  | Indicates whether the payment method selected by the user is displayed as a quick payment method. The currently supported quick payment methods include `ALIPAY_CN`, `APPL... |
| `availablePaymentMethod.paymentMethodMetaData` | PaymentMethodMetaData |  | Additional information required for some specific payment methods. Scenario: Apple Pay (Apple Pay) |
| `availablePaymentMethod.paymentMethodMetaData.applePayConfiguration` | ApplePayConfiguration |  | Apple Pay payment preferences. |
| `availablePaymentMethod.paymentMethodMetaData.requiredBillingContactFields` | Enum |  | Billing information fields that you require from the user to fulfill the order. The valid values are: - `name` - `postalAddress` |
| `availablePaymentMethod.paymentMethodMetaData.requiredShippingContactFields` | Enum |  | Shipping information fields that you require from the user to fulfill the order. Valid values are: - `email` - `name` - `phone` - `postalAddress` |
| `availablePaymentMethod.paymentMethodMetaData.buttonsBundled` | Boolean |  | Indicate whether the Apple Pay button is combined with other payment methods. Valid values are: - `true`: Indicates that Apple Pay is bundled with other payment methods; ... |
| `availablePaymentMethod.paymentMethodMetaData.is3DSAuthentication` | Boolean |  | Indicates whether the transaction authentication type is 3D secure. Valid values are: - `true`: indicates that the transaction authentication type is 3D secure. - `false`... |
| `availablePaymentMethod.paymentMethodMetaData.billingAddress` | Address |  | The billing address that is associated with the cardholder's card account. |
| `availablePaymentMethod.paymentMethodMetaData.region` | String, REQUIRED | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. |
| `availablePaymentMethod.paymentMethodMetaData.state` | String | max 8 | The state, country, or province name. |
| `availablePaymentMethod.paymentMethodMetaData.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `availablePaymentMethod.paymentMethodMetaData.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `availablePaymentMethod.paymentMethodMetaData.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `availablePaymentMethod.paymentMethodMetaData.zipCode` | String | max 32 | ZIP or postal code. |
| `availablePaymentMethod.paymentMethodMetaData.paymentMethodRegion` | String | max 6 | The region code that represents the country or region of the payment method. The value of this parameter is a 2-letter [ISO country code](https://www.iso.org/obp/ui/#sear... |
| `availablePaymentMethod.paymentMethodMetaData.tokenizeMode` | String | max 64; values: ENABLED,DISABLED,ASKFORCONSENT | The tokenize mode. Indicates whether card information needs to be stored. When you opt to store card details, the actual information is replaced with a unique and secure ... |
| `availablePaymentMethod.paymentMethodMetaData.isCardOnFile` | Boolean |  | Indicates whether the card used in this transaction has been previously processed and stored by the merchant. Valid values are: - `true`: The card used in the transaction... |
| `availablePaymentMethod.paymentMethodMetaData.recurringType` | String | max 64; values: SCHEDULED,UNSCHEDULED | The recurring type for the payment. Valid values are: - `SCHEDULED`: indicates a regular automated deduction payment. - `UNSCHEDULED`: indicates an irregular automated de... |
| `availablePaymentMethod.paymentMethodMetaData.networkTransactionId` | String | max 100 | This parameter is used for automated deduction payments. The value of this parameter is the value of the same field that is returned by [notifyPayment](https://docs.a... |
| `availablePaymentMethod.paymentMethodMetaData.selectedCardBrand` | String | max 64 | The card brand that the user used to pay. |
| `availablePaymentMethod.paymentMethodMetaData.enableAuthenticationUpgrade` | Boolean |  | Indicates whether you allow non-3D Secure authentication to be upgraded to 3D Secure authentication. Valid values are: - `true`: indicates that you allow non-3D Secure au... |
| `availablePaymentMethod.paymentMethodMetaData.mpiData` | MpiData |  | The MPI (Merchant Plug-In) information. Note: Specify this parameter if you integrate external 3D MPI. |
| `availablePaymentMethod.paymentMethodMetaData.threeDSVersion` | String | max 16 | The version of 3D Secure protocol. Valid values are: - `1.0.2` - `2.1.0` - `2.2.0` |
| `availablePaymentMethod.paymentMethodMetaData.cavv` | String | max 64 | The cardholder authentication value. The value of this parameter can be Cardholder Authentication Verification Value (CAVV) or Authentication Verification Value (AVV). ... |
| `availablePaymentMethod.paymentMethodMetaData.dsTransactionId` | String | max 64 | The unique transaction identifier assigned by the Directory Server (DS) for 3D Secure authentication. Note: Specify this parameter when 3D Secure authentication is ... |
| `availablePaymentMethod.paymentMethodMetaData.eci` | String | max 2; values: 02,01,00 | Electronic Commerce Indicator (ECI) that is returned by the card scheme. This parameter is used to indicate the type of cardholder identity authentication. Valid values a... |
| `availablePaymentMethod.paymentMethodMetaData.is3DSAuthentication` | Boolean |  | Indicates whether the transaction authentication type is 3D secure. Specify this parameter when the value of _paymentMethodType_ is `CARD`. Valid values of this parameter... |
| `availablePaymentMethod.paymentMethodMetaData.cardNo` | String | max 32 | The card number. Specify this parameter when all the following conditions are met: - You have the PCI qualification. - The value of _paymentMethodType_ is `CARD`. - You c... |
| `availablePaymentMethod.paymentMethodMetaData.cvv` | String | max 3 | The card verification value (CVV), which is also known as a card security code (CSC) or a card veirification code (CVC). Specify this parameter when all the following con... |
| `availablePaymentMethod.paymentMethodMetaData.expiryYear` | String | max 2 | The year the card expires. Pass in the last two digits of the year number. For example, if the expiry year is 2025, the value of this parameter is `25`. Specify this para... |
| `availablePaymentMethod.paymentMethodMetaData.expiryMonth` | String | max 2 | The month the card expires. Pass in two digits representing the month. For example, if the expiry month is February, the value of this parameter is `02`. Specify this par... |
| `availablePaymentMethod.paymentMethodMetaData.cardholderName` | UserName |  | The cardholder's name. Specify this parameter when all the following conditions are met: - You have the PCI qualification. - The value of _paymentMethodType_ is `CARD`. -... |
| `availablePaymentMethod.paymentMethodMetaData.firstName` | String | max 32 | The cardholder's first name. |
| `availablePaymentMethod.paymentMethodMetaData.middleName` | String | max 32 | The cardholder's middle name. |
| `availablePaymentMethod.paymentMethodMetaData.lastName` | String | max 32 | The cardholder's last name. |
| `availablePaymentMethod.paymentMethodMetaData.fullName` | String | max 128 | The cardholder's full name. |
| `availablePaymentMethod.paymentMethodMetaData.billingAddress` | Address |  | The billing address that is associated with the cardholder's card account. Specify this parameter when _paymentMethodType_ is `CARD` and you require risk control. Providi... |
| `availablePaymentMethod.paymentMethodMetaData.region` | String, REQUIRED | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. |
| `availablePaymentMethod.paymentMethodMetaData.state` | String | max 8 | The state, country, or province name. |
| `availablePaymentMethod.paymentMethodMetaData.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `availablePaymentMethod.paymentMethodMetaData.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `availablePaymentMethod.paymentMethodMetaData.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `availablePaymentMethod.paymentMethodMetaData.zipCode` | String | max 32 | ZIP or postal code. |
| `availablePaymentMethod.paymentMethodMetaData.cpf` | String | max 11 | The Cadastro Pessoal de Pessoa Física (CPF) is the tax ID of the Brazilian individual taxpayer. Specify this parameter when all the following conditions are met: - When t... |
| `availablePaymentMethod.paymentMethodMetaData.dateOfBirth` | String | max 8 | The date of birth of the cardholder. The value of this parameter is an 8-digit date of birth in the format of `YYYY-MM-DD` that follows the [ISO 8601](https://www.iso.org... |
| `availablePaymentMethod.paymentMethodMetaData.businessNo` | String | max 10 | The business number of the company that holds the corporate card. The value of this parameter is a 10-digit business number, such as `97XXXXXX11`. Specify this parameter ... |
| `availablePaymentMethod.paymentMethodMetaData.cardPasswordDigest` | String | max 2 | The first two digits of the card payment password. Note: Specify this parameter when all the following conditions are met: - You have the PCI qualification.... |
| `availablePaymentMethod.paymentMethodMetaData.paymentMethodRegion` | String | max 6 | The region code that represents the country or region of the payment method. The value of this parameter is a 2-letter [ISO country code](https://www.iso.org/obp/ui/#sear... |
| `availablePaymentMethod.paymentMethodMetaData.payerEmail` | Email | max 64 | The email address of the payer. Note: Specify this parameter when all the following conditions are met: - _paymentMethodType_ is `CARD`. - _paymentMethodR... |
| `availablePaymentMethod.paymentMethodMetaData.tokenizeMode` | String | max 64; values: ENABLED,DISABLED,ASKFORCONSENT | The tokenize mode. Indicates whether card information needs to be stored. When you opt to store card details, the actual information is replaced with a unique and secure ... |
| `availablePaymentMethod.paymentMethodMetaData.isCardOnFile` | Boolean |  | Indicates whether the card used in this transaction has been previously processed and stored by the merchant. Valid values are: - `true`: The card used in the transaction... |
| `availablePaymentMethod.paymentMethodMetaData.recurringType` | String | max 64; values: SCHEDULED,UNSCHEDULED | The recurring type for the payment. Valid values are: - `SCHEDULED`: indicates a regular automated deduction payment. - `UNSCHEDULED`: indicates an irregular automated de... |
| `availablePaymentMethod.paymentMethodMetaData.networkTransactionId` | String | max 100 | This parameter is used for automated deduction payments. The value of this parameter is the value of the same field that is returned by [notifyPayment](https://docs.a... |
| `availablePaymentMethod.paymentMethodMetaData.selectedCardBrand` | String | max 64 | The card brand that the user used to pay. See Card brands to check the valid values. Note: Specify this ... |
| `availablePaymentMethod.paymentMethodMetaData.enableAuthenticationUpgrade` | Boolean |  | Indicates whether you allow non-3D Secure authentication to be upgraded to 3D Secure authentication. Valid values are: - `true`: indicates that you allow non-3D Secure au... |
| `availablePaymentMethod.paymentMethodMetaData.mpiData` | MpiData |  | The MPI (Merchant Plug-In) information. Note: Specify this parameter if you integrate external 3D MPI. |
| `availablePaymentMethod.paymentMethodMetaData.threeDSVersion` | String | max 16 | The version of 3D Secure protocol. Valid values are: - `1.0.2` - `2.1.0` - `2.2.0` |
| `availablePaymentMethod.paymentMethodMetaData.cavv` | String | max 64 | The cardholder authentication value. The value of this parameter can be Cardholder Authentication Verification Value (CAVV) or Authentication Verification Value (AVV). ... |
| `availablePaymentMethod.paymentMethodMetaData.dsTransactionId` | String | max 64 | The unique transaction identifier assigned by the Directory Server (DS) for 3D Secure authentication. Note: Specify this parameter when 3D Secure authentication is ... |
| `availablePaymentMethod.paymentMethodMetaData.eci` | String | max 2; values: 02,01,00 | Electronic Commerce Indicator (ECI) that is returned by the card scheme. This parameter is used to indicate the type of cardholder identity authentication. Valid values a... |
| `availablePaymentMethod.paymentMethodMetaData.googlePayConfiguration` | GooglePayConfiguration |  | Google Pay payment preferences. |
| `availablePaymentMethod.paymentMethodMetaData.configuration` | Configuration |  | Basic merchant information. |
| `availablePaymentMethod.paymentMethodMetaData.merchantName` | String | max 128 | The merchant name you want displayed in the payment sheet. |
| `availablePaymentMethod.paymentMethodMetaData.merchantId` | String | max 32 | The Google Pay ID that the merchant registers and activates on Google. When using the Antom payment sheet, the merchant does not need to register this ID separately. ... |
| `availablePaymentMethod.paymentMethodMetaData.emailRequired` | Boolean |  | Indicates whether to collect the user's email information reserved in Google Pay. The default value is `false`. |
| `availablePaymentMethod.paymentMethodMetaData.shippingAddressRequired` | Boolean |  | Indicates whether to collect the user's complete shipping address reserved in Google Pay. The default value is `false`. |
| `availablePaymentMethod.paymentMethodMetaData.shippingAddressParameters` | ShippingAddressParameters |  | The additional information related to shipping. Note: When the value of _shippingAddressRequired_ is `true`, please specify this parameter. |
| `availablePaymentMethod.paymentMethodMetaData.allowedCountryCodes` | String | max 100 | Logistics country code in accordance with ISO 3166-1 standard. If this field is not specified, all transportation address countries will be effective by default. |
| `availablePaymentMethod.paymentMethodMetaData.phoneNumberRequired` | Boolean |  | Indicates whether the specified logistics shipping address requires collecting the user's phone number. The default value is `false`. |
| `availablePaymentMethod.paymentMethodMetaData.allowedAuthMethods` | String | values: PAN_ONLY,CRYPTOGRAM_3DS | Specify the supported card transaction authenticate methods. Valid values are: - `PAN_ONLY`: This authentication method is associated with payment cards stored on file wi... |
| `availablePaymentMethod.paymentMethodMetaData.billingAddressRequired` | Boolean |  | Indicate whether it is necessary to collect the billing address. The default value is `false`. Note: Additional data requests may lead to a decrease in payment conv... |
| `availablePaymentMethod.paymentMethodMetaData.billingAddressParameters` | BillingAddressParameters |  | The additional information related to billing adress. |
| `availablePaymentMethod.paymentMethodMetaData.format` | String | values: MIN,FULL | Billing address format required to complete the transaction. - `MIN`: Name, country code, and postal code (default). - `FULL`: Name, street address, locality, region, cou... |
| `availablePaymentMethod.paymentMethodMetaData.phoneNumberRequired` | Boolean |  | Indicates whether it is necessary to collect the buyer's phone number. The default value is `false`. |
| `availablePaymentMethod.paymentMethodMetaData.buttonsBundled` | Boolean |  | Indicate whether the Google Pay button is combined with other payment methods. Valid values are: - `true`: Indicates that Google Pay is bundled with other payment methods... |
| `availablePaymentMethod.paymentMethodMetaData.is3DSAuthentication` | Boolean |  | Indicates whether the transaction authentication type is 3D secure. Valid values of this parameter are: - `true`: indicates that the transaction authentication type is 3D... |
| `availablePaymentMethod.paymentMethodMetaData.cardNo` | String | max 32 | The card number. Note: Specify this parameter when the merchant is PCI DSS compliant and has decrypted the googlePayToken. |
| `availablePaymentMethod.paymentMethodMetaData.cvv` | String | max 3 | The card verification value (CVV), which is also known as a card security code (CSC) or a card veirification code (CVC). Note: Specify this parameter when the merch... |
| `availablePaymentMethod.paymentMethodMetaData.expiryMonth` | String | max 2 | The month the card expires. Pass in two digits representing the month. For example, if the expiry month is February, the value of this parameter is `02`. Note: Spec... |
| `availablePaymentMethod.paymentMethodMetaData.expiryYear` | String | max 2 | The year the card expires. Pass in the last two digits of the year number. For example, if the expiry year is 2025, the value of this parameter is `25`. Note: Speci... |
| `availablePaymentMethod.paymentMethodMetaData.cardholderName` | UserName |  | The cardholder's name. Note: Specify this parameter when the merchant is PCI DSS compliant and has decrypted the googlePayToken. |
| `availablePaymentMethod.paymentMethodMetaData.firstName` | String | max 32 | The cardholder's first name. |
| `availablePaymentMethod.paymentMethodMetaData.middleName` | String | max 32 | The cardholder's middle name. |
| `availablePaymentMethod.paymentMethodMetaData.lastName` | String | max 32 | The cardholder's last name. |
| `availablePaymentMethod.paymentMethodMetaData.fullName` | String | max 32 | The cardholder's full name. |
| `availablePaymentMethod.paymentMethodMetaData.billingAddress` | Address |  | The billing address that is associated with the cardholder's card account. |
| `availablePaymentMethod.paymentMethodMetaData.region` | String, REQUIRED | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. |
| `availablePaymentMethod.paymentMethodMetaData.state` | String | max 8 | The state, country, or province name. |
| `availablePaymentMethod.paymentMethodMetaData.city` | String | max 32 | The city, district, suburb, town, or village name. |
| `availablePaymentMethod.paymentMethodMetaData.address1` | String | max 256 | Address line 1, for example, the street address, PO box, and company name. |
| `availablePaymentMethod.paymentMethodMetaData.address2` | String | max 256 | Address line 2, for example, the apartment, suite, unit, and building information. |
| `availablePaymentMethod.paymentMethodMetaData.paymentMethodRegion` | String | max 6 | The region code that represents the country or region of the payment method. The value of this parameter is a 2-letter [ISO country code](https://www.iso.org/obp/ui/#sear... |
| `availablePaymentMethod.paymentMethodMetaData.tokenizeMode` | String | max 64; values: ENABLED,DISABLED,ASKFORCONSENT | The tokenize mode. Indicates whether card information needs to be stored. When you opt to store card details, the actual information is replaced with a unique and secure ... |
| `availablePaymentMethod.paymentMethodMetaData.isCardOnFile` | Boolean |  | Indicates whether the card used in this transaction has been previously processed and stored by the merchant. Valid values are: - `true`: The card used in the transaction... |
| `availablePaymentMethod.paymentMethodMetaData.recurringType` | String | max 64; values: SCHEDULED,UNSCHEDULED | The recurring type for the payment. Valid values are: - `SCHEDULED`: indicates a regular automated deduction payment. - `UNSCHEDULED`: indicates an irregular automated de... |
| `availablePaymentMethod.paymentMethodMetaData.networkTransactionId` | String | max 100 | This parameter is used for automated deduction payments. The value of this parameter is the value of the same field that is returned by [notifyPayment](https://docs.a... |
| `availablePaymentMethod.paymentMethodMetaData.selectedCardBrand` | String | max 64 | The card brand that the user used to pay. |
| `availablePaymentMethod.paymentMethodMetaData.enableAuthenticationUpgrade` | Boolean |  | Indicates whether you allow non-3D Secure authentication to be upgraded to 3D Secure authentication. Valid values are: - `true`: indicates that you allow non-3D Secure au... |
| `availablePaymentMethod.paymentMethodMetaData.mpiData` | MpiData |  | The MPI (Merchant Plug-In) information. Note: Specify this parameter if you integrate external 3D MPI. |
| `availablePaymentMethod.paymentMethodMetaData.threeDSVersion` | String | max 16 | The version of 3D Secure protocol. Valid values are: - `1.0.2` - `2.1.0` - `2.2.0` |
| `availablePaymentMethod.paymentMethodMetaData.cavv` | String | max 64 | The cardholder authentication value. The value of this parameter can be Cardholder Authentication Verification Value (CAVV) or Authentication Verification Value (AVV). ... |
| `availablePaymentMethod.paymentMethodMetaData.dsTransactionId` | String | max 64 | The unique transaction identifier assigned by the Directory Server (DS) for 3D Secure authentication. Note: Specify this parameter when 3D Secure authentication is ... |
| `availablePaymentMethod.paymentMethodMetaData.eci` | String | max 2; values: 02,01,00 | Electronic Commerce Indicator (ECI) that is returned by the card scheme. This parameter is used to indicate the type of cardholder identity authentication. Valid values a... |
| `allowedPaymentMethodRegions` | Array | max 6 | A list of region codes that represent the countries or regions of payment methods. The value of this parameter is a 2-letter [ISO country code](https://www.iso.org/obp/ui... |
| `productScene` | String | max 32; values: CHECKOUT_PAYMENT,ELEMENT_PAYMENT | Specified product scenarios include valid values: - `CHECKOUT_PAYMENT`: Indicates that the merchant integrates using the Checkout Page. - `ELEMENT_PAYMENT`: Indicates tha... |
| `subscriptionInfo` | SubscriptionInfo |  | Subscription Information. Note: This parameter is required only for the first payment during subscription creation. |
| `subscriptionInfo.subscriptionDescription` | String, REQUIRED | max 256 | The description of the subscription, used for displaying user consumption records and other actions. |
| `subscriptionInfo.subscriptionStartTime` | Datetime, REQUIRED |  | The date and time when the subscription becomes active. - The value follows the ISO 8601 standard format. For ex... |
| `subscriptionInfo.subscriptionEndTime` | Datetime |  | The date and time when the subscription ends. Note: Specify this parameter when you want to designate the subscription end time. - The value follows the [ISO 8601](... |
| `subscriptionInfo.subscriptionExpiryTime` | Datetime |  | The expiration time of the subscription creation request. The user must complete the payment for the subscription before this time. After this time, payment is no longer ... |
| `subscriptionInfo.periodRule` | PeriodRule, REQUIRED |  | The subscription period rule, used to define a subscription's billing period. |
| `subscriptionInfo.periodRule.periodType` | String, REQUIRED | max 20; values: YEAR,MONTH,WEEK,DAY | The subscription period type. Valid values are: - `YEAR`: indicates that the subscription period is measured in years. - `MONTH`: indicates that the subscription period i... |
| `subscriptionInfo.periodRule.periodCount` | String, REQUIRED |  | The number of period types within one subscription period. If the value of _periodType_ is `MONTH` and the value of _periodCount_ is `2`, it means that the subscription p... |
| `subscriptionInfo.trials` | Array |  | List of subscription trial information. Note: Specify this parameter if the subscription includes any trial periods. |
| `subscriptionInfo.trials.trialStartPeriod` | String, REQUIRED |  | The start subscription period of the trial. For example, if the trial starts from the first subscription period, specify this parameter as `1`. - Value range: 1 - unlimit... |
| `subscriptionInfo.trials.trialAmount` | Amount, REQUIRED |  | The discounted amount for each subscription trial period. The value of this parameter must be a positive number. |
| `subscriptionInfo.trials.currency` | String, REQUIRED | max 3 | The transaction currency, which is a 3-letter currency code that follows the ISO 4217 standard. |
| `subscriptionInfo.trials.value` | String, REQUIRED | max 16 | The value of the amount as a positive integer in the smallest currency unit. If the currency is USD and the amount is $1.00, set the value of this parameter to `100`; or ... |
| `subscriptionInfo.subscriptionNotifyUrl` | String, REQUIRED | max 2048 | The URL that is used to receive the subscription result notification. You can also configure the subscription notification URL in Antom Dashboard. If you specify this URL... |
| `merchantAccountId` | String | max 64 | A unique ID to identify a specific merchant account. Note: Specify this parameter when you use one client ID across multiple locations. |
| `metadata` | String | max 2048 | This parameter is used for submitting metadata, including information for special use cases. Specify this parameter when you need to add metadata information. |

#### Response fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `result` | Result, REQUIRED |  | The result of the API call. If this API is successfully called, the payment session is created successfully. |
| `result.resultCode` | String, REQUIRED | max 64 | Result code. The result code that might be returned are listed in the Result/Error codes table on this page. |
| `result.resultStatus` | String, REQUIRED | values: S,F,U | Result status. Valid values are: - `S`: Indicates that the result status is successful. - `F`: Indicates that the result status is failed. - `U`: Indicates that the resul... |
| `result.resultMessage` | String, REQUIRED | max 256 | Result message that explains the result code. |
| `paymentSessionId` | String, REQUIRED | max 64 | The encrypted ID that is assigned by Antom to identify a payment session. |
| `paymentSessionData` | String, REQUIRED | max 4096 | The encrypted payment session data. Pass the data to your front end to initiate the client-side SDK. |
| `paymentSessionExpiryTime` | Datetime, REQUIRED |  | The specific date and time after which the payment session will expire. - The value follows the ISO 8601 standar... |
| `normalUrl` | String | max 2048 | The URL used to redirect to the Checkout Page. |

## consult (One-time Payments)

`POST /v1/payments/consult`

Use `productCode` `CASHIER_PAYMENT` and `paymentFactor.presentmentMode` `TILE`. Show a method when `paymentOptions.enabled` is `true`. For FPX the response lists banks in `paymentOptionDetail.supportBanks`.

#### Request fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `merchantRegion` | String | max 2 | The country or region where the merchant operates the business. The parameter is a 2-letter country or region code that follows [ISO 3166 Country Codes](https://www.iso.o... |
| `merchantAccountId` | String | max 32 | The unique ID to identify a merchant account. Note: Specify this parameter when you use a single client ID across multiple locations. |
| `userRegion` | String | max 2 | The 2-letter country or region code. For more information, see ISO 3166 Country Codes standard. The payment methods will be sorted b... |
| `allowedPaymentMethodRegions` | Array | max 6 | A list of region codes that represent the countries or regions of payment methods. The value of this parameter is a 2-letter [ISO country code](https://www.iso.org/obp/ui... |
| `buyer` | Buyer |  | Buyer information. |
| `buyer.referenceBuyerId` | String | max 64 | The unique ID to identify the buyer. Note: Specify this parameter when you want to use the promotion offered by Antom to this order. |
| `env` | Env, REQUIRED |  | Information about the environment where the order is placed, such as the device information. |
| `env.terminalType` | String, REQUIRED | values: WEB,WAP,APP,MINI_APP | Terminal type used by the merchant. Valid values are: - `WEB`: The terminal type of the merchant side is a website. - `WAP`: The terminal type of the merchant side is a m... |
| `env.osType` | String | values: IOS,ANDROID | Type of the operation system. Valid values are: - `IOS`: The OS type is iOS. - `ANDROID`: The OS type is Android. Note: This field is required for all _terminalType... |
| `env.userAgent` | String | max 1024 | The user agent. |
| `env.deviceTokenId` | String | max 64 | Token identifier of the device. |
| `env.clientIp` | String | max 32 | IP address of the client device. |
| `env.cookieId` | String | max 64 | The cookie ID of the buyer. |
| `paymentAmount` | Amount, REQUIRED |  | The payment amount that the merchant requests to receive in the order currency. |
| `paymentAmount.currency` | String, REQUIRED | max 3 | The 3-letter currency code that follows the ISO 4217 standard. |
| `paymentAmount.value` | Integer |  | The value of the amount as a positive integer in the smallest currency unit. For example, if the currency is USD and the amount is $1.00, set the value of this parameter ... |
| `settlementStrategy` | SettlementStrategy |  | The settlement strategy. |
| `settlementStrategy.settlementCurrency` | String | max 3 | The ISO currency code of the currency that the merchant wants to settle against. This field is required if the merchant signed up for multiple settlement currencies. Othe... |
| `paymentFactor` | PaymentFactor |  | Factors that impact the payment. This field is used to indicate the payment scenario. Note: Specify this parameter when the value of _paymentMethodType_ is `CARD`. |
| `paymentFactor.presentmentMode` | String | values: TILE,UNIFIED | The presentation mode chosen by the merchant to display the payment methods on the cashier page. Valid values are: - `TILE`: specify this presentation mode when presentin... |
| `productCode` | String, REQUIRED | max 32; values: CASHIER_PAYMENT | Represents the payment product that is being used. The payment product that can be used is stipulated in the contract. The fixed value is: - `CASHIER_PAYMENT`: Indicates ... |

#### Response fields

| Field | Type | Limits / values | Description |
| --- | --- | --- | --- |
| `result` | Result, REQUIRED |  | The request result that contains information such as status and error codes. Note: This field doesn't indicate the payment result. This field only indicates whether... |
| `result.resultCode` | String, REQUIRED | max 64 | The result code. |
| `result.resultStatus` | String, REQUIRED | values: S,F,U | The result status. Valid values are: - `S`: Indicates that the result status is successful. - `F`: Indicates that the result status is failed. - `U`: Indicates that the r... |
| `result.resultMessage` | String | max 256 | The result message. |
| `paymentOptions` | Array |  | The payment option list. |
| `paymentOptions.paymentMethodType` | String, REQUIRED | max 64 | The payment method type. See Payment methods to check the valid values. |
| `paymentOptions.paymentMethodCategory` | String | max 32; values: ALIPAY_PLUS,WALLET,CARD,ONLINE_BANKING,BANK_TRANSFER,OTC,MOBILE_BANKING_APP,BUY_NOW_PAY_LATER | The category of the payment method. Valid values are: - `ALIPAY_PLUS`: indicates that the payment method category is an Alipay+ payment method, including Alipay, AlipayHK... |
| `paymentOptions.paymentMethodRegion` | Array | max 6 | A list of region codes that represent the countries or regions of payment methods. The value of this parameter is a 2-letter [ISO country code](https://www.iso.org/obp/ui... |
| `paymentOptions.paymentOptionDetail` | PaymentOptionDetail |  | Details about the payment option. Note: This parameter is returned when one of the following conditions is met: - When the value of _paymentMethodType_ is `CARD... |
| `paymentOptions.paymentOptionDetail.supportCardBrands` | Array, REQUIRED |  | The list of supported card brands. Note: This parameter is returned when the value of _paymentMethodType_ is `CARD`. |
| `paymentOptions.paymentOptionDetail.cardBrand` | String, REQUIRED |  | The name of the card brand. See the Card brands to check the valid values. |
| `paymentOptions.paymentOptionDetail.logo` | Logo, REQUIRED |  | The logo of the card brand. |
| `paymentOptions.paymentOptionDetail.logoName` | String, REQUIRED | max 12 | The logo name of the card brand. See the Card brands to check the valid values. |
| `paymentOptions.paymentOptionDetail.logoUrl` | String | max 2048 | The logo URL of the card brand. |
| `paymentOptions.paymentOptionDetail.funding` | Array | values: CREDIT,DEBIT,PREPAID,CHARGE,DEFERRED_DEBIT | The funding type of the card. Valid values are: - `CREDIT`: indicates a credit card. - `DEBIT`: indicates a debit card. - `PREPAID`: indicates a prepaid card - `CHARGE`: ... |
| `paymentOptions.paymentOptionDetail.supportBanks` | Array , REQUIRED |  | The list of supported banks. This parameter is returned when the value of _paymentMethodType_ is `P24` or `ONLINEBANKING_FPX`. |
| `paymentOptions.paymentOptionDetail.bankIdentifierCode` | String |  | The unique code of the bank. See the Bank list to check the valid values. |
| `paymentOptions.paymentOptionDetail.bankShortName` | String |  | The short name of the bank. The unique code of the bank. See the Bank list to check the valid values. |
| `paymentOptions.paymentOptionDetail.bankLogo` | Logo |  | The logo information about the bank. Note: This parameter is returned when the value of _paymentMethodType_ is `ONLINEBANKING_FPX`. |
| `paymentOptions.paymentOptionDetail.logoName` | String, REQUIRED | max 32 | The logo name of the bank. |
| `paymentOptions.paymentOptionDetail.logoUrl` | String | max 2048 | The logo URL of the bank. |
| `paymentOptions.paymentOptionDetail.interactionTypes` | Array | values: QR,REDIRECT | Indicates the type of payment interaction flow used by the user. Valid values are: - `QR`: The user pays by scanning a QR code. - `REDIRECT`: The user pays via a redirect... |
| `paymentOptions.enabled` | Boolean, REQUIRED |  | Indicates whether the payment method is available. |
| `paymentOptions.disableReason` | String | max 64; values: PAYMENT_ACCOUNT_NOT_AVAILABLE,EXCEED_CHANNEL_LIMIT_RULE,SERVICE_DEGRADE,CHANNEL_NOT_SUPPORT_CURRENCY,CHANNEL_DISABLE,CHANNEL_NOT_IN_SERVICE_TIME,QUERY_IPP_INFO_FAILED,LIMIT_CENTER_ACCESS_FAIL,CURRENT_CHANNEL_NOT_EXIST | The reason why the payment method is not available. Valid values are: - `PAYMENT_ACCOUNT_NOT_AVAILABLE` - `EXCEED_CHANNEL_LIMIT_RULE` - `SERVICE_DEGRADE` - `CHANNEL_NOT_S... |
| `paymentOptions.logo` | Logo |  | The logo information about the payment method. |
| `paymentOptions.logo.logoName` | String, REQUIRED | max 32 | The logo name of the payment method. |
| `paymentOptions.logo.logoUrl` | URL | max 2048 | The logo URL of the payment method. |
| `paymentOptions.promoNames` | Array | max 512 | The list of the promotion names. In JSON format. The keys are returned as a language and a country code, connected by an underscore, such as `zh_CN`, while the value is t... |
| `paymentOptions.installment` | Installment |  | Information about installments. Note: This parameter is returned when the payment method supports installments. |
| `paymentOptions.installment.supportCardBrands` | Array |  | The list of card brands that support the installment plans. This parameter is returned when the value of _paymentMethodType_ is `CARD`. |
| `paymentOptions.installment.cardBrand` | String, REQUIRED | values: VISA,MASTERCARD,AMEX,HIPERCARD,ELO | The name of the card brand. Valid values are: - `VISA`: indicates Visa. - `MASTERCARD`: indicates Mastercard. - `AMEX`: indicates American Express (Amex). - `HIPERCARD`: ... |
| `paymentOptions.installment.logo` | Logo, REQUIRED |  | The logo of the card brand. |
| `paymentOptions.installment.logoName` | String, REQUIRED | max 12 | The logo name of the card brand. Valid values are: `VISA`, `MASTERCARD`, `AMEX`, `HIPERCARD`, and `ELO`. |
| `paymentOptions.installment.logoUrl` | String | max 2048 | The logo URL of the card brand. |
| `paymentOptions.installment.plans` | Array |  | The list of installment plans for payment methods that support installments. |
| `paymentOptions.installment.enabled` | Boolean, REQUIRED |  | Indicates whether the installment payment is available. |
| `paymentOptions.installment.interestRate` | String | max 8 | The interest rate the customer is charged on the installments. |
| `paymentOptions.installment.installmentNum` | String, REQUIRED | max 8 | The number of installment payments. The valid value is from `2` to `12`. |
| `paymentOptions.installment.interval` | String, REQUIRED | max 16 | The interval of each installment payment. The valid value is `MONTH`. |
| `paymentOptions.installment.minInstallmentAmount` | Amount, REQUIRED |  | The minimum amount that can be paid by installments. |
| `paymentOptions.installment.currency` | String, REQUIRED | max 3 | The 3-letter currency code that follows the ISO 4217 standard. |
| `paymentOptions.installment.value` | Integer |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). For detail... |
| `paymentOptions.installment.maxInstallmentAmount` | Amount, REQUIRED |  | The maximum amount that can be paid by installments. |
| `paymentOptions.installment.currency` | String, REQUIRED | max 3 | The 3-letter currency code that follows the ISO 4217 standard. |
| `paymentOptions.installment.value` | Integer |  | The amount to charge as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). For detail... |
| `paymentOptions.installment.supportedBank` | SupportedBank |  | Banks that support installment plans. Note: This parameter is returned when the bank supports installment plans. |
| `paymentOptions.installment.logo` | Logo, REQUIRED |  | Bank logo. |
| `paymentOptions.installment.logoName` | String, REQUIRED | max 32 | The name of the bank logo. |
| `paymentOptions.installment.logoUrl` | String, REQUIRED | max 1024 | The URL of the bank logo image. |
| `paymentOptions.installment.bankShortName` | String, REQUIRED | max 255 | The bank abbreviation, used to uniquely identify the bank. |
| `paymentOptions.installment.extendInfo` | String | max 4096 | Installment plan extension information. Used to return extension information related to the installment plan. Note: This parameter is returned only when the payment... |
| `paymentOptions.promotionInfos` | Array |  | Promotion information. This parameter is returned when the payment method offers a promotion to the buyer. |
| `paymentOptions.promotionInfos.promotionType` | String, REQUIRED | max 32 | Promotion type. Valid values are: `DISCOUNT`: a deduction from the regular price. `INTEREST_FREE`: payment in interest-free installments. |
| `paymentOptions.promotionInfos.discount` | Discount |  | Discount information. Note: This parameter is returned when the payment method offers a promotion to the buyer. |
| `paymentOptions.promotionInfos.discountTag` | String | max 64 | The discount tag is displayed as a superscript next to the discount name. |
| `paymentOptions.promotionInfos.discountName` | String | max 128 | The discount name displayed in the default language. |
| `paymentOptions.promotionInfos.estimateSavingsAmount` | Amount |  | The estimated discount amount. |
| `paymentOptions.promotionInfos.value` | Integer, REQUIRED |  | The discount amount as a positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). Note:... |
| `paymentOptions.promotionInfos.currency` | String, REQUIRED |  | A 3-letter currency code that follows the ISO 4217 standard. |
| `paymentOptions.promotionInfos.interestFree` | InterestFree |  | The information about interest-free installment plans. |
| `paymentOptions.promotionInfos.provider` | String, REQUIRED | max 128 | Issuing banks or financial institutions that offer interest-free installment plans. |
| `paymentOptions.promotionInfos.expireTime` | Datetime, REQUIRED |  | The promotion expiration time. - The value follows the ISO 8601 standard format. For example, "2019-11-27T12:01:... |
| `paymentOptions.promotionInfos.installmentFreeNums` | Array, REQUIRED |  | The number of interest-free installments. - Value range: 2-unlimited |
| `paymentOptions.promotionInfos.minPaymentAmount` | Amount, REQUIRED |  | The minimum payment amount required for the promotion. |
| `paymentOptions.promotionInfos.value` | Integer, REQUIRED |  | A positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). Note: For details about the ... |
| `paymentOptions.promotionInfos.currency` | String, REQUIRED |  | A 3-letter currency code that follows the ISO 4217 standard. |
| `paymentOptions.promotionInfos.maxPaymentAmount` | Amount |  | The maximum payment amount required for the promotion. The empty value indicates that there is no restriction on the maximum amount. |
| `paymentOptions.promotionInfos.value` | Integer, REQUIRED |  | A positive integer in the smallest currency unit. (That is, 100 cents to charge $1.00, or 100 to charge JPY 100, a 0-decimal currency). Note: For details about the ... |
| `paymentOptions.promotionInfos.currency` | String, REQUIRED |  | A 3-letter currency code that follows the ISO 4217 standard. |
| `paymentOptions.promotionInfos.freePercentage` | Integer, REQUIRED |  | The interest-free percentage. A value of 0 indicates no interest-free, 100% indicates the buyer is completely interest-free, while the value between 0-100% indicates part... |
