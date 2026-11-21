# MsBookshelf

**A Multi-Library Management System**

MsBookshelf is a multi-tenant library management platform for managing
independent libraries, branches, staff, members, book collections, and
circulation from one platform.

> **Project status:** Responsive frontend foundation / UI concept.
> Current dashboard records are sample data. Authentication, Google
> Drive integration, and production database authorization must be
> implemented before real-world use.

## Features

-   Multi-library platform with tenant isolation.
-   Multiple branches under each library.
-   Role-based workspaces for platform and library staff.
-   Book catalogue separated from physical book-copy inventory.
-   Manual book issue and return workflows.
-   Google Drive storage for book covers, PDFs, scanned documents, and
    other library files.
-   Responsive desktop, tablet, and mobile interface.
-   Light Modern SaaS design with collapsible navigation.

## Technology

  -----------------------------------------------------------------------
  Technology                          Responsibility
  ----------------------------------- -----------------------------------
  Next.js App Router                  Web application and server-side
                                      endpoints

  React + TypeScript                  Frontend and application logic

  Tailwind CSS                        Responsive styling

  Lucide React                        Icons

  Turso / libSQL                      Application database

  Google OAuth 2.0                    Google account connection and
                                      sign-in, if selected

  Google Drive API                    File storage for images and PDFs

  Vercel or another Node-compatible   Application deployment
  host                                
  -----------------------------------------------------------------------

**Supabase is not part of this architecture.**

## Roles and access model

MsBookshelf separates the platform role, library membership, and library
data. A library is a tenant/workspace, not a user role.

  -----------------------------------------------------------------------
  Role                    Scope                   Responsibilities
  ----------------------- ----------------------- -----------------------
  Super Admin             Entire platform         Approve/suspend
                                                  libraries, manage
                                                  platform settings,
                                                  oversee usage and
                                                  platform-level access

  Library Owner           One library             Own the library
                                                  workspace, manage its
                                                  configuration, appoint
                                                  or replace its Manager

  Manager                 One library             Manage branches, staff,
                                                  books, copies, members,
                                                  circulation and library
                                                  reports

  Librarian               Assigned library/branch Manage catalogue,
                                                  members, and book
                                                  issue/return operations
                                                  as permitted

  Helper                  Assigned library/branch Perform limited daily
                                                  tasks according to
                                                  granted permissions

  Member                  One library membership  View their own
                                                  membership and loan
                                                  information
  -----------------------------------------------------------------------

### Role rules to enforce

-   A user may have memberships in more than one library.
-   Each library has its own independent data and staff assignments.
-   Each library may have multiple branches.
-   Branch-level staff access must be limited to authorized branches.
-   The platform should enforce one active Manager per library if that
    remains the business rule.
-   The maximum number of Helpers should be configurable per library.
-   Permissions must be checked on the server for every protected
    operation.
-   The frontend must never decide access based only on a role value
    supplied by the browser.

The exact relationship between Library Owner and Manager can be
finalized before implementing staff invitations and permissions. If a
library owner should also act as its manager, represent that explicitly
rather than silently granting two roles.

## Google Drive file storage

Files are stored in the Google Drive account connected by the library or
its authorized owner.

Examples: - Book cover images - PDF books and reference documents -
Scanned forms and certificates - Library documents and reports

### Recommended flow

1.  A Library Owner or authorized Manager connects a Google account
    using Google OAuth.
2.  The user grants the required Google Drive permissions.
3.  The server securely stores the OAuth refresh token.
4.  An authorized user uploads a file through the MsBookshelf
    application.
5.  The server uploads the file to the connected Drive account.
6.  Turso stores the Drive file ID, library ID, branch ID (if
    applicable), MIME type, name, size, and timestamps.
7.  When a user requests a file, the server verifies library and branch
    permissions before providing access.

### Important Drive considerations

-   A Gmail address alone does not grant API access to Google Drive. The
    account must authorize the application through OAuth.
-   Store files in a dedicated MsBookshelf folder in the connected Drive
    account.
-   Prefer private files and server-mediated access. Do not make
    sensitive library documents public by default.
