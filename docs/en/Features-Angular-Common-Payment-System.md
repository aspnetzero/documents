# Common Payment System

ASP.NET Zero provides a payment system to get payments easily. In order to start a payment, just use `CreatePayment` method of `IPaymentManager`. You can create a new application service or you can use existing `CreatePayment` method of `PaymentAppService`. Then, you need to redirect user to `gateway-selection` route of `account` module as shown below;

```typescript
this._router.navigate(
                ['account/gateway-selection'],
                {
                    queryParams: {
                        paymentId: paymentId
                    },
                }
            );
```

ASP.NET Zero's common payment system will handle the rest of the payment flow. 

- If the payment process is successfull, user will be redirected to `SuccessUrl` provided when creating the payment request.
- If the payment process is unsuccessfull, user will be redirected to `ErrorUrl` provided when creating the payment request.
- If `SuccessUrl` or `ErrorUrl` is not provided, user will be redirected to a common page and result of the payment process will be displayed.

## Creating Payment Request

### SubscriptionPayment Entity

`IPaymentManager` is used to create a payment requet. It's `CreatePayment` method requires a `SubscriptionPayment` entity. Here is the detials of `SubscriptionPayment` entity.

* `TenantId`: Represents which Tenant this payment request belongs to.
* `PaymentPeriodType`: Period type of the payment if this is a payment for a specific period. Currently, Monthly and Annual are supported.
* `DayCount`: Integer value of `PaymentPeriodType` field.
* `Gateway`: Name of payment gateway which processed this payment. This is set by ASP.NET Zero when the payment is successfull.
* `Status`: Status of payment. This is set by ASP.NET Zero. 
* `ExternalPaymentId`: Id of the payment in the external payment gateway system like Stripe or PayPal. This is set by ASP.NET Zero. 
* `InvoiceNo`: Invoice number if an invoice generated in ASP.NET Zero for this payment.
* `SuccessUrl`: URL to redirect user if payment is successfull.
* `ErrorUrl`: URL to redirect user if payment is failed.
* `IsRecurring`: Represents if this is a recurring payment or not. If it is recurring, user's credit card will be charged at the end of every payment cycle. This is only supported by Stripe at the moment.
* `IsProrationPayment`: This is a special field. If the tenant is on a recurring payment plan and operation is upgrade, then it is a proration payment.
* `ExtraProperties`: A dictionary to store additional information on the payment object. 
* `SubscriptionPaymentProducts`: List of products to be purchased for this payment. 

### SubscriptionPaymentProduct Entity

* `SubscriptionPaymentId`: Id of the related payment record.
* `Description`: Description of the purchased product.
* `Amount`: Price of the product.
* `Count`: Count of products to be purchased.
* `TotalAmount`: Total price of products to be purchased.
* `ExtraProperties`: A dictionary to store additional information on the product object. 