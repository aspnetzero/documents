# PayPal

In order to configure PayPal settings, open `appsettings.json` file in ***.Web.Mvc** project and fill the values;

- **IsActive:** This setting can be used to enable/disable PayPal. If set to false, end users will not see PayPal option during the payment process.
- **Environment:** Determines PayPal environment. "sandbox" is used for testing environment and "live" is used for production environment.
- **BaseUrl:** Url for making API calls to PayPal. You can find correct urls in your PayPal developer dashboard. 
- **ClientId**: ClientId for the PayPal app.
- **ClientSecret:** ClientSecret for the PayPal app.
- **DemoUsername:** Username for a demo account to show users in Demo mode for testing purposes.
- **DemoPassword:** Password for a demo account to show users in Demo mode for testing purposes.
- **DisabledFundings:** The disabled funding sources for the transaction, see https://developer.paypal.com/docs/checkout/reference/customize-sdk/#disable-funding for more details.

Note: Current implementation of PayPal doesn't support recurring payments. So, If a tenant wants to pay via PayPal, AspNet Zero will not charge Tenant's account automatically. In that case, Tenant needs to pay the subscription price on every subscription cycle.

## Next

- [Visual Settings](Features-Mvc-Core-Visual-Settings)