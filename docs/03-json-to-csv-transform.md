# JSON to CSV Transform — NAV OData to Power Automate

## Problem Statement

Microsoft Dynamics NAV OData v4 returns nested JSON with `@odata.context` metadata and a `value` array containing entity objects. Power Automate's native table actions — **Create CSV Table** and **Create HTML Table** — expect flat arrays of objects with simple string/number values. When NAV returns nested objects (such as line items embedded within a work order, or arrays within fields), the default behavior is to receive `[object Object]` in the CSV output instead of the actual field values.

This document describes the recommended flattening pattern: a **Select** action to reshape the response, followed by **Create CSV Table** or a **Join** action.

## The Transformation Pattern

```
NAV OData HTTP GET
    → Parse JSON (schema from NAV $metadata)
        → Select (map nested fields to flat object)
            → Create CSV Table
                → Send email / Save to SharePoint / Post to Teams
```

## Example NAV OData JSON Input

A typical response from `GET /ODataV4/Company('ACME')/WorkOrders?$top=2`:

```json
{
  "@odata.context": "https://nav.acme.com:7048/BC/ODataV4/$metadata#WorkOrders",
  "value": [
    {
      "No_": "WO-2025-001",
      "Description": "HVAC Service - Unit 4B",
      "Assigned_Technician": "John Smith",
      "Finishing_Date": "2025-06-15",
      "Quantity": 3.5,
      "Location_Code": "SITE-042",
      "Customer_Signed": true,
      "Line_Items": [
        { "Item_No": "FLT-001", "Qty": 2 },
        { "Item_No": "BLT-004", "Qty": 1 }
      ]
    },
    {
      "No_": "WO-2025-002",
      "Description": "Electrical Panel Inspection",
      "Assigned_Technician": "Maria Lopez",
      "Finishing_Date": "2025-06-16",
      "Quantity": 2.0,
      "Location_Code": "SITE-017",
      "Customer_Signed": false,
      "Line_Items": []
    }
  ]
}
```

## Power Automate Select Action Configuration

In the **Select** action:

- **From:** `body('Parse_JSON')?['value']`

Map the following key-value pairs:

| Output Column | Expression |
|---|---|
| `WorkOrder` | `item()?['No_']` |
| `Description` | `item()?['Description']` |
| `Technician` | `item()?['Assigned_Technician']` |
| `Date` | `formatDateTime(item()?['Finishing_Date'], 'dd/MM/yyyy')` |
| `Hours` | `string(item()?['Quantity'])` |
| `Site` | `item()?['Location_Code']` |
| `Signed` | `if(equals(item()?['Customer_Signed'], true), 'Yes', 'No')` |
| `LineItems` | `join(item()?['Line_Items'], '; ')` |

## Example CSV Output

After passing the Select output to **Create CSV Table**:

```csv
WorkOrder,Description,Technician,Date,Hours,Site,Signed,LineItems
WO-2025-001,HVAC Service - Unit 4B,John Smith,15/06/2025,3.5,SITE-042,Yes,"{ ""Item_No"": ""FLT-001""; { ""Item_No"": ""BLT-004"""
WO-2025-002,Electrical Panel Inspection,Maria Lopez,16/06/2025,2.0,SITE-017,No,
```

> **Note:** For nested arrays like `Line_Items`, the `join()` expression produces a string representation. If full line-level detail is needed in CSV, flatten via a separate loop before the Select action.

## Select + Join Pattern for Summary Rows

When the goal is a single summary row per work order (not one row per line item):

1. **Select** — flatten each work order to a summary object (as above)
2. **Create CSV Table** — standard action, input = Select output
3. **Join** — alternatively, use `join(outputs('Select'), '\n')` to produce newline-delimited JSON for downstream processing

## Edge Cases

| Scenario | Issue | Fix |
|---|---|---|
| Null field value | `null` appears as the string "null" in CSV | `if(equals(item()?['field'], null), '', item()?['field'])` |
| Comma in description field | CSV column boundary breaks | Wrap the field: `concat('"', replace(item()?['Description'], '"', '""'), '"')` |
| Date format inconsistency | NAV returns ISO 8601; report requires DD/MM/YYYY | `formatDateTime(item()?['Finishing_Date'], 'dd/MM/yyyy')` |
| Boolean fields | `true`/`false` are JSON booleans, not strings | `if(item()?['Customer_Signed'], 'Yes', 'No')` |
| Empty array fields | `join()` on empty array returns empty string | Handle with `if(empty(item()?['Line_Items']), 'None', join(...))` |
| Large response pagination | OData `@odata.nextLink` pagination not handled | Add a Do Until loop checking for `@odata.nextLink` and accumulating results |
| Special characters in text | Ampersands, quotes, angle brackets break XML/HTML output | Use `replace()` expressions to escape; or use CSV Table (not HTML Table) for raw data |
