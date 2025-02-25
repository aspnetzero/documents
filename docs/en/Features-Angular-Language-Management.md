# Language Management

Language management page is used to manage (add/edit/delete) **application languages** and change **localized texts**:

<img src="images/language-list-core-3.png" alt="Language management" class="img-thumbnail" />

We can create new language, edit/delete an existing language and **set a language as default**. Note that; tenants can not create/edit/delete default languages, but host users can do. 

When we click to **Change text** for any language, we are redirected to a new view to edit language texts:

<img src="images/language-change-text-modal-core-3.png" alt="Language texts" class="img-thumbnail" />

We can select any language as a **base** (reference) and change **target** language's texts. Base language is just to help the translation progress. Since there maybe different [localization sources](https://aspnetboilerplate.com/Pages/Documents/Localization#DocLocalizationSources), we select the source to translate. When we click the edit icon, we can see the edit modal for the selected text:

<img src="images/language-change-text-modal-core-1.png" alt="Language text editing" class="img-thumbnail" />

**Host** users (if allowed) can edit languages and localized texts. These languages will be default for all tenants for a multi-tenant application. **Tenants** inherit languages and localized texts and can **override** localized texts.

If you want to add a new language, you can review the documentation on [How to Add a New Language in an ASP.NET Zero Angular Application](Adding-New-Localization-Angular)

See [language management](https://aspnetboilerplate.com/Pages/Documents/Zero/Language-Management) and [localization](https://aspnetboilerplate.com/Pages/Documents/Localization) documents for more information.

## Next

- [Audit Logs](Features-Angular-Audit-Logs)