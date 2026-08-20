# TaskFlow Product Scope

## 1. Product Overview

TaskFlow is a project and task management REST API designed for teams that
need to organize projects, collaborate on tasks, assign responsibilities, and
track delivery progress.

The first version provides a secure backend API. A web or mobile frontend may
be added later as a separate client.

## 2. Business Goals

- Centralize project and task information.
- Make ownership and responsibility visible.
- Allow project managers to organize project teams.
- Allow members to track and update their assigned work.
- Protect project data from unauthorized access.
- Provide a maintainable foundation for future product capabilities.

## 3. Target Users

### System Administrator

Responsible for platform-level administration, user management, and operational
oversight.

### Project Manager

Responsible for creating and managing projects, organizing project membership,
and coordinating tasks.

### Project Member

Participates in projects, works on assigned tasks, and collaborates through
task comments.

## 4. Version 1 Scope

### 4.1 Authentication and Users

- Register a user account.
- Authenticate using email and password.
- Issue JWT access tokens.
- Store passwords using a secure one-way password hash.
- Apply global role-based authorization.
- Allow administrators to manage user account status.

### 4.2 Projects

- Create projects.
- View accessible projects.
- View project details.
- Update project information.
- Archive projects.
- List projects with pagination and sorting.

### 4.3 Project Membership

- Add users to a project.
- Remove users from a project.
- List project members.
- Prevent duplicate memberships.
- Restrict membership management to authorized users.

### 4.4 Tasks

- Create tasks inside a project.
- View project tasks.
- Update task details.
- Assign tasks to project members.
- Track task status and priority.
- Define and update due dates.
- Filter, search, sort, and paginate tasks.

### 4.5 Comments

- Add comments to tasks.
- View task comments.
- Associate every comment with its author.
- Restrict comments to members of the related project.

### 4.6 Auditability

Core records will store creation and update timestamps. Where applicable, the
system will also record the user responsible for creating or updating a record.

## 5. Out of Scope for Version 1

- Frontend web or mobile application.
- Email and push notifications.
- Real-time updates.
- File attachments.
- Custom task workflows.
- Time tracking.
- Billing and subscriptions.
- Multi-tenant organizations.
- Social login.
- Single sign-on.
- User-configurable roles and permissions.
- Permanent physical deletion of projects through the public API.

These capabilities may be evaluated for later releases.

## 6. Authorization Model

TaskFlow uses two authorization layers:

1. A global role describes platform-level capabilities.
2. Project membership determines access to a specific project.

A global role does not automatically make a user a member of every project.
Project resources require both authentication and the appropriate
project-level relationship unless an explicitly documented administrative
operation applies.

## 7. Global Roles

### ADMIN

An administrator can:

- View and manage user account status.
- Assign supported global roles.
- Perform explicitly defined platform administration operations.
- Access operational endpoints when permitted.

The ADMIN role does not silently grant participation rights inside every
project. Any exceptional administrative access must be explicit, auditable,
and implemented as a dedicated operation.

### PROJECT_MANAGER

A project manager can:

- Create projects.
- Manage projects they own or are authorized to manage.
- Add and remove members from managed projects.
- Create, update, and assign tasks within managed projects.
- Archive managed projects.

The PROJECT_MANAGER role does not grant access to unrelated projects.

### MEMBER

A member can:

- View projects in which they have an active membership.
- View tasks inside those projects.
- Update permitted fields on assigned tasks.
- Add comments to tasks within those projects.

The MEMBER role does not grant project creation or membership-management
permissions.

## 8. Authorization Matrix

| Capability | ADMIN | PROJECT_MANAGER | MEMBER |
|---|---:|---:|---:|
| Register and authenticate | Yes | Yes | Yes |
| View own profile | Yes | Yes | Yes |
| Manage user account status | Yes | No | No |
| Assign a global role | Yes | No | No |
| Create a project | No | Yes | No |
| View an accessible project | If a member | If authorized | If a member |
| Update project details | If designated manager | If designated manager | No |
| Archive a project | If designated manager | If designated manager | No |
| Manage project membership | If designated manager | If designated manager | No |
| View project members | If a member | If authorized | If a member |
| Create a project task | If authorized | If designated manager | No |
| View project tasks | If a member | If authorized | If a member |
| Update task details | If authorized | If designated manager | Limited |
| Change task status | If authorized | If designated manager | If assigned |
| Assign a task | If authorized | If designated manager | No |
| Add a task comment | If a member | If authorized | If a member |
| View task comments | If a member | If authorized | If a member |

`If authorized` means the user must have an explicit relationship with the
project, such as active membership or designation as its manager.

## 9. Project-Level Rules

- Every project has one designated manager.
- The project manager must have an active project membership.
- Only active project members can access project content.
- Only the designated manager can manage project membership.
- A task can be assigned only to an active member of its project.
- A user removed from a project immediately loses access to its protected
  resources.
- A member can update the status of a task assigned to them.
- A regular member cannot change task ownership, project membership, or project
  settings.
- Comments remain associated with their original authors for audit purposes.
- Archived projects are read-only except for explicitly permitted
  administrative operations.

## 10. Important Assumptions

- Version 1 supports one designated manager per project.
- A user has one global role at a time.
- Project membership is separate from the global role.
- Project removal deactivates membership rather than destroying its history.
- Project deletion is implemented as archival in version 1.
- Task and project history must not be silently destroyed.
- Authorization is enforced at both the HTTP endpoint and service layers.
- Resource ownership and membership checks occur on the server.
- Client-provided roles, user identifiers, and ownership claims are never
  trusted without server-side verification.

## 11. Success Criteria

Version 1 is successful when authenticated teams can securely manage projects,
members, tasks, assignments, and comments through a documented REST API, while
users remain unable to access projects outside their authorization boundary.
