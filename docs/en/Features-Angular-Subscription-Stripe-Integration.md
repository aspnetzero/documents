# Stripe Integration

In order to configure Stripe, open `appsettings.json` file in ***.Web.Host** project and fill the fields below:

- **IsActive:** This setting can be used to enable/disable Stripe. If set to false, end users will not see Stripe option during the payment process.
- **BaseUrl:** URL for making API calls to Stripe. You can find correct URLs in your Stripe dashboard. 
- **SecretKey:** Your Stripe SecretKey.
- **PublishableKey:** Your Stripe PublishableKey.
- **WebhookSecret:** Your Stripe WebHookSecret which is used to validate WebHook requests.

Stripe supports recurring payments. If a tenant wants to pay via Stripe and accepts automatically billing the account used for the initial payment, then Stripe charges the amount from Tenants account on each subscription cycle and notifies ASP.NET Zero. Then, ASP.NET Zero extends the subscription for the paid period (either monthly or annual).

<img src="images/subscription-stripe-recurring-payments.png" alt="Stripe recurring payments" class="img-thumbnail" />

If "Automatically bill my account" option is not selected on the payment page, tenants should login to the system and manually extend their subscription by clicking the "**Extend**" button on the subscription page and pay manually.	

## Testing Stripe WebHooks on Localhost

In order to get Stripe's WebHook request on your local environment, you need to use an external tool. [https://webhookrelay.com](https://webhookrelay.com) is one of the best tools on the web at the moment. [How to receive Stripe webhooks on localhost](https://webhookrelay.com/blog/2017/12/26/receiving-stripe-webhooks-localhost/) can be used to test Stripe's WebHooks on the localhost. Basically, you need to create an account on [https://webhookrelay.com](https://webhookrelay.com), then need to download relay.exe to your development machine. 

Then, you need to run relay.exe like this;

```./relay.exe forward --bucket stripe http://localhost:22742/Stripe/WebHooks```

This will give you an url something like "https://my.webhookrelay.com/v1/webhooks/aa180d45-87d5-4e9c-8bfa-e535a91df3fc". You need to enter this url as an WebHook endpoint on Stripe's WebHook dashboard ([https://dashboard.stripe.com/account/webhooks](https://dashboard.stripe.com/account/webhooks)).

Don't forget to enter your production app's URL as a WebHook endpoint when you publish your app to production.

Note that;

- Tenants can disable or enable Stripe to charge their accounts automatically on the Subscription Page. 
- When upgrading to an higher edition, stripe calculates the cost for upgrade and charges it from Tenants account. However, ASP.NET Zero can't show this amount during the edition upgrade process. But, this amount can be seen at the Payment History tab on Subscription page after a successful payment process.
- When a tenant subscribes to an edition using Stripe and if admin user changes the edition of the Tenant on Tenant page to a higher edition, Tenant's account will be charged on stripe automatically. If the Tenant is moved to an Edition with a lower price, there will be no change or refund on Stripe.

## Next

- [Visual Settings](Features-Angular-Visual-Settings)