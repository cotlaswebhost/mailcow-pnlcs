# PNLCS Mailcow Extended Module

A standalone, shippable PNLCS server module for selling Mailcow email hosting. No core PNLCS modifications required.

## Features

### Client Area (No Mailcow Login Required)
- **Dashboard**: View total mailboxes, total storage, used storage (per-mailbox breakdown).
- **Mailbox Management**: Create, delete, reset password for mailboxes within purchased slots.
- **Alias Management**: Create/delete aliases, assign aliases to mailboxes (forwarding).
- **Storage Add-on Management**: Purchase extra storage (GB) as an add-on. Allocate purchased extra GB to specific individual mailboxes. Cannot decrease base storage.
- **Self-Service**: All operations performed inside PNLCS client area.

### Admin Area
- **Provisioning**: Auto-create Mailcow domain + domain admin on order acceptance.
- **Suspend/Unsuspend/Terminate**: Standard lifecycle.
- **Change Package**: Adjust base mailbox count and base storage per mailbox.
- **Usage Sync**: Hourly sync of mailbox count and disk usage from Mailcow for overage billing.

### Billing Logic (Core Differentiator)
- **Base Plan**: Sell N inboxes × M GB base storage each.
  - Total domain quota = N × M GB (assigned in Mailcow, **not visible to client**).
  - Client sees: "5 inboxes, 20 GB each" = 100 GB total usable base.
- **Storage Add-on**: Client purchases extra storage (e.g., +60 GB).
  - This increases the **domain total quota** in Mailcow (100 → 160 GB).
  - The extra 60 GB is held as an **allocatable pool**.
  - Client can allocate from this pool to **any individual mailbox**.
  - **Constraint**: Base storage per mailbox is fixed. Client cannot reduce base allocation to "game" the system. If they want 40 GB on one inbox, they must purchase add-on storage.
- **Example**: Client has 20 inboxes × 20 GB = 400 GB domain quota.
  - They purchase +40 GB add-on. Domain quota becomes 440 GB.
  - They can allocate +20 GB to inbox A, +20 GB to inbox B. they cannot allocate 40 gb to one inbox and increase its storage, he has purchased addon of 20gb for two inboxes, so he can only use 20 gb each to increase two inboxes size, he can not assign 10 gb to one and 30 gb to another, or 40 gb to one. addon is per inbox, not per domain.
  - They cannot reduce inbox C from 20 GB to 5 GB to free up 15 GB for another inbox.

## Installation

1. Copy the module folder to `modules/Servers/MailcowExtended/` in your PNLCS installation.
2. Ensure `pnlcs.json` is present.
3. Go to **Setup → Modules** and enable "Mailcow Extended".
4. Go to **Setup → Servers** and add a new server with type "Mailcow Extended".
   - Hostname: `mail.yourdomain.com` (no https:// prefix)
   - API Key: Mailcow read-write API key (from Mailcow admin → Configuration → Access → API)
5. Create a product and select "Mailcow Extended" as server module.
6. Configure product options:
   - `base_mailboxes`: Number of inboxes in plan
   - `base_storage_gb`: Base storage per inbox (GB)
   - `max_mailboxes`: Maximum inboxes allowed (for upgrades)
   - `webmail_url`: URL to webmail (e.g., `https://mail.yourdomain.com/SOGo/`)

## Development Notes

- **Reference**: `modules/Servers/` in the PNLCS repository for existing module structure (e.g., cPanel, Plesk).
- **Mailcow API**: `https://<mailcow>/api/v1/`. Auth via `X-API-Key` header.
  - Create domain: `POST /api/v1/add/domain`
  - Create mailbox: `POST /api/v1/add/mailbox`
  - Edit domain quota: `POST /api/v1/edit/domain`
  - Edit mailbox quota: `POST /api/v1/edit/mailbox`
  - List mailboxes: `GET /api/v1/get/mailbox/all/<domain>`
  - Create alias: `POST /api/v1/add/alias`
- **PNLCS Module Interface**: Implement `ServerModuleInterface`. See `docs.pnlcs.com/concepts/servers-and-modules/`.
- **Screenshots**: See `/screenshots/` folder for UI reference (provided in repo).
- **Testing**: Use Mailcow demo instance (`demo.mailcow.email`) with public API key for development.

## File Structure
MailcowExtended/
├── pnlcs.json
├── MailcowExtendedModule.php
├── lib/
│ ├── MailcowApiClient.php
│ └── QuotaManager.php
├── templates/
│ ├── clientarea.tpl
│ ├── adminarea.tpl
│ └── css/
│ └── style.css
└── lang/
└── en.php

## License

MIT (consistent with PNLCS)

