---
title: Givetel API Documentation
---

# Overview

## API Purpose

This API provides functionality to push leads to the Givetel dialler.

## API Endpoint Structure

The URI for the Givetel API structure is as follows:

- ` https://givetel-api.com/v2/[ENDPOINT NAME]`

## Rate Limiting

This API is subject to account-level rate limiting.

If you are being rate-limited, you will receive a response of `429 - Too Many Requests` with a `Retry-After` header that specifies the number of seconds to wait before making another request.

## Authentication

To maintain security, the API utilizes two stages of authentication/authorization.

There is a basic requirement for every api call to have an API key and a generated JSON Web Token in the `x-api-key` and `Authorization` header respectively.

The JWT generated lasts for 10 minutes and must be regenerated after that time.

## **Token Generation**

Endpoint: `https://givetel-api.com/v2/token`

### Methods

#### POST

Generate a new authentication token.

Has a default expiry of 10 minutes, a time that's defined in the `expires_at` key of the response, in UTC.

##### _Example Request_

```bash
curl -X POST https://givetel-api.com/v2/token -H "x-api-key: YOUR-API-KEY"
```

##### _Example Response_

```json
{
  "expires_at": "2023-08-07 01:51:48",
  "data": "this_is_a_long_jwt_value"
}
```

## **Leads**

Endpoint: `https://givetel-api.com/v2/leads`

### Methods

#### POST

Create a new lead in the Givetel dialler. Returns the `lead_id` of the newly created lead.

Please see [Create Lead Schema](#Create%20Lead%20Schema) for the full list of available fields for a lead. Please note the following fields are `required`:

- `phone`
- `supporter_firstname`
- `supporter_lastname`
- `external_reference_number`
- `charity` (nb: this is the charity acronym, not the full charity name.)
- `source`
- `call_type` (nb: currently `lead_conversion` is the only supported value.)

##### _Example Request_

```bash
curl -X POST https://givetel-api.com/v2/leads \
    -H "x-api-key: YOUR-API-KEY" \
    -H "Authorization: Bearer this_is_a_long_jwt_value" \
    --data '{
    "phone": "0400000000",
    "supporter_firstname": "Test",
    "supporter_lastname": "Supporter",
    "external_reference_number": "record-id-001",
    "charity": "TCA",
    "source": "example-acq-source",
    "call_type": "lead_conversion"
}'
```

##### _Example Response_

```json
{
  "lead_id": "nk7dxhv37hh488sxj040x97hjjl7yzgo",
  "charity": "TCA",
  "campaign_id": "24",
  "phone": "0400000000",
  "supporter_email": "",
  "rejection_reason": null,
  "rejected": false,
  "receive_again_after": "2024-08-01 02:05:04.250633",
  "provider_id": "6e14da48-888b-43c1-a750-bfe75d245b54",
  "partner_id": "clequ3gxt000meicp3cs1sv68",
  "data_source_id": "clepbkgqy001gs9q2eyaw82xk",
  "pledge_id": 117239210,
  "data": {
    "phone": "0400000000",
    "external_reference_number": "record-id-001",
    "supporter_firstname": "Test",
    "supporter_lastname": "Supporter",
    "source": "example-acq-source",
    "campaign_name": "Test Charity Lead Conversion",
    "lead_import_label": "Jesse Test API",
    "country": "AU",
    "supporter_timezone": "Sydney/Australia"
  }
}
```

### Create Lead Schema
[Scheme](../reference/givetel-openapi-spec.yaml/components/schemas/NewLead)

The following are all available fields for a lead being sent via the Givetel Lead API.

- `call_type`: string - must be exactly "lead_conversion" (**REQUIRED**)
- `charity`: string - the charity acronym ie "CFD" (**REQUIRED**)
- `supporter_firstname`: string (**REQUIRED**)
- `supporter_lastname`: string (**REQUIRED**)
- `external_reference_number`: string (**REQUIRED**)
- `phone`: string - ie "0411111111" (**REQUIRED**)
- `source`: string (**REQUIRED**)
- `source_date`: string - ISO-8601 (YYYY-MM-DD) date ie "2024-03-05". Should mark the date that the lead was generated. We auto-generate this if you don't have it.
- `source_time`: string - time ie "3:30pm". Should mark the time that the lead was generated. We auto-generate this if you don't have it.
- `supporter_title`: string
- `supporter_email`: string - ie "test@email.com"
- `mob_phone`: string
- `work_phone`: string
- `supporter_mob`: string
- `supporter_gender`: string
- `address1`: string
- `city`: string
- `state`: string - ie "NSW". Defaults to "NSW", if not passed. Appropriate calling hours are defined using the `state` field, so be sure to pass a valid Australian state acronym.
- `postcode`: string
- `country`: string (ISO 3166-1-formatted country code ie "AU") - defaults to "AU"
- `product`: string
- `source_campaign_name`: string
- `preference`: string
- `interest`: string
- `volunteered`: string
- `petition_signed`: string
- `petition_type`: string
- `appeal`: string
- `appeal_type`: string
- `event_participant`: string
- `event_participant_type`: string
- `campaigner`: string
- `start_date`: string - ISO-8601 (YYYY-MM-DD) date ie "2024-03-05". Should mark the date that a supporter's first donation will be made. Note this is only used for Welcome/Validation calls, and should not be supplied for `lead_conversion` leads.
- `additional_data`: json blob ie `{ "key": "value", "key2": "value2"}`. Should be used for objects containing metadata that doesn't fit into our schema. Note it is always preferred to use the native fields in the schema, where possible.
