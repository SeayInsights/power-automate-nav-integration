# AI Builder Document Processing — Worknote & Invoice OCR

## What AI Builder Document Processing Does

AI Builder document processing is a supervised machine learning service within the Microsoft Power Platform that extracts structured data from unstructured documents. You train a custom model by uploading sample documents and tagging the fields you want to extract. Once published, the model can be called from Power Automate flows to automatically extract fields such as work order numbers, dates, part numbers, and signatures from PDFs and images — with no manual data entry.

## Step 1: Create a Custom Model in AI Builder

1. Navigate to [make.powerapps.com](https://make.powerapps.com) and select the correct environment
2. In the left navigation, go to **AI Hub** → **Document Processing**
3. Click **+ New model** → **Document processing**
4. Name the model: `WorkNote-Invoice-Extractor`
5. Click **Create** to enter the model wizard

> **Tip:** Create the model in the same environment where your production flows run. Models are environment-specific and cannot be moved without retraining.

## Step 2: Upload Training Documents

AI Builder requires a minimum of 5 documents per document type. Recommended: 10–15 per type for production accuracy.

1. In the model wizard, click **+ Add documents**
2. Upload 10–15 sample worknotes (PDF or JPEG/PNG format)
3. Create a second collection and upload 10–15 sample supplier invoices
4. Label each collection (e.g., `Worknotes`, `SupplierInvoices`)

**Requirements for training documents:**
- Documents must be representative of real production documents (same font, layout, logos)
- Minimum resolution: 300 DPI for scanned documents
- Redact or replace any real customer names, addresses, or PII before uploading

## Step 3: Tag Fields for Extraction

For each document in your training set, draw bounding boxes around the field values you want to extract:

1. Open each training document in the tagging interface
2. Draw a bounding box around the value (not the label) for each field
3. Assign the bounding box to the corresponding field name from the field list
4. Repeat for all documents in the training collection

## Field Mapping Table

| Source Field | AI Builder Field Name | NAV Target Field | Notes |
|---|---|---|---|
| Work Order # | `WorkOrderNumber` | `No_` | Primary key — exact match required |
| Technician Name | `TechnicianName` | `Assigned_Technician` | Full name as on HR record |
| Completion Date | `CompletionDate` | `Finishing_Date` | Format: DD/MM/YYYY — normalize in flow |
| Parts Used | `PartsUsed` | `Item_List` | Comma-separated item numbers |
| Labor Hours | `LaborHours` | `Quantity` | Decimal value — e.g., 2.5 |
| Customer Signature | `CustomerSignature` | `Customer_Signed` | Presence of signature = true |
| Job Description | `JobDescription` | `Description` | Free text — truncate to 250 chars in flow |
| Site Address | `SiteAddress` | `Location_Code` | Map to NAV Location Code via lookup table in SharePoint |

## Step 4: Train and Evaluate the Model

1. After tagging all documents, click **Train**
2. Training typically takes 5–20 minutes
3. Review the **Confidence scores** per field once training completes:
   - **90%+** — Production ready
   - **70–89%** — Consider adding more training samples
   - **Below 70%** — Requires additional samples or layout standardization
4. Use the **Quick Test** panel to test against 2–3 documents not used in training
5. Review false positives and false negatives in the evaluation output

## Step 5: Publish and Use in Power Automate

1. Once confidence scores are satisfactory, click **Publish**
2. The model is now available in Power Automate via the **AI Builder** connector
3. In the flow, add action: **AI Builder → Extract information from documents**
4. Select your published model
5. Pass the document (from SharePoint, email attachment, or Power Apps upload) as the input
6. Map the extracted field outputs to subsequent flow actions

## Confidence Threshold Recommendations

Configure a Condition action after the AI Builder step to route based on confidence:

| Score Range | Routing | Action |
|---|---|---|
| >= 0.90 | Auto-process | Write directly to NAV |
| 0.70 – 0.89 | Supervisor review | Create SharePoint review task; notify supervisor via Teams |
| < 0.70 | Manual entry | Flag for manual data entry; notify submitting technician |

## Common Issues and Resolutions

| Issue | Cause | Resolution |
|---|---|---|
| Low confidence on `WorkOrderNumber` | Inconsistent font or position across documents | Request standardized worknote template from operations team; retrain with new samples |
| Date extracted in wrong format | Mixed date formats (DD/MM vs MM/DD) in training set | Normalize all training documents to one format before retraining; add `formatDateTime()` expression in flow |
| `PartsUsed` field truncated | Field spans multiple lines or columns | Tag the full multi-line region; or split into `Part1`, `Part2`, `Part3` fields |
| Model not visible in Power Automate | Model published in wrong environment | Verify the AI Builder model and the Power Automate flow are in the same Power Platform environment |
| Signature detection unreliable | Training documents had signature in varying positions | Add more training examples with signatures in different positions; use a Boolean presence check rather than coordinate-based detection |
