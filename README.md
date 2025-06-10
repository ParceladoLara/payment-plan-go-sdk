# Lara Payment Plan Go SDK

This is the Lara Payment Plan SDK, the core of the Lara Credit Proposal System.

## About

This SDK, like other Lara Payment Plan SDKs, is a wrapper around the Lara Payment Plan Rust library.
Currently, the library is available for Linux and Windows. MacOS support is planned for the future.

## Installation
To install the Lara Payment Plan Go SDK, you can use npm:

```bash
go get github.com/ParceladoLara/payment-plan-go-sdk/v3
```

## Usage
To use the Lara Payment Plan Go SDK, you can import it into your project as follows:

### CalculatePlan
Calculates a number of payment plans without down payment given a set of `Params`.
Returns a `Response[]` consisting of plans from 1 installment up to the number of installments requested,so that you can choose the one that best fits your needs, if the `InstallmentAmount` of the plan is less than the `MinInstallmentAmount` it will not be included in the response.

```go
import (
	"fmt"
	"time"

	payment_plan "github.com/ParceladoLara/payment-plan-go-sdk/v3"
)

params := payment_plan.Params{
	RequestedAmount:                7800,
	FirstPaymentDate:               time.Date(2025, 05, 3, 0, 0, 0, 0, time.FixedZone("-03", -3*60*60)),
	DisbursementDate:               time.Date(2025, 04, 5, 0, 0, 0, 0, time.FixedZone("-03", -3*60*60)),
	Installments:                   4,
	DebitServicePercentage:         0,
	Mdr:                            0.05,
	TacPercentage:                  0,
	IofOverall:                     0.0038,
	IofPercentage:                  0.000082,
	InterestRate:                   0.0235,
	MinInstallmentAmount:           100,
	MaxTotalAmount:                 1000000,
	DisbursementOnlyOnBusinessDays: true,
}

resp, err := payment_plan.CalculatePaymentPlan(params)

if err != nil {
	fmt.Println("Error calculating payment plan:", err)
} else {
  for _, plan := range resp {
    fmt.Printf("Plan: %+v\n", plan)
  }
}
```

Parameters:
- `MaxTotalAmount`: Maximum total amount for the plan.
- `MinInstallmentAmount`: Minimum installment amount.
- `RequestedAmount`: The amount requested for the plan.
- `FirstPaymentDate`: The date of the first payment.
- `DisbursementDate`: The date of the disbursement.
- `Installments`: The number of installments.
- `DebitServicePercentage`: Percentage for debit service.
- `Mdr`: Merchant Discount Rate.
- `TacPercentage`: Total Amount Charged percentage.
- `IofOverall`: Overall IOF (Tax on Financial Operations).
- `IofPercentage`: IOF percentage.
- `InterestRate`: Interest rate for the plan.
- `DisbursementOnlyOnBusinessDays`: Whether disbursement is only on business days.

Errors:
- If `RequestedAmount` is less than or equal to 0, an error will be thrown.
- If `Installments` is less than or equal to 0, an error will be thrown.
- If the any of the dates are invalid
- If any of the tax parameters are too unreasonable for a `Excel XIRR` calculation

### CalculateDownPaymentPlan
Calculates a number of payment plans with down payment given a set of `DownPaymentParams`.
Returns a `DownPaymentResponse[]` with the payment plan details, this will a plan for 1 installment up to the number of installments requested, so that you can choose the one that best fits your needs, if the `InstallmentAmount` of the plan is less than the `MinInstallmentAmount` it will not be included in the response.

```go
import (
  "fmt"
  "time"

  payment_plan "github.com/ParceladoLara/payment-plan-go-sdk/v3"
)

params := payment_plan.Params{
	RequestedAmount:                7800,
	FirstPaymentDate:               time.Date(2025, 05, 3, 0, 0, 0, 0, time.FixedZone("-03", -3*60*60)),
	DisbursementDate:               time.Date(2025, 04, 5, 0, 0, 0, 0, time.FixedZone("-03", -3*60*60)),
	Installments:                   4,
	DebitServicePercentage:         0,
	Mdr:                            0.05,
	TacPercentage:                  0,
	IofOverall:                     0.0038,
	IofPercentage:                  0.000082,
	InterestRate:                   0.0235,
	MinInstallmentAmount:           100,
	MaxTotalAmount:                 1000000,
	DisbursementOnlyOnBusinessDays: true,
}

downPaymentParams := payment_plan.DownPaymentParams{
	Params:               params,
	RequestedAmount:      1000,
	MinInstallmentAmount: 100,
	FirstPaymentDate:     time.Date(2025, 05, 3, 0, 0, 0, 0, time.FixedZone("-03", -3*60*60)),
	Installments:         4,
}

resp, err := payment_plan.CalculateDownPaymentPlan(downPaymentParams)
if err != nil {
  fmt.Println("Error calculating down payment plan:", err)
} else {
  for _, plan := range resp {
    fmt.Printf("Down Payment Plan: %+v\n", plan)
  }
}
```
Parameters:
- `RequestedAmount`: The amount requested for the down payment.
- `MinInstallmentAmount`: Minimum installment for the down payment plan.
- `Installments`: The number of installments for the down payment plan.
- `FirstPaymentDate`: The date of the first payment for the down payment plan.
- `Params`: A `Params` object containing the parameters for the payment plan(see above).

