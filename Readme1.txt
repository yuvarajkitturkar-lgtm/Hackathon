This PoC contains a program and a configuration table

a. Include  ZSD_CREDIT_EVALUATION
b. Table ZAI_CREDIT

What program ZSD_CREDIT_EVALUATION does... 

START

Get customer credit configuration from ZAI_CREDIT
IF customer is excluded
    EXIT
ENDIF

--------------------------------------------------

Read Customer Financial Data

Select cleared items from BSAD
Select open items from BSID

--------------------------------------------------

Payment Behaviour Calculation

FOR each cleared invoice
    Calculate days to payment (AUGDT – BUDAT)
    Accumulate total days
    Count invoices
ENDFOR

FOR each open invoice
    Calculate overdue days (Today – Due Date)
    Accumulate total days
    Count invoices

    IF overdue
        Increase overdue count
        Track maximum overdue days
    ENDIF
ENDFOR

Calculate average days to pay

--------------------------------------------------

Credit Rating Logic

IF max overdue days > 90
    Rating = High Risk
ELSE IF max overdue days > 30
    Rating = Medium Risk
ELSE
    Rating = Low Risk
ENDIF

--------------------------------------------------

Credit Exposure Calculation

Sum all open items from BSID → Total Exposure

Get Credit Limit from KNKK
Get Customer Name from KNA1

--------------------------------------------------

Risk Classification

IF Exposure > Credit Limit
    Risk Color = Red
ELSE IF Exposure > 80% of Credit Limit
    Risk Color = Orange
ELSE
    Risk Color = Green
ENDIF

--------------------------------------------------

Create HTML Dashboard

Display tiles for:
    Customer
    Credit Limit
    Open Items (Exposure)
    Exposure Status
    Avg Days to Pay
    Credit Rating

--------------------------------------------------

Show dashboard in ABAP browser popup

Mark program as executed once

END


Configuration table - 

KUNNR	type KUNNR	CHAR	10	0	0	Customer Number
EXCLUDE	type ZAI_EXCLUDE	CHAR	1	0	0	Exclude all
LOW	type ZAI_LOW	CHAR	1	0	0	Exclude Low risk
MEDIUM	type ZAI_MEDIUM	CHAR	1	0	0	Exclude Medium Risk
HIGH	type ZAI_HIGH	CHAR	1	0	0	High Risk exclude
