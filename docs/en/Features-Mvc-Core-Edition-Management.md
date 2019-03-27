# Edition Management

*If you're not developing a multi-tenant application, you can skip this section.*

Most **SaaS** (multi-tenant) applications have **editions** (packages) those have different **features**. Thus, they can provide different **price and feature options** to their tenants (customers). **Editions page** (available in host login) is used to manage application's editions:

<img src="images/editions-page-core-4.png" alt="Editions Page" class="img-thumbnail" />

Editions are used to group feature values and assign to tenants. We can create a new edition by clicking "**Create new edition**" button:

<img src="images/edition-edit-1.png" alt="Edit Edition" class="img-thumbnail" />

An edition can be free or paid. If it's a paid edition then you should enter monthly and annual prices. You can allow tenants to use trial version of this edition for a specified days. Then you can determine an
expire strategy: How many days to allow a tenant to use the application after subscription expires. And finally, you can deactivate tenant or assign to a free edition if they don't extend their subscription.

Features tab is used to determining features available for the edition:

<img src="images/edition-feature-editing-core-1.png" alt="Edit edition features" class="img-thumbnail" />

## Next

- [Tenant Management](Features-Mvc-Core-Tenant-Management)