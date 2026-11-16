---
title: Team Members
---

Team members let other people work in your organization without sharing a login.
Everyone you invite gets their own AutoPilot account and a role that decides what they can see and do.
A role applies to the whole organization, so it covers every deployment in it.

## Invite A Team Member

Open your organization, go to the **Team Members** tab, and click **Invite**.
Enter their email address, pick a role, and click **Invite** again.

They'll get an email with an **Accept Invitation** link.
If they don't have an AutoPilot account yet, they'll be asked to create one first.
They need to accept it while signed in with the same email address you invited.
The invitation expires after 7 days.

Until they accept, they show up in the list as pending.
You can cancel a pending invitation from the menu at the end of their row.

Only Owners and Managers can invite people.
You can't invite an email address that's already on the team or already has a pending invitation.

## Organization Roles

**Owner** is the person who created the organization.
There's only one, and it has full access to everything.
The Owner always receives invoice emails and can't be removed from the team.
If you need to hand ownership over to someone else, open a ticket with JetRails support.

**Manager** can do everything the Owner can, except retire the organization.
That includes inviting and removing team members, changing roles, and handling billing.

**Member** is for people doing day to day work on your existing deployments, like your developers.
They can run actions, trigger deployment runs, and manage instances and hibernation.
They can't create or delete deployments, approve patches, or change the maintenance window.
They also can't invite anyone or see anything billing related.

**Finance** is for whoever pays the bills.
They can view and pay invoices, manage credit cards, update the billing address, and check your credit balance and security deposit.
They can't see your deployments at all.

## Role Permissions

{.compact}
|                                                       | Owner                                                        | Manager                                                      | Member                                                       | Finance                                                      |
|-------------------------------------------------------|:------------------------------------------------------------:|:------------------------------------------------------------:|:------------------------------------------------------------:|:------------------------------------------------------------:|
| **Deployments**                                       |                                                              |                                                              |                                                              |                                                              |
| View deployments, backups, certificates, and activity | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| View deployment credentials                           | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Run actions and restart services or instances         | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Hibernate and scale, now or on a schedule             | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Trigger, rerun, and cancel deploy runs                | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Manage IP whitelists and SSH keys                     | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Create, clone, modify, and delete deployments         | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Create and delete backups                             | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Approve patches and set the maintenance window        | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Manage monitoring alert emails                        | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Manage deploy pipeline settings                       | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Import and manage SSL certificates                    | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| **Billing**                                           |                                                              |                                                              |                                                              |                                                              |
| View and pay invoices                                 | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #2da44e">:icon-check-circle-fill:</span> |
| View credit balance and security deposit              | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #2da44e">:icon-check-circle-fill:</span> |
| Manage credit cards and the billing address           | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #2da44e">:icon-check-circle-fill:</span> |
| **Team**                                              |                                                              |                                                              |                                                              |                                                              |
| View team members                                     | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> |
| Invite, remove, and change roles                      | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| **Organization**                                      |                                                              |                                                              |                                                              |                                                              |
| Edit organization details                             | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Manage integrations                                   | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Unsubscribe from organization emails                  | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> |
| Manage email subscriptions for the team               | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Retire the organization                               | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |

## Manage Team Members

Owners and Managers can change someone's role or remove them from the menu at the end of their row.
You can't change your own role.
If you want out, pick **Leave Organization** from the menu on your own row.

The **Receive Invoices** checkbox sends a copy of every invoice email to that team member.
The Owner always gets them, and Owners and Managers can turn it on for anyone else on the team.
