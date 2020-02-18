# Subscription

Tenants can manage (show, extend or upgrade) their edition/plan subscriptions using this page:

<img src="images/subscription-1.png" alt="Subscription" class="img-thumbnail" />

All payment records for extending/upgrading current licenses are kept in the system. These records can be seen on "Payment History" tab on subscription page.

<img src="images/subscription-payment-history-core.png" alt="Payment History" class="img-thumbnail" />

Tenants can also create & print invoices for these payments by clicking the "Show Invoice" button. System will automatically generate an invoice number and show the generated invoice. In order to use this function, both Host & Tenant must set invoice informations on host setting/tenant setting page.

<img src="images/host-settings-invoice.png" alt="Tenant Invoice Settings" class="img-thumbnail" />

After all, invoices for payments related to subscription can be generated. You can see a sample invoice below:

<img src="images/sample-invoice-core.png" alt="Sample Invoice" class="img-thumbnail" />

AspNet Zeros subscription system allows using two payment gateways, one is [PayPal](https://www.paypal.com) and the other one is [Stripe](https://stripe.com/). You can configure both payment gateways in the `appsettings.json` file in ***.Web.Mvc** project.

#### Minimum Update Amount

Since payment systems have accepted the minimum payment amount, you may need to set the minimum payment amount according to your payment system. Settings are located in [`*.Core.Shared/AbpZeroTemplateConsts.cs`](https://github.com/aspnetzero/aspnet-zero-core/blob/218d375bb11609ce924a1d873aff7540ed8468b0/aspnet-core/src/MyCompanyName.AbpZeroTemplate.Core.Shared/AbpZeroTemplateConsts.cs#L19-L24)

```csharp
// Note:
// Minimum accepted payment amount. If a payment amount is less then that minimum value payment progress will continue without charging payment
// Even though we can use multiple payment methods, users always can go and use the highest accepted payment amount.
//For example, you use Stripe and PayPal. Let say that Stripe accepts min 5$ and PayPal accepts min 3$. If your payment amount is 4$.
// User will prefer to use a payment method with the highest accept value which is a Stripe in this case.
public const decimal MinimumUpgradePaymentAmount = 1M;
```

 and [`angular/src/shared/AppConsts.ts`](https://github.com/aspnetzero/aspnet-zero-core/blob/218d375bb11609ce924a1d873aff7540ed8468b0/angular/src/shared/AppConsts.ts#L31) . 

```typescript
static readonly MinimumUpgradePaymentAmount = 1;
```

Default value is **1**. 

Payment progress will be continued without charging any amount if the payment amount is less than given value.

## Next

- [PayPal Integration](Features-Mvc-Core-Subscription-PayPal)
- [Stripe Integration](Features-Mvc-Core-Subscription-Stripe)