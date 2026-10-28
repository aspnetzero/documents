# Workflows

ASP.NET Zero includes a built-in **workflow and approval builder**. Tenant administrators design request types such as leave requests, purchase approvals or document reviews in a visual designer. Employees then start requests from a catalog, and approvers decide on them from their inbox.

The same feature is available in the MVC, Angular and React UIs. The designer is a shared component, so the screens and labels described in these documents are the same in all three.

## What you can build

A workflow combines a **form** with a **flow**.

- The **form** is what the requester fills in: text, numbers, dates, selections, users, attachments and more.
- The **flow** is the path a request takes after it is submitted: approvals by people, roles or organization units, conditions that branch on form values, timers, notifications and, if enabled, calls to other systems.

For example, a leave request can go to the employee's team lead, then to HR only when more than five days are requested, and finally notify the employee of the result.

## Key concepts

| Concept | Description |
|---|---|
| **Workflow definition** | A request type, for example "Leave request". It has a form, a flow and a trigger. |
| **Draft and version** | You edit a draft. Publishing turns the draft into a new version (v1, v2, …). Requests that are already running keep the version they started with. |
| **Trigger** | How requests are started: **Manual** (from the Request Catalog), **Scheduled** or **Public form**. |
| **Request** | One run of a workflow, started by a requester. Each request gets a reference number and a timeline. |
| **Task** | A decision assigned to an approver. Tasks appear in the **Approval Inbox**. |
| **Delegation** | A period during which an approver's tasks go to a colleague. |

## Where to find it

The workflow pages are tenant pages. Log in as a tenant user to use them.

| Menu | Pages |
|---|---|
| **Workflows** | Start a Request, My Requests, Approval Inbox, Workflow Delegations |
| **Workflows > Workflow Administration** | Workflow Definitions, Workflow Templates, Workflow Instances, Workflow Operations, Workflow Audit, Process analytics, Workflow Settings and the pages of optional features (events, AI, secrets, connection profiles, action catalog) |

The host has a separate **Workflows** menu with **Workflow Health** and **Workflow AI usage**. Host administrators configure the defaults and limits that apply to every tenant in **Administration > Settings > Workflows**.

The menus are only shown when workflows are enabled and the user has the related permissions. See [Workflow Administration](Features-Workflows-Administration) for the settings and permissions.

## Documents

- [Create Your First Workflow](Features-Workflows-Create-Your-First-Workflow): A step-by-step tutorial that builds and runs a leave request workflow. Start here.
- [Workflow Designer](Features-Workflows-Designer): The form builder, every step type, approvers, conditions, triggers, validation, simulation and versions.
- [Requests and Approvals](Features-Workflows-Requests-And-Approvals): Starting requests, deciding on tasks, requesting changes and delegating.
- [Workflow Administration](Features-Workflows-Administration): Host and tenant settings, permissions and the monitoring pages.

## Next

- [Create Your First Workflow](Features-Workflows-Create-Your-First-Workflow)