Errors:
- If `RequestedAmount` is less than or equal to 0, an error will be thrown.
- If `Installments` is less than or equal to 0, an error will be thrown.
- If the any of the dates are invalid
- If any of the tax parameters are too unreasonable for a `Excel XIRR` calculation

### DisbursementDateRange
Calculates a start and end date for a disbursement period based on a `baseDate` and a `numberOfDays`.

```go
import (
  "fmt"
  "time"

  payment_plan "github.com/ParceladoLara/payment-plan-go-sdk/v3"
)
baseDate := time.Date(2078, 02, 12, 0, 0, 0, 0, time.FixedZone("UTC", 0))
days := 5
startDate, endDate, err := payment_plan.DisbursementDateRange(baseDate, days)
fmt.Println("Start Date:", startDate) // 2078-02-16T03:00:00.000Z
fmt.Println("End Date:", endDate) // 2078-02-22T03:00:00.000Z
/*
  2078-02-12 = Saturday(invalid)
  2078-02-13 = Sunday(invalid)
  2078-02-14 = Bank holiday(invalid)
  2078-02-15 = Bank holiday(invalid)
  2078-02-16 = Wednesday(valid) 1
  2078-02-17 = Thursday(valid) 2
  2078-02-18 = Friday(valid) 3
  2078-02-19 = Saturday(invalid)
  2078-02-20 = Sunday(invalid)
  2078-02-21 = Tuesday(valid) 4
  2078-02-22 = Wednesday(valid) 5


  So the disbursement period is from 2078-02-16 to 2078-02-22.
*/
```
Parameters:
- `baseDate`: The base date from which to calculate the disbursement period.
- `numberOfDays`: The number of days to calculate the disbursement period.
- Returns an array where `[0]` is the start date and `[1]` is the end date of the disbursement period.

### GetNonBusinessDaysBetween
Calculates the non-business(weekends and bank holidays) days between two dates, and returns an array of non-business days.

```go
import {
  "fmt"
  "time"

  payment_plan "github.com/ParceladoLara/payment-plan-go-sdk/v3"
}
startDate := time.Date(2078, 11, 12, 0, 0, 0, 0, time.FixedZone("UTC", 0));
endDate := time.Date(2078, 11, 22, 0, 0, 0, 0, time.FixedZone("UTC", 0));
result := payment_plan.GetNonBusinessDaysBetween(startDate, endDate);
fmt.Println("Non-business days:", result); // [2078-11-12T03:00:00.000Z, 2078-11-13T03:00:00.000Z, 2078-11-15T03:00:00.000Z, 2078-11-19T03:00:00.000Z, 2078-11-20T03:00:00.000Z]

/*
  2078-11-12 = Saturday(non-business)
  2078-11-13 = Sunday(non-business)
  2078-11-14 = Monday(business)
  2078-11-15 = Tuesday(non-business)
  2078-11-16 = Wednesday(business)
  2078-11-17 = Thursday(business)
  2078-11-18 = Friday(business)
  2078-11-19 = Saturday(non-business)
  2078-11-20 = Sunday(non-business)
  2078-11-21 = Monday(business)
  2078-11-22 = Tuesday(business)
  So the non-business days are 2078-11-12, 2078-11-13, 2078-11-15, 2078-11-19, and 2078-11-20.
*/
```
Parameters:
- `startDate`: The start date from which to calculate the non-business days.
- `endDate`: The end date until which to calculate the non-business days.
- Returns an array of non-business days between the two dates.

### NextDisbursementDate
Calculates the next disbursement date based on a `baseDate`

```go
import (
  "fmt"
  "time"

  payment_plan "github.com/ParceladoLara/payment-plan-go-sdk/v3"
)
const baseDate = new Date('2078-02-12');
baseDate := time.Date(2078, 02, 12, 0, 0, 0, 0, time.FixedZone("UTC", 0));
result := payment_plan.NextDisbursementDate(baseDate);
fmt.Println("Next Disbursement Date:", result); // 2078-02-16T03:00:00.000Z
/*
  2078-02-12 = Saturday(invalid)
  2078-02-13 = Sunday(invalid)
  2078-02-14 = Bank holiday(invalid)
  2078-02-15 = Bank holiday(invalid)
  2078-02-16 = Wednesday(valid) 1
*/
```
Parameters:
- `baseDate`: The base date from which to calculate the next disbursement date.
- Returns the next valid disbursement date after the `baseDate`.

Warning:
As of now, if `baseDate=today` `baseDate` will be considered a invalid date, but this can change in the future.

## Contributing

This repository is public, but the Lara Payment Plan library is private.
To build the library, you must have access to the private repository.
The code here is generated from the private repository and then made public.
Pull requests are welcome, but changes will be made in the private repository and then propagated here.

## License
This software is provided free of charge for personal or internal business use only.
Modification, redistribution, sublicensing, or reverse engineering is not permitted.
Copyright (c) 2025 SWEETPAY SOLUCOES FINANCEIRAS LTDA. All rights reserved.