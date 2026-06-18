# ethio-receipt-verify

Free, open-source Ethiopian bank and mobile-wallet receipt verification.

Why pay for a wrapper when the banks already publish the receipts publicly?

**Live site:** [cheki](https://cheki-pi.vercel.app) — web UI powered by this library.

## What this is

This repo reverse-engineers the public receipt endpoints used by paid
verification services (check.et, verify.et, etc.) and exposes them as a simple
Python library and CLI.

None of the supported endpoints require an API key, account, or subscription.
They are public URLs that return the official receipt as PDF, HTML, or JSON.

## Supported banks / wallets

| Code | Name | Status | Public endpoint |
|------|------|--------|-----------------|
| `cbe` | Commercial Bank of Ethiopia | works | `https://apps.cbe.com.et:100/?id=<REF><LAST_8>` |
| `telebirr` | Telebirr | works | `https://transactioninfo.ethiotelecom.et/receipt/<REF>` |
| `boa` | Bank of Abyssinia | works | `https://cs.bankofabyssinia.com/api/onlineSlip/getDetails/?id=<TRX>` |
| `mpesa` | M-Pesa Ethiopia | works | `https://m-pesabusiness.safaricom.et/api/receipt/getReceipt?trxNo=<REF>` |
| `zemen` | Zemen Bank | partial | PDF download works; parsing needs real receipt samples |
| `dashen` | Dashen Bank | partial | URL known; endpoint often returns "please try again later" |
| `awash` | Awash Bank | partial | URL pattern uncertain; port 8225 gives 403 |
| `cbebirr` | CBE Birr | partial | URL known; server currently times out |
| `siinqee` | Siinqee Bank | unsupported | No public receipt URL known |
| `kaafiebirr` | Kaafi Ebirr | unsupported | Public endpoint not found yet |

## Install

```bash
pip install git+https://github.com/1RB/ethio-receipt-verify.git
```

Or from source:

```bash
git clone https://github.com/1RB/ethio-receipt-verify.git
cd ethio-receipt-verify
pip install -e ".[dev]"
```

## CLI usage

```bash
# list supported banks
ethio-verify --list-banks

# CBE (requires full receiving account number)
ethio-verify cbe FT12345P01YB --account 10005XXXXXXXXXXX

# Telebirr (only needs the transaction number)
ethio-verify telebirr DET123456789

# Bank of Abyssinia (requires full receiving account)
ethio-verify boa AB123456789 --account 10005XXXXXXXXXXX

# M-Pesa
ethio-verify mpesa MP123456789

# JSON output
ethio-verify cbe FT12345P01YB --account 10005XXXXXXXXXXX --json
```

## Python usage

```python
from ethio_receipt_verify import verify

result = verify("cbe", "FT12345P01YB", account_number="10005XXXXXXXXXXX")
print(result.to_dict())
```

## How it works

Each verifier hits the same public receipt endpoint the paid services use, then
parses the response into a normalized `VerificationResult`.

### CBE

CBE concatenates the reference with the last 8 digits of the receiving account
and serves a PDF:

```
https://apps.cbe.com.et:100/?id=<REFERENCE><LAST_8_DIGITS>
```

### Telebirr

Telebirr publishes a full HTML receipt page by transaction number:

```
https://transactioninfo.ethiotelecom.et/receipt/<TRANSACTION_NUMBER>
```

### Bank of Abyssinia

BOA's receipt page is a React app that fetches JSON from a public T24 API:

```
https://cs.bankofabyssinia.com/api/onlineSlip/getDetails/?id=<TRX_ID>
```

where `<TRX_ID>` is the reference concatenated with the last 5 digits of the
receiver account.

### M-Pesa Ethiopia

Safaricom Ethiopia exposes a JSON receipt endpoint:

```
https://m-pesabusiness.safaricom.et/api/receipt/getReceipt?trxNo=<TRANSACTION_NUMBER>
```

## Disclaimers

- This tool only accesses **public** receipt endpoints. It does not bypass
  authentication, exploit vulnerabilities, or access private banking APIs.
- Receipts are public to anyone who knows the transaction reference (and in some
  cases a short account suffix). This is a design choice by the banks/wallets,
  not something introduced by this project.
- Use responsibly. Do not use this to harass, defraud, or violate anyone's
  privacy.
- The authors are not affiliated with any Ethiopian bank or wallet.
- Endpoints and response formats can change at any time; contributions and
  issue reports are welcome.

## License

MIT

## Contributing

Found a working endpoint for Awash, CBE Birr, Kaafi Ebirr, or Siinqee? Open a
PR with the source URL and a sample (sanitized) response.
