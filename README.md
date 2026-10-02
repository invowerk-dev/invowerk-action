# invowerk API — GitHub Action

Validate XRechnung, ZUGFeRD, Factur-X and Peppol BIS e-invoices in a workflow and fail the job when one is invalid.

Checks e-invoices from a GitHub Actions workflow. Validate takes an XML invoice (UBL or CII) or a ZUGFeRD / Factur-X PDF. It returns a JSON report with the result, each finding and the file's SHA-256. XRechnung goes through the KoSIT validator; PDFs are also checked for PDF/A. A jq expression on the report, such as .valid == false, fails the job. Parse, generate, render, convert and explain are inputs too. Invoices are processed in memory, not stored. 500 free credits per month.

Calls the [invowerk API](https://invowerk.dev) from a workflow: every operation, one step each. Jobs are waited for and their result is downloaded.

## Get an API key

[Create a key](https://invowerk.dev/go/gh-action?to=/app/api-keys) and store it as the repository secret `INVOWERK_API_KEY`. The key is optional: without `api-key` the action runs on the anonymous tier, with lower limits.

## Usage

```yaml
on: push
jobs:
  invowerk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7.0.1
      - uses: invowerk-dev/invowerk-action@v1.0.0
        with:
          operation: validate_invoice
          file: beispiel-xrechnung.xml
          api-key: ${{ secrets.INVOWERK_API_KEY }}
          fail-if: '.valid == false'
```

`fail-if: '.valid == false'` fails the build when `validate_invoice` finds a problem — the answer is still written to the `output` file.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `operation` | yes |  | The API operation to call: get_me, parse_invoice, convert_invoice, generate_invoice, explain_codes, render_invoice, validate_invoice. |
| `file` | no |  | Path of the file to upload, for operations that take one. |
| `json` | no |  | The request body as a JSON object, for operations that take one. |
| `query` | no |  | Path, query and form parameters, one key=value per line; repeat a key for a list. |
| `api-key` | no |  | Your invowerk API key, from a secret. Optional: without one the anonymous tier's lower limits apply; a key raises them. Create one at https://invowerk.dev/go/gh-action?to=/app/api-keys |
| `output` | no |  | Where to write the answer (default: a file in the runner's temp directory, named after the operation). |
| `fail-if` | no |  | A jq expression on a JSON answer; the step fails when it is true (e.g. `.valid == false`). |
| `base-url` | no | `https://api.invowerk.dev` | The API base URL. |

## Outputs

| Output | Description |
|---|---|
| `status` | The HTTP status of the last API call. |
| `output` | The path of the file holding the answer. |

## Operations

### `get_me`

Your plan, remaining requests, and remaining credits — `GET /v1/me`.

```yaml
- uses: invowerk-dev/invowerk-action@v1.0.0
  with:
    operation: get_me
    api-key: ${{ secrets.INVOWERK_API_KEY }}
```

### `parse_invoice`

Read an e-invoice into JSON — `POST /v1/parse`.
Costs 2 credits.

```yaml
- uses: invowerk-dev/invowerk-action@v1.0.0
  with:
    operation: parse_invoice
    file: beispiel-xrechnung.xml
    api-key: ${{ secrets.INVOWERK_API_KEY }}
```

### `convert_invoice`

Convert an e-invoice between UBL and CII — `POST /v1/convert`.
Costs 3 credits.

```yaml
- uses: invowerk-dev/invowerk-action@v1.0.0
  with:
    operation: convert_invoice
    file: beispiel-xrechnung.xml
    query: |
      target_syntax=cii
    api-key: ${{ secrets.INVOWERK_API_KEY }}
```

### `generate_invoice`

Create an e-invoice from JSON — `POST /v1/generate`.
Costs 5 credits.

```yaml
- uses: invowerk-dev/invowerk-action@v1.0.0
  with:
    operation: generate_invoice
    json: |
      {
        "buyer": {
          "address": {
            "city": "Munich",
            "country_code": "DE",
            "post_code": "80331"
          },
          "electronic_address": "buyer@example.com",
          "name": "Buyer AG"
        },
        "buyer_reference": "04011000-1234512345-06",
        "currency": "EUR",
        "due_date": "2026-02-15",
        "flavor": "xrechnung",
        "invoice_number": "INV-001",
        "issue_date": "2026-01-15",
        "lines": [
          {
            "line_net_amount": "100.00",
            "name": "Widget",
            "net_price": "100.00",
            "quantity": "1",
            "tax_category": "S",
            "tax_rate": "19.00"
          }
        ],
        "payment_iban": "DE75512108001245126199",
        "seller": {
          "address": {
            "city": "Berlin",
            "country_code": "DE",
            "post_code": "10115"
          },
          "contact_email": "accounts@example.com",
          "contact_name": "Accounts",
          "contact_phone": "+49 30 1234567",
          "electronic_address": "seller@example.com",
          "name": "Seller GmbH",
          "vat_id": "DE123456789"
        },
        "syntax": "ubl",
        "tax_breakdown": [
          {
            "category": "S",
            "rate": "19.00",
            "tax_amount": "19.00",
            "taxable_amount": "100.00"
          }
        ],
        "totals": {
          "line_total": "100.00",
          "tax_exclusive": "100.00",
          "tax_inclusive": "119.00"
        }
      }
    api-key: ${{ secrets.INVOWERK_API_KEY }}
```

### `explain_codes`

Explain an error code — `GET /v1/explain`.

```yaml
- uses: invowerk-dev/invowerk-action@v1.0.0
  with:
    operation: explain_codes
    query: |
      code=<code>
    api-key: ${{ secrets.INVOWERK_API_KEY }}
```

### `render_invoice`

Render an e-invoice as HTML — `POST /v1/render`.
Costs 1 credit.

```yaml
- uses: invowerk-dev/invowerk-action@v1.0.0
  with:
    operation: render_invoice
    api-key: ${{ secrets.INVOWERK_API_KEY }}
```

### `validate_invoice`

Check an e-invoice — `POST /v1/validate`.
Costs 1 credit.

```yaml
- uses: invowerk-dev/invowerk-action@v1.0.0
  with:
    operation: validate_invoice
    file: beispiel-xrechnung.xml
    api-key: ${{ secrets.INVOWERK_API_KEY }}
    fail-if: '.valid == false'
```

API reference: https://invowerk.dev/docs · Base URL: `https://api.invowerk.dev`

## Support

https://invowerk.dev/support

This repository is generated from the live API; changes to its files are overwritten.
