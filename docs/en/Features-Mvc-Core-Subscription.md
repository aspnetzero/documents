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

When subscription of a Tenant is about to expire, an email is sent to email address of the tenant to remind this expiration. This job is handled in **SubscriptionExpireEmailNotifierWorker.cs** background worker. This worker runs once in every day and sends email to tenants whose license will expire after N days later. The day count of N is stored in a setting named "**App.TenantManagement.SubscriptionExpireNotifyDayCount**" and its default value is 7 days. You can change it in **AppSettings.cs** class in *.Core project.

When the subscription of a Tenant is expired, **SubscriptionExpirationCheckWorker.cs** (it is localed next to SubscriptionExpireEmailNotifierWorker.cs) comes into play and executes the logic below:

* If **WaitingDayAfterExpire** is set for the Edition of the tenant, ASP.NET Zero waits for **WaitingDayAfterExpire** days to end Tenant's subscription.
* If **"assign to another edition"** option is selected for the Edition of the Tenant, tenant will be moved to the fallback edition.
* If "**deactive tenant**" option is selected for the Edition of the Tenant, tenant will be disabled and will not be able to use the system.
* If the subscription expires for a tenant who subscribed for trial usage, the tenant will be disabled and will not be able to use the system.

## Next

- [PayPal Integration](Features-Mvc-Core-Subscription-PayPal)
- [Stripe Integration](Features-Mvc-Core-Subscription-Stripe)