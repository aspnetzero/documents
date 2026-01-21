**Title:** React UI has arrived in ASP.NET Zero
**Description:** ASP.NET Zero introduces React UI, a modern, enterprise ready frontend built with React, Vite, Ant Design, and TypeScript. Deliver scalable applications faster with the same backend infrastructure.

# React UI has arrived in ASP.NET Zero

We’re excited to introduce **React UI for ASP.NET Zero**, a new frontend option designed for teams that prefer the React ecosystem and need the full power of an enterprise ready backend.

With this release, ASP.NET Zero now offers **Angular, MVC, and React** frontends all backed by the same, production infrastructure. You can choose the UI technology that best fits your team, without compromising on features, scalability, or security.

By providing React as a native and fully integrated UI option, ASP.NET Zero helps teams reduce upfront development effort, shorten time to market, and avoid the hidden costs of building and maintaining an enterprise frontend from scratch.

Instead of assembling and maintaining a complex React enterprise stack from scratch, teams start with a production ready frontend that already solves authentication, authorization, multi tenancy, and localization concerns.


> **Who is this for?**
>
> This release is designed for teams building a new React based enterprise application or launching a SaaS product who want to avoid spending months assembling authentication, authorization, multi tenancy, and administration UI from scratch.


<img src="/Images/Blog/react-tenant-dashboard.png" alt="React Dashboard" class="img-thumbnail" />


## Why React for ASP.NET Zero?

React has become the frontend of choice for many development teams. Beyond its popularity, React offers tangible advantages for enterprise applications:

* **Strong ecosystem and hiring pool** – Easier onboarding and long term maintainability
* **Component driven architecture** – Clean, reusable UI patterns
* **High performance** – Efficient rendering and predictable state updates
* **TypeScript support** – Safer, more maintainable codebases

By adding React UI, ASP.NET Zero meets developers where they already are enabling teams to build modern enterprise applications using familiar tools and workflows.

Teams often underestimate how much enterprise functionality must live in the frontend.

What starts as a simple React setup often grows into months of extra work when authentication, permissions, multi tenant navigation, and administration features are added later typically under delivery pressure.

## Built on a Modern React Stack

The React UI is built from the ground up using technologies from the React ecosystem, carefully chosen for performance, scalability, and developer experience.

All of these technologies come preconfigured and integrated, allowing teams to focus on delivering business value instead of spending weeks on setup and architectural decisions.

Instead of evaluating and integrating each of these technologies independently, the React UI provides a pre wired, opinionated setup that is already aligned with ASP.NET Zero’s backend architecture.

### Vite for Fast Development and Optimized Builds

The React UI uses **Vite** as its build tool to provide a fast and efficient development experience. Instant startup times, extremely fast hot module replacement, and optimized production builds significantly reduce feedback cycles, especially in large scale applications.

Leveraging native ES modules and modern bundling approaches, Vite enables faster development cycles while maintaining build quality and performance.

### Ant Design for Enterprise Ready UI Components

The UI is built on **Ant Design**, a widely used React component library commonly adopted in enterprise applications. It provides a broad set of components with a consistent design language, strong TypeScript support, and built in theming options.

This allows teams to focus on business logic and application behavior rather than implementing common UI patterns from scratch.

### State Management with Redux Toolkit

Application state is managed using **Redux Toolkit**, providing a centralized approach to state management. The setup includes simplified APIs, DevTools support, and TypeScript integration.

This ensures consistent behavior, easier debugging, and long term maintainability as applications grow.


### Routing with React Router

Routing is handled using **React Router**. The configuration includes declarative route definitions, lazy loaded pages, protected routes based on authentication and permissions, and nested layouts.

This structure provides a consistent approach to organizing application navigation.

## Metronic Theme

As with Angular and MVC, the React UI is integrated with **Metronic 8**. It includes **13 predefined theme variations**, responsive layouts, and configurable layout options.

Layout, and appearance settings can be adjusted directly through the UI, without requiring changes at the code level.

This ensures visual and behavioral consistency across Angular, MVC, and React frontends, making it easier for teams to switch or run multiple UIs without redefining design standards.

This integration provides a ready to use UI foundation that aligns with the existing ASP.NET Zero frontends.


<img src="/Images/Blog/metronic-13-themes.png" alt="Metronic 13 Themes" class="img-thumbnail" />


## Authentication, Administration, and Configuration

The UI includes authentication and security features covering standard login flows, multi factor authentication, and external authentication providers. Administrative functionality includes user management, roles, permissions, organization units, tenants, editions, and feature configuration.

Host and tenant level settings include security policies, email configuration, appearance options, and integrations such as LDAP. Configuration options available in Angular and MVC are also present in the React UI.

<img src="/Images/Blog/react-roles.png" alt="React Roles Page" class="img-thumbnail" />

## Internationalization (i18n)

Localization support is available in the React UI. Applications can switch languages dynamically, leverage Ant Design’s localized components, and display dates and times based on the selected locale.

This makes it straightforward to build fully multilingual enterprise applications.


## Real Time Notifications with SignalR

Built in **SignalR integration** supports real time communication scenarios within the application. Notifications can be sent to individual users or groups, with a management interface available for viewing notification history.

## Audit Logging and Entity History

The React UI fully supports ASP.NET Zero’s auditing infrastructure, providing detailed insight into system activity. User actions, entity changes, and login attempts are recorded and presented through searchable and filterable views.

These capabilities support compliance requirements, security reviews, and operational monitoring.

## Payments and Subscriptions

For SaaS scenarios, the React UI provides support for payments and subscriptions. Integrations with Stripe and PayPal enable recurring billing, while subscription lifecycle and invoice management can be handled through the admin interface.

This helps teams launch and operate SaaS products with reduced UI development effort.

<img src="/Images/Blog/react-subscription-management.png" alt="Subscription Management" class="img-thumbnail" />

## Developer Experience

Developer experience was a key focus when designing the React UI.

### TypeScript and Tooling

The frontend is written in **TypeScript** with strict typing enabled by default. Backend APIs are consumed via NSwag generated, type safe **service proxies**, helping establish clear contracts between the frontend and backend.

This approach helps improve code quality, minimize certain classes of runtime errors, and support long term maintainability.

As a result, teams spend less time debugging integration issues and more time delivering business features.

### Custom Hooks and Project Structure

Reusable custom React hooks simplify common concerns such as authentication, permissions, theming, localization, and data table behavior. The project structure is clean and scalable, making it easy to navigate, extend, and maintain as the application grows.

In addition to these, a range of other established patterns and utilities are included to support developers in building and maintaining complex React applications.

## Getting Started

Starting with React UI is straightforward:

1. Download your ASP.NET Zero solution 
2. Configure the database connection and apply the required database migrations.
3. Install frontend dependencies
4. Start the development server
5. Build for production when ready

[React UI Getting Started Guide](https://docs.aspnetzero.com/aspnet-core-react/latest/Getting-Started-React)

Hot module replacement is enabled to provide an efficient development experience.

Whether you are starting a new project or modernizing an existing one, React UI allows you to move faster with a predictable cost model and no surprises down the road.

## Feature Comparison

| Feature               | Angular | MVC | React |
| --------------------- | ------- | --- | ----- |
| Metronic Theme        | ✅       | ✅   | ✅     |
| Multi-tenancy         | ✅       | ✅   | ✅     |
| User Management       | ✅       | ✅   | ✅     |
| Role Management       | ✅       | ✅   | ✅     |
| Organization Units    | ✅       | ✅   | ✅     |
| Audit Logging         | ✅       | ✅   | ✅     |
| ...      | ...       | ...   | ...     |
| Custom Dashboards     | ✅       | ✅   | ✅     |
| SignalR Notifications | ✅       | ✅   | ✅     |
| Payment Integration   | ✅       | ✅   | ✅     |
| UI Customization      | ✅       | ✅   | ✅     |

Choosing React does not mean giving up any enterprise capability. It simply provides an alternative frontend workflow while preserving the same backend features and infrastructure.

For more details and the complete feature set, visit the [Features page](https://aspnetzero.com/features). 

All frontends provide the same enterprise feature set. React UI focuses on **modern frontend workflows, faster iteration, and the React ecosystem**.

## What’s Next?

React UI is actively developed to maintain long term parity with existing ASP.NET Zero frontends while continuously improving developer experience.

This release is just the beginning. Upcoming improvements include:

* Power Tools support for React
* More sample applications
* Social login integrations
* QR code based login integrations
* Passwordless login integrations

We actively support and continue to develop the React UI to maintain long term parity and ongoing improvements.

## Conclusion

React UI marks an important milestone for ASP.NET Zero, enabling teams to build enterprise grade applications using Angular, MVC, or React on the same proven backend foundation. 

For existing customers, this means adopting React without evaluating or assembling a separate enterprise frontend stack. For teams evaluating ASP.NET Zero, React UI provides a modern, production ready React ecosystem without sacrificing scalability, security, or long term maintainability.

We look forward to seeing what you build.
To learn more about the architecture, features, and setup of the React UI, you can visit the dedicated React page:

👉 [Explore React UI](https://aspnetzero.com/react)

If you want to see which ASP.NET Zero edition best fits your needs, you can explore our pricing options and choose the plan that matches your project scale and requirements. 

👉 [View Pricing](https://aspnetzero.com/pricing)

For questions or feedback, contact us at:
📩 **[info@aspnetzero.com](mailto:info@aspnetzero.com)**

## Additional Resources

* React: [https://aspnetzero.com/react](https://aspnetzero.com/react)
* Documentation: [https://docs.aspnetzero.com](https://docs.aspnetzero.com)
* GitHub: [https://github.com/aspnetzero/aspnet-zero-core](https://github.com/aspnetzero/aspnet-zero-core)
* Support Portal: [https://support.aspnetzero.com](https://support.aspnetzero.com)