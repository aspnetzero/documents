# Subscription

Tenants can manage (show, extend or upgrade) their edition/plan subscriptions using this page:

<img src="D:/Github/documents/docs/en/images/subscription-1.png" alt="Subscription" class="img-thumbnail" />

All payment records for extending/upgrading current licenses are kept in the system. These records can be seen on "Payment History" tab on subscription page.

<img src="D:/Github/documents/docs/en/images/subscription-payment-history-core.png" alt="Payment History" class="img-thumbnail" />

Tenants can also create & print invoices for these payments by clicking the "Show Invoice" button. System will automatically generate an invoice number and show the generated invoice. In order to use this function, both Host & Tenant must set invoice informations on host setting/tenant setting page.

<img src="D:/Github/documents/docs/en/images/host-settings-invoice.png" alt="Tenant Invoice Settings" class="img-thumbnail" />

After all, invoices for payments related to subscription can be generated. You can see a sample invoice below:

<img src="D:/Github/documents/docs/en/images/sample-invoice-core.png" alt="Sample Invoice" class="img-thumbnail" />

AspNet Zeros subscription system allows using two payment gateways, one is [PayPal](https://www.paypal.com) and the other one is [Stripe](https://stripe.com/). You can configure both payment gateways in the `appsettings.json` file in ***.Web.Mvc** project.

## Next

- [PayPal Integration](Features-Mvc-Core-Subscription-PayPal)
- [Stripe Integration](Features-Mvc-Core-Subscription-Stripe)