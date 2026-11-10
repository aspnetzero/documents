# Cookie Consent

ASP.NET Zero can show a cookie consent banner to the users of your application. Users can accept all optional cookies, reject them, or choose the cookie categories they allow, and they can change their choice at any time.

The banner is built with [vanilla-cookieconsent](https://github.com/orestbida/cookieconsent) v3. It supports [Google Consent Mode v2](https://developers.google.com/tag-platform/security/guides/consent), so Google Analytics and Google Tag Manager only use the storage that the user has agreed to.

## Enabling Cookie Consent

Cookie consent is **off by default**. To turn it on, a host administrator navigates to **Administration > Settings > User Management** and selects **Cookie consent enabled** in the **Cookie consent** section.

| Setting | Setting name | Default |
|---------|--------------|---------|
| **Cookie consent enabled** | `App.UserManagement.IsCookieConsentEnabled` | `false` |

The setting is defined for both the host and tenants. A tenant uses the host's value until a tenant administrator changes it in the **User Management** tab of the tenant settings, so each tenant can turn the banner on or off for its own users.

The initial value can also be set in `appsettings.json` before the first run:

```json
{
  "App": {
    "App.UserManagement.IsCookieConsentEnabled": "true"
  }
}
```

> Values in `appsettings.json` are only the **initial defaults**. Once the value has been changed on the settings page, it is stored in the database and the configuration file no longer has any effect on it.

The banner is created when the application starts, so a change of the setting takes effect after the page is reloaded.

## What Users See

When cookie consent is on, a user who has not made a choice yet sees the consent banner at the bottom right of the page, both on the account pages (such as the login page) and in the application. The banner has three buttons:

| Button | Result |
|--------|--------|
| **Accept all** | Allows all cookie categories. |
| **Reject all** | Allows only the strictly necessary cookies. |
| **Manage preferences** | Opens the preferences dialog, where the user can turn each optional category on or off and save the choice. |

The choice is saved in the `cc_cookie` cookie of the browser, so the banner is not shown again on that browser.

Users can change their choice at any time with the **Cookie preferences** link. The link is shown in the footer of the application and next to the language selection on the account pages. It is only shown when cookie consent is enabled.

## Cookie Categories

The banner has three categories by default:

| Category | Can be turned off | Google consent types granted when accepted | Cookies deleted when rejected |
|----------|-------------------|--------------------------------------------|-------------------------------|
| `necessary` (Strictly necessary) | No | `functionality_storage`, `security_storage` | - |
| `analytics` (Analytics) | Yes | `analytics_storage` | `_ga*`, `_gid` |
| `marketing` (Marketing) | Yes | `ad_storage`, `ad_user_data`, `ad_personalization`, `personalization_storage` | `_gcl_*` |

The optional categories are off until the user accepts them. When the user rejects an optional category, the cookies listed for it are deleted.

See [Customization](#customization) to change these categories.

## Google Tag and Consent Mode

ASP.NET Zero can load a Google tag for you, either Google Analytics 4 or a Google Tag Manager container. Set the id of the tag with `CookieConsent:GoogleTagId` in the `appsettings.json` file of the `Web.Host` project:

```json
{
  "CookieConsent": {
    "GoogleTagId": "G-XXXXXXXXXX"
  }
}
```

| Value | Behaviour |
|-------|-----------|
| Empty (default) | No Google tag is loaded. |
| An id that starts with `GTM-` | The Google Tag Manager container (`gtm.js`) is loaded. |
| Any other id, such as `G-XXXXXXXXXX` | The Google tag (`gtag.js`) is loaded and configured with this id. |
| An invalid id | Nothing is loaded and a warning is written to the browser console. |

The value is exposed to the client as the `App.CookieConsent.GoogleTagId` setting. It is an application-level setting, so the host and all tenants use the same tag. It cannot be changed on the settings page.

When cookie consent is on, the Google tag is loaded with Consent Mode as follows:

1. Before the banner is shown, all Google consent types are set to `denied`, except the types of the strictly necessary category. `wait_for_update` is set to `500` milliseconds.
2. If the user has already made a choice, the consent types of the accepted categories are updated to `granted`.
3. The Google tag is loaded.
4. Every time the user accepts, rejects or changes a choice, the consent types are updated again.

> **Important:** If cookie consent is off but a Google tag id is set, the Google tag is loaded without Consent Mode, so it sets its cookies without asking the user. Since the tag is shared by all tenants but a tenant can turn cookie consent off for its own users, keep cookie consent on wherever your users have to give consent, for example in the EU, the EEA and the UK.

Use `CookieConsent:GoogleTagId` instead of adding the Google tag snippet to `index.html` yourself. A tag added to `index.html` runs before the Consent Mode defaults are set.

> If you add a Content Security Policy to your application, allow `https://www.googletagmanager.com` for scripts, and the Google Analytics endpoints for connections, so that the Google tag can be loaded.

## Customization

The cookie consent files of the Angular application are in the `src/shared/common/cookie-consent` folder:

| File | Purpose |
|------|---------|
| `cookie-consent.config.ts` | The configuration of the banner: categories, layout and revision. This is the file to change in most cases. |
| `cookie-consent.service.ts` | `CookieConsentService`. Its `init` method starts the banner and loads the Google tag when the application starts. Its `showPreferences` method opens the preferences dialog. |
| `create-cookie-consent-config.ts` | Converts the configuration and the localized texts to the configuration of vanilla-cookieconsent. |
| `google-consent-mode.ts` | Sets and updates the Consent Mode state and loads the Google tag. |
| `cookie-preferences-link.component.ts` | The **Cookie preferences** link (`<cookie-preferences-link />`). Its `buttonClass` input sets the CSS classes of the link. |
| `cookie-consent.css` | The colors of the banner. |

### Configuration

`cookieConsentConfiguration` in `cookie-consent.config.ts` has the following options:

| Option | Default | Description |
|--------|---------|-------------|
| `revision` | `0` | The revision of your cookie policy. Increase it when your cookie policy changes. Users who gave their consent for another revision see the banner again. |
| `googleConsentWaitForUpdate` | `500` | The `wait_for_update` value, in milliseconds, of the Consent Mode defaults. |
| `guiOptions` | A box at the bottom right | The layout and position of the banner and of the preferences dialog. See the [GUI options](https://cookieconsent.orestbida.com/reference/configuration-reference.html#guioptions) of vanilla-cookieconsent. |
| `categories` | `necessary`, `analytics`, `marketing` | The cookie categories. See below. |

### Adding a category

Each item of `categories` has the following properties:

| Property | Description |
|----------|-------------|
| `name` | The name of the category, such as `analytics`. |
| `titleKey`, `descriptionKey` | The localization keys of the title and the description of the category in the preferences dialog. |
| `readOnly` | `true` for a category that is always on and cannot be turned off by the user. |
| `googleConsentTypes` | The Google consent types that are granted when the user accepts the category. Use an empty array if the category is not related to Google tags. |
| `cookiesToClear` | The names of the cookies, or regular expressions that match them, that are deleted when the user rejects the category. |

For example, to ask for consent for the cookies of a live chat widget, add a category like below:

```typescript
{
    name: 'support',
    titleKey: 'CookieConsent_SupportTitle',
    descriptionKey: 'CookieConsent_SupportDescription',
    googleConsentTypes: [],
    cookiesToClear: [/^livechat_/],
},
```

Then add the `CookieConsent_SupportTitle` and `CookieConsent_SupportDescription` keys to the localization files in the `Localization/AbpZeroTemplate` folder of the `*.Core` project.

### Running scripts only after consent

vanilla-cookieconsent can run a script only after the user accepts a category. Set the `type` of the script to `text/plain` and its `data-category` to the name of the category:

```html
<script type="text/plain" data-category="analytics" src="https://example.com/analytics.js"></script>
```

In code, use `acceptedCategory` to check whether the user has accepted a category:

```typescript
import * as CookieConsent from 'vanilla-cookieconsent';

if (CookieConsent.acceptedCategory('analytics')) {
    // The user has accepted the analytics cookies.
}
```

See the [vanilla-cookieconsent documentation](https://cookieconsent.orestbida.com/) for all options and methods.

### Styling

`cookie-consent.css` maps the [CSS variables](https://cookieconsent.orestbida.com/advanced/ui-customization.html) of vanilla-cookieconsent to the Bootstrap variables of the theme, such as `--bs-primary` and `--bs-body-bg`. So the banner uses the colors of the active theme. Change these variables to customize the colors.

## Localization

The texts of the banner come from the localization source of the application, so the banner is shown in the language of the user. Right-to-left languages are also supported.

| Key | English text |
|-----|--------------|
| `CookieConsent_Title` | We value your privacy |
| `CookieConsent_Description` | We use cookies that are necessary for this application to work. With your consent, we also use optional cookies to analyze usage and improve your experience. ... |
| `CookieConsent_AcceptAll` | Accept all |
| `CookieConsent_RejectAll` | Reject all |
| `CookieConsent_ManagePreferences` | Manage preferences |
| `CookieConsent_Preferences` | Cookie preferences |
| `CookieConsent_PreferencesDescription` | Choose which optional cookies you allow. ... |
| `CookieConsent_SavePreferences` | Save preferences |
| `CookieConsent_Close` | Close |
| `CookieConsent_NecessaryTitle`, `CookieConsent_NecessaryDescription` | Title and description of the strictly necessary category. |
| `CookieConsent_AnalyticsTitle`, `CookieConsent_AnalyticsDescription` | Title and description of the analytics category. |
| `CookieConsent_MarketingTitle`, `CookieConsent_MarketingDescription` | Title and description of the marketing category. |

Change these texts in the localization files, or from the **Languages** page, if you need a different wording. For example, if your application does not use marketing cookies, remove the `marketing` category and update `CookieConsent_Description`.

## Upgrading from an Earlier Version

Earlier versions used the [cookieconsent](https://github.com/osano/cookieconsent) package, which only showed a message with a **Got it!** button. If you upgrade an existing project, note the following changes:

- The `cookieconsent` package is replaced by `vanilla-cookieconsent`. Its script is removed from the `scripts` of `angular.json`, and its style is replaced by `node_modules/vanilla-cookieconsent/dist/cookieconsent.css` and `src/shared/common/cookie-consent/cookie-consent.css`.
- `CookieConsentService` is moved from `src/shared/common/session/cookie-consent.service.ts` to `src/shared/common/cookie-consent/cookie-consent.service.ts`. Update your imports if you use it.
- The `.cc-window`, `.cc-btn` and `.cc-message` styles in `metronic-customize-angular.css` are removed. Use the variables in `cookie-consent.css` instead.
- The `CookieConsent_Message` and `CookieConsent_Dismiss` localization keys are removed. If you changed or translated them, apply your changes to the new keys.
- The `CookieConsent:GoogleTagId` configuration and the `App.CookieConsent.GoogleTagId` setting are new.
- The new banner stores the choice in a different cookie, so users who dismissed the old message see the new banner once.

## Next

- [Tenant Settings](Features-Angular-Tenant-Settings)