-   Use the minimum practical Google Drive OAuth scopes.
-   Handle revoked consent, expired access tokens, deleted files, quota
    limits, and upload failures.
-   Define who owns the Drive connection: the library owner, the library
    organization, or another designated account.
-   Do not assume that a user's personal Gmail Drive has unlimited
    storage or is suitable for permanent institutional records.

## Turso database and security

Turso/libSQL stores application data and file metadata. It does not
replace the application's user authentication and authorization system.

### Database design principles

-   Use UUIDs as application identifiers.
-   Store `library_id` on every library-owned record.
-   Store `branch_id` where records are branch-specific.
-   Keep the book catalogue separate from physical book copies.
-   Use foreign keys and database constraints as additional integrity
    protections.
-   Store timestamps in UTC using ISO 8601.
-   Use server-side transactions for multi-step operations where
    appropriate.

### Credentials and secrets

Configure secrets only in the deployment environment or local
`.env.local` file.

Example `.env.example`:

``` env
TURSO_DATABASE_URL=
TURSO_AUTH_TOKEN=

GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_REDIRECT_URI=

APP_URL=
SESSION_SECRET=
```

Never commit real credentials. Never expose `TURSO_AUTH_TOKEN`,
`GOOGLE_CLIENT_SECRET`, Google refresh tokens, or `SESSION_SECRET` in
browser code or `NEXT_PUBLIC_` variables.

If a database token is accidentally shared or committed, revoke it and
issue a replacement immediately.

### Authentication is a separate decision

Turso provides database connectivity, not end-user sign-in sessions.
Choose and implement an authentication method, for example:

-   Google OAuth sign-in, if Google accounts are acceptable for all
    users; or
-   Email/password authentication implemented securely by the
    application.

In either case, the application must create secure sessions, verify
users on the server, and load roles and permissions from trusted Turso
records. Google Drive authorization and application sign-in should be
treated as separate permissions, even if both use the same Google
account.

## Getting started

### Requirements

-   Node.js 20.9 or later
-   npm

### Install

``` bash
git clone <your-repository-url>
cd msbookshelf
npm install
```

### Run locally

``` bash
npm run dev
```

Open <http://localhost:3000>.

### Production build

``` bash
npm run build
npm start
```

## Project structure

``` text
msbookshelf/
├── src/
│   ├── app/
│   │   ├── globals.css
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── components/
│   │   └── dashboard-shell.tsx
│   └── lib/
│       └── dashboard-data.ts
├── .env.example
├── package.json
├── postcss.config.mjs
├── tsconfig.json
└── README.md
```

## Current frontend status

Implemented in the frontend foundation: - Super Admin, Manager, and
Helper dashboard previews. - Responsive collapsible navigation. - Sample
metric cards, charts, activity feed, and tables. - Demo role switching,
search, notifications, and UI feedback.

Not yet implemented: - Production user authentication and session
management. - Turso connection and server-side data access. - Library
Owner, Librarian, and Member protected experiences. - Google OAuth
connection and Google Drive upload/download. - Server-side tenant and
branch authorization. - Production CRUD workflows, audit logging, and
deployment configuration.

The demo role selector is not security and must not be used to authorize
real users.

## Roadmap

-   [x] Responsive UI foundation
-   [x] Initial Super Admin, Manager, and Helper dashboard previews
-   [ ] Finalize complete role and permission matrix
-   [ ] Implement authentication and secure sessions
-   [ ] Connect Turso/libSQL
-   [ ] Implement tenant and branch authorization
-   [ ] Library onboarding and approval
-   [ ] Library Owner and staff management
-   [ ] Google OAuth and Drive connection
-   [ ] Secure image and PDF upload/download
-   [ ] Book catalogue and physical copy inventory
-   [ ] Member registration and membership management
-   [ ] Manual issue and return workflows
-   [ ] Reports and audit logs
-   [ ] Production deployment and monitoring

## License

No license has been selected. Add a LICENSE file before distributing the
project or accepting external contributions.

------------------------------------------------------------------------

**MsBookshelf --- One platform. Independent libraries. Connected
management.**
