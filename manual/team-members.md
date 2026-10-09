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
| View deployments, backups, certificates, and activity | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Run actions and restart services                      | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Trigger, rerun, and cancel deployment runs            | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Start, stop, and restart instances                    | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Change auto scaling group capacity                    | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Hibernate and wake up deployments                     | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Schedule hibernation                                  | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Schedule scaling                                      | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Update IP whitelists                                  | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Add and remove SSH keys                               | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Create, clone, and delete deployments                 | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Rename and modify deployments                         | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Create and delete backups                             | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Approve or reject patches                             | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Change the maintenance window                         | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Manage alert subscriptions and dismiss notifications  | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Connect a repository and change deploy settings       | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Import and manage SSL certificates                    | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Connect the GitHub integration                        | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| View and submit partnership leads                     | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Edit organization details                             | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Change the support plan                               | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Update the billing address                            | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #2da44e">:icon-check-circle-fill:</span> |
| View and pay invoices                                 | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #2da44e">:icon-check-circle-fill:</span> |
| Manage credit cards                                   | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #2da44e">:icon-check-circle-fill:</span> |
| View credit balance and security deposit              | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #2da44e">:icon-check-circle-fill:</span> |
| View team members                                     | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> |
| Invite, remove, and change the role of team members   | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |
| Retire the organization                               | <span style="color: #2da44e">:icon-check-circle-fill:</span> | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     | <span style="color: #cf222e">:icon-x-circle-fill:</span>     |

## Manage Team Members

Owners and Managers can change someone's role or remove them from the menu at the end of their row.
You can't change your own role.
If you want out, pick **Leave Organization** from the menu on your own row.

The **Receive Invoices** checkbox sends a copy of every invoice email to that team member.
The Owner always gets them, and Owners and Managers can turn it on for anyone else on the team.
