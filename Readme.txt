The PoC contains 3 programs
1. Report ZAI_POD_EXTRACT2
2. Report ZAI_EXTRACT_DATA
3. Report ZOTC_CUST_DASHBOARD_HTML

ZAI_POD_EXTRACT2 is the main program that calls ZAI_EXTRACT_DATA and ZOTC_CUST_DASHBOARD_HTML.

What program does.... 

START

User selects input document file

Upload file from frontend
Convert binary file → XSTRING

Create document extraction object

Authenticate with BTP service
IF authentication successful

    Determine MIME type from file extension

    Send document to Document Information Extraction API
    Receive JOB_ID

    IF JOB_ID created
        Check job status repeatedly
        IF status = DONE
            Read extracted data (Header + Line Items)
            Display values
            Call ALV program with JOB_ID
        ELSE
            Handle failure
    ELSE
        Display "Job creation failed"

ELSE
    Display "Authentication failed"

END

What ALV Program does...

START

Input JOB_ID

Create DOX reader object
Authenticate with BTP

Read document extraction result using JOB_ID
    Parse JSON response
    Extract HEADER fields
    Extract LINE ITEM fields

Set display mode = HEADER
Load HEADER data to ALV
Open ALV screen

ALV Toolbar Buttons:
    HEADER  → show header fields
    ITEMS   → show line item fields
    CREATE_INV → create invoice
    SALES_DASH → open sales dashboard

IF CREATE_INV selected
    Get delivery number from HEADER
    Read item UoM from LIPS table
    Parse line item quantities
    Calculate invoice_qty = total_qty - reject_qty
    Prepare BAPI billing data
    Call BAPI_BILLINGDOC_CREATEMULTIPLE
    IF error
        ROLLBACK
        Show error
    ELSE
        COMMIT
        Display created invoice number in ALV

IF SALES_DASH selected
    Get delivery number from HEADER
    Submit sales dashboard program

END

What Sales dashboard program does....

START

Input Delivery Number (P_VBELN)

Find Customer Number from delivery (LIKP)

IF delivery not found
    Show error
ENDIF

--------------------------------------------------

Get Customer Sales Data

Count Sales Orders and total amount from VBAK

Count Deliveries from LIKP
Sum Delivered Quantity from LIPS

Count Invoices and total amount from VBRK
    (Invoice type = F2)

Calculate:
    Pending Billing Count = Deliveries – Invoices
    Pending Billing Amount = Sales Order Amount – Invoice Amount

--------------------------------------------------

Chart Preparation

Find maximum document count
Calculate scaling factor for chart bars

Calculate bar height and Y-position for:
    Sales Orders
    Deliveries
    Invoices

--------------------------------------------------

Build HTML Dashboard

Create tiles showing:
    Sales Orders (count + amount)
    Deliveries (count + quantity)
    Invoices (count)
    Pending Billing (count + amount)

Create SVG bar chart for:
    SO
    DEL
    INV

--------------------------------------------------

Display HTML Dashboard
    using ABAP Browser popup

END

