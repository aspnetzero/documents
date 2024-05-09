# ASP.NET Zero Common Payment System Integration

ASP.NET Zero offers developers various functionalities with its powerful features. In this article, we will explore step-by-step how to utilize ASP.NET Zero's provided payment system infrastructure, as well as how to use the integrated Stripe and PayPal.

Integrating a common payment system into your platform offers numerous benefits, streamlining transaction processes and enhancing user experience. With a common payment system, you can centralize payment handling, simplifying management and ensuring consistency across different payment methods.

## Using the Payment Manager
You can manage payment transactions through the payment manager provided by ASP.NET Zero `(IPaymentManager)`. To initiate a payment, you can use the `CreatePayment` method of `IPaymentManager`. Additionally, you can utilize the `CreatePayment` method of the `PaymentAppService` to initiate the payment process.

## Creating Payment Request
When creating a payment request, you will utilize the `SubscriptionPayment` entity. The details of the SubscriptionPayment entity are as follows:

- **TenantId:** Represents which Tenant this payment request belongs to.
- **PaymentPeriodType:** Period type of the payment if this is a payment for a specific period. Currently, Monthly and Annual are supported.
- **DayCount:** Integer value of PaymentPeriodType field.
- **Gateway:** Name of payment gateway which processed this payment. This is set by ASP.NET Zero when the payment is successfull.
- **Status:** Status of payment. This is set by ASP.NET Zero.
- **ExternalPaymentId:** Id of the payment in the external payment gateway system like Stripe or PayPal. This is set by ASP.NET Zero.
- **InvoiceNo:** Invoice number if an invoice generated in ASP.NET Zero for this payment.
- **SuccessUrl:** URL to redirect user if payment is successfull.
- **ErrorUrl:** URL to redirect user if payment is failed.
- **IsRecurring:** Represents if this is a recurring payment or not. If it is recurring, user's credit card will be charged at the end of every payment cycle. This is only supported by Stripe at the moment.
- **IsProrationPayment:** This is a special field. If the tenant is on a recurring payment plan and operation is upgrade, then it is a proration payment.
- **ExtraProperties:** A dictionary to store additional information on the payment object.
- **SubscriptionPaymentProducts:** List of products to be purchased for this payment..

## SubscriptionPaymentProduct Entity

`SubscriptionPaymentProduct` entity represents the details of products purchased in payment transactions. The properties of purchased products for each payment request are maintained through this entity.

- **SubscriptionPaymentId:** This property represents the identifier of the corresponding payment record. It has a unique identity for each payment request and is used to distinguish payment records from each other.
- **Description:** This property contains the description of the purchased product or service.
- **Amount:** This property represents the price of the product, i.e., the unit price of the purchased product or service. This price is specified in the currency.
- **Count:** This property specifies the quantity of products to be purchased. In case a user purchases multiple products, the quantity of each product is specified with this property.
- **TotalAmount:** This property represents the total price, i.e., the total cost of the purchased products. It is calculated by multiplying the Amount property by the quantity of the product.
- **ExtraProperties:** This property is used to store additional information about the product object in a dictionary.

Integration of payment methods in ASP.NET Zero is highly flexible and easily customizable. By following these steps, you can seamlessly integrate a secure and functional payment system into your ASP.NET Zero project.

## Integrated Stripe and PayPal
ASP.NET Zero provides integrated support for payment systems like Stripe and PayPal, enabling developers to easily build their applications on a reliable payment infrastructure.

## Creating Stripe and PayPal Accounts
Firstly, you'll need to create Stripe and PayPal accounts. You can create your accounts and obtain API keys from the developer panels of Stripe and PayPal.

After creating the accounts, it is necessary to fill in the required fields in the `appsettings.json` file.

```json
"Payment": {
  "PayPal": {
    "IsActive": "true",
    "Environment": "sandbox",
    "BaseUrl": "https://api.sandbox.paypal.com/v1",
    "ClientId": "",
    "ClientSecret": "",
    "DemoUsername": "",
    "DemoPassword": "",
    "DisabledFundings": []
  },
  "Stripe": {
    "IsActive": "true",
    "BaseUrl": "https://api.stripe.com/v1",
    "SecretKey": "",
    "PublishableKey": "",
    "WebhookSecret": "",
    "PaymentMethodTypes": [
      "card"
    ]
  }
},
```

### Paypal

- **IsActive:** Indicates whether the PayPal account is active or not. If set to "true", the PayPal payment system is enabled.
- **Environment:** Specifies the environment in which the PayPal account operates. "sandbox" represents an environment used for development and testing purposes. The value "live" is used for live transactions.
- **BaseUrl:** Specifies the base URL for requests sent to the PayPal API.
- **ClientId and ClientSecret:** Represent the client ID and client secret used to access the PayPal API.
- **DemoUsername and DemoPassword:** Represent the demo username and password used for accessing the PayPal API.
- **DisabledFundings:** This field specifies the funding sources disabled in the PayPal payment flow. For example, "credit" or "card".

### Stripe

- **IsActive:** Specifies whether the Stripe account is active or not. If set to "true", the Stripe payment system is enabled.
- **BaseUrl:** Specifies the base URL for requests sent to the Stripe API.
- **SecretKey:** Represents the secret key used to access the Stripe API.
- **PublishableKey:** Represents the publishable key used for creating Stripe payment forms.
- **WebhookSecret:** Represents the secret key used to authenticate the Stripe webhook.
- **PaymentMethodTypes:** This field specifies the payment methods available in the Stripe payment system. For example, the value "card" represents credit card payments.

After filling in the required fields, you can easily utilize the Stripe and PayPal payment systems.

![Common Payment Gateway Selection](/Images/Blog/integrated-common-payment-gateway-selection.png)

After completing all the configurations, there are active payment methods available at the "Payment/GatewaySelection" address. Once a method is selected from these options, the payment process is initiated, and upon successful completion, the subscription is activated.

## Don't Miss Out! 

We have published new blog posts about ASP.NET Zero. You can check them out below:

* [Mastering Multi-lingual Database Design in ASP.NET Core with EF Core](https://aspnetzero.com/blog/mastering-multi-lingual-database-design-in-asp.net-core-with-ef-core)

* [ASP .NET Core Configuration](https://aspnetzero.com/blog/asp.net-core-configuration)

* [Introducing ASP.NET Zero v.13.1](https://aspnetzero.com/blog/introducing-asp.net-zero-v.13.1)


## Conclusion

The common payment system provided by ASP.NET Zero offers developers convenience in integrating payment methods seamlessly. With these capabilities, developers can streamline their workflow, improve user experience, and enhance business performance. 