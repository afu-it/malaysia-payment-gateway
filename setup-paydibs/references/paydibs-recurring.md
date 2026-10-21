# Paydibs Recurring (CIT And MIT)

Checked 2026-10-21 against docs version 4.0. Sources:

- Initiation (CIT): `https://v3api-docs.paydibs.com/docs/recurring/initiation`
- Subsequent payments (MIT): `https://v3api-docs.paydibs.com/docs/recurring/subsequent-payments`
- Billing cycle rules: `https://v3api-docs.paydibs.com/docs/recurring/billing-cycle-rules`
- MIT Sign: `https://v3api-docs.paydibs.com/docs/sign-hashing/merchant-request-sign`, `.../merchant-response-sign`

Enum values (`Mode`, `Src`, `Bhv`, `BillCycTyp`, `Freq`, `WkDy`) are in `paydibs-codes.md`.

## Two Models

| Model | Fields | Who charges later |
| --- | --- | --- |
| Merchant-managed | `Mode=FLX`, `Src=MER` | You send `TxnType=MIT` with the stored `RcurId` |
| Paydibs-managed | `Mode=FIX`, `Src=PG` | Paydibs charges by the rule or fixed dates set in the CIT; you must not call MIT |

Note: the CIT field table heads the schedule fields with "For CIT Mode = FIX, SRC = MER, BillCycTyp = FD or RL use only ... for merchant who wish Paydibs to handle next recurring payment", while the process summary says Paydibs-managed is `Mode=FIX`, `Src=PG`. Use `FIX` + `PG` for Paydibs-managed and confirm with Paydibs.

## Step 1: CIT (first payment)

A normal PAY request (`paydibs-api.md`, PAY Request) to `/PPG/trigger` plus these fields. The PAY Sign formula is unchanged (recurring fields are not in the Sign).

Mandatory for CIT:

| Field | Type | Max | Notes |
| --- | --- | --- | --- |
| `Mode` | A | 50 | `FLX` or `FIX` |
| `Src` | A | 200 | `MER` or `PG` |
| `RcurIdc` | B | - | `true` marks a recurring transaction; then `CustReferenceId` is required |
| `CustReferenceId` | ANS (space) | 100 | Customer reference; must be provided when `RcurIdc=true` |

Conditional mandatory (Paydibs-managed only): `BillCycTyp` (A 50, `FD` fixed dates or `RL` rule).

Optional: `MaxCyc` (N, maximum number of charges; when reached the recurring becomes inactive; empty = indefinite), `Bhv` (A 50, `NONE` verify only or `IMDT` immediate charge + recurring; default immediate charge).

When `BillCycTyp=RL` (all optional unless marked):

| Field | Type | Notes |
| --- | --- | --- |
| `Freq` | A | Mandatory if `RL`. `HR`, `WKLY`, `MTLY`, `YRLY` |
| `Intrvl` | N | Every N units of `Freq`. Default `1` |
| `StDt` | ANS | `yyyy-MM-dd`, first eligible date. Must be at least the next day. If empty for rule, computed from today + rule |
| `EndDt` | ANS | `yyyy-MM-dd`, last eligible date (inclusive); omit for open-ended. Must be at least the next day |
| `DyOfMth` | N | `1` to `31`, with `MTLY` or `YRLY`. `31` = last valid day of shorter months |
| `WkDy` | N (values are text) | With `WKLY`. `MON` ... `SUN` (`THUR` for Thursday) |
| `MthOfYrs` | N | `1` (January) to `12` (December), with `YRLY` |
| `RcurAmt` | N 12(2) | Amount for later charges, if different from `MerchantTxnAmt`; otherwise later charges use `MerchantTxnAmt` |

When `BillCycTyp=FD`:

| Field | Type | Notes |
| --- | --- | --- |
| `FixDt` | ANS | Mandatory if `FD`. Comma-separated `yyyy-MM-dd` dates, max 30; one scheduler per date |
| `RcurAmt` | N 12(2) | As above |

CIT form sample (docs, merchant-managed):

```html
<form method="post" action="<PAYDIBS_BASE_URL>/PPG/trigger">
  <!-- all normal PAY fields, then: -->
  <input type="hidden" name="CustReferenceId" value="Cust123">
  <input type="hidden" name="Mode" value="FLX">
  <input type="hidden" name="Src" value="MER">
  <input type="hidden" name="MaxCyc" value="100">
  <input type="hidden" name="RcurIdc" value="true">
  <input type="hidden" name="Sign" value="<PAY Sign>">
  <input type="hidden" name="MerchantCallbackURL" value="https://merchant.example/callback">
</form>
```

## Step 2: CIT Response

The normal PAY response (RURL form post and callback JSON) plus `RcurId` (AN 200, system-generated recurring ID, e.g. `AGIDOM0000TestSG20180430009`). Only successful transactions create a recurring record; failed ones return no usable `RcurId`. Merchant-managed: store `RcurId` (required for MIT). Paydibs-managed: store it for reference.

## Billing Cycle Rules (`BillCycTyp=RL`)

The rules page uses descriptive names: `frequency` = `Freq`, `interval` = `Intrvl`, `day_of_month` = `DyOfMth`, `weekday` = `WkDy`, `month_of_year` = `MthOfYrs` (mapping by matching descriptions with the CIT table).

| Field | Hourly | Weekly | Monthly | Yearly | Note |
| --- | --- | --- | --- | --- | --- |
| frequency | yes | yes | yes | yes | Required |
| interval | yes | yes | yes | yes | Optional |
| day_of_month | no | no | yes | yes | |
| weekday | no | yes | no | no | |
| month_of_year | no | no | no | yes | Required if yearly |

Valid examples from the page: `{frequency: HR}`; `{HR, interval 24}`; `{WKLY, weekday MON}`; `{WKLY, interval 2, weekday FRI}`; `{MTLY, day_of_month 1}`; `{MTLY, interval 2, day_of_month 15}`; `{MTLY, day_of_month 31}`; `{YRLY, month_of_year 6, day_of_month 30}`; `{YRLY, interval 2, month_of_year 3, day_of_month 15}`.

Invalid: weekday with anything except weekly; yearly without month_of_year; day_of_month with hourly or weekly.

(The interval description also lists "every 30 minutes" as an example although no minute frequency exists. Do not rely on it.)

## Step 3: MIT (merchant-managed subsequent charge)

URL: `<payment URL>/PPG/trigger`. Server-side only (no browser). The docs sample is an HTTP request with query-string parameters; send it server-to-server.

| Field | Type | Max | Notes |
| --- | --- | --- | --- |
| `TxnType` | A | 3 | `MIT`, uppercase |
| `MerchantID` | AN | 6 | |
| `MerchantPymtID` | ANS (space) | 20 | New unique ID for this charge |
| `MerchantOrdID` | ANS (space) | 20 | Order/invoice no. |
| `MerchantOrdDesc` | ANS (space) | 100 | No Mandarin |
| `MerchantTxnAmt` | N | 12(2) | |
| `MerchantCurrCode` | A | 3 | |
| `MerchantCallbackURL` | ANS | 255 | Final status is posted here; HTTP 200 required, up to 5 retries; `&` to `;` |
| `RcurId` | AN | 200 | From the CIT response |
| `Sign` | ANS | 128 | MIT request Sign |

The docs sample also sends `CustName`, `CustPhone`, `CustEmail` (not in the table). Sending them is harmless.

MIT request Sign: `MerchantPassword + TxnType + MerchantID + MerchantPymtID + MerchantOrdID + MerchantTxnAmt + MerchantCurrCode + RcurId`. Doc example `abc123MITC001TEST1TESTORD110.00MYRAGID100` reproduces `a0d4000a7593751a...7e08ff6`.

MIT response (JSON, same session):

| Field | Type | Max | Notes |
| --- | --- | --- | --- |
| `MITSuccess` | B | 5 | `true` = accepted by the acquirer, not final. `false` = error |
| `TxnType` | A | 3 | `MIT` |
| `MerchantID` | AN | 6 | |
| `MerchantPymtID` | ANS (space) | 20 | |
| `MerchantOrdID` | ANS (space) | 20 | Echo |
| `MerchantTxnAmt` | N | 12(2) | Echo |
| `MerchantCurrCode` | A | 3 | Echo |
| `PTxnID` | AN | 30 | Paydibs transaction ID (absent on failure) |
| `RecurringID` | AN | 200 | Echo of `RcurId` (note different name) |
| `Sign` | ANS | 128 | MIT response Sign |
| `MitDesc` | ANS | - | Optional, error description |

The samples also contain `Method` (`MC` on success, empty on failure).

MIT response Sign: `MerchantPassword + MerchantID + MerchantPymtID + MitDesc + RecurringID`, skipping `MitDesc` when absent.

After `MITSuccess=true`, settle only from the QUERY result (`PTxnStatus=0`) or the verified callback. Failure sample: `"MITSuccess": false, "MitDesc": "Unable to proceed MIT recurring transaction. Recurring ID: ABC11."`.

## Not Documented (checked 2026-10-21)

- How to cancel or pause a recurring record (no recurring cancel API is published).
- Notifications for Paydibs-managed (`Src=PG`) scheduled charges other than the callback mechanism.
- Which channels support recurring (the CIT flow mentions saving card details).
