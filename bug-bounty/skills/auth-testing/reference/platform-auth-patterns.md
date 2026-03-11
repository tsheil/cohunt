# Platform-Specific Authorization Bypass Patterns

> Referenced from [auth-testing SKILL.md](../SKILL.md) — detailed testing procedures for WordPress/CMS plugin auth bypass and Salesforce Experience Cloud guest user access control. Load when the target runs WordPress, Drupal, Joomla, or Salesforce Experience Cloud.

## Table of Contents

- [WordPress/CMS Plugin Authorization Bypass](#wordpresscms-plugin-authorization-bypass)
- [Salesforce Experience Cloud / Aura Guest User Access Control](#salesforce-experience-cloud--aura-guest-user-access-control)

---

## WordPress/CMS Plugin Authorization Bypass

WordPress plugins are a top bug bounty target. The most common pattern: AJAX handlers registered via `wp_ajax_nopriv_*` missing `current_user_can()` checks, allowing unauthenticated access to admin functionality.

**Key CVE:** CVE-2026-2371 (Greenshift plugin <= 12.8.3, 100K+ installs) — `gspb_el_reusable_load()` AJAX handler accepts arbitrary `post_id`, renders private/draft reusable blocks without checking `current_user_can('read_post')`. Nonce exposed on public pages via `ajax="1"` shortcode.

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Enumerate AJAX handlers | `grep -r 'wp_ajax_nopriv_' wp-content/plugins/` | Handlers accessible without login |
| 2 | Missing capability check | Call AJAX handler with `action=handler_name` — no cookies | Data returned without auth (private posts, settings, user data) |
| 3 | Nonce bypass | Check if nonce is exposed on public pages or predictable | AJAX calls succeed with leaked/missing nonce |
| 4 | Post status leak | Request private/draft/password-protected content IDs | Content rendered regardless of visibility status |

**Applies to:** WordPress, Drupal (route access callbacks), Joomla (component ACL), any CMS with plugin-registered endpoints. **Severity:** Medium-High (private content disclosure) to Critical (admin functionality exposure).

### Drupal-Specific Patterns

```
[] Route access callbacks — Check if custom routes use _access: 'TRUE' (permissive) vs proper permission checks
[] REST resource permissions — /jsonapi/* and /api/* endpoints may bypass node access restrictions
[] Entity access hooks — Custom entities missing hook_entity_access() implementations
[] Update module exposure — /update.php accessible without proper auth on misconfigured sites
```

### Joomla-Specific Patterns

```
[] Component ACL bypass — Test com_* components for missing JAccess checks
[] API endpoints — /api/index.php/v1/* endpoints may have weaker auth than web UI
[] Extension installer — Can non-admin users access extension management?
```

---

## Salesforce Experience Cloud / Aura Guest User Access Control

Salesforce Experience Cloud sites expose the `/s/sfsites/aura` endpoint for the Lightning Aura framework. When guest user profiles are misconfigured with overly broad permissions, unauthenticated users can query CRM objects (Accounts, Contacts, Cases, Documents) directly. **ShinyHunters (UNC6240) breached ~400 organizations** in March 2026 using this pattern — no CVE assigned (Salesforce classifies as customer misconfiguration).

**Discovery:** Send empty POST `{}` to `/s/sfsites/aura` — `aura:invalidSession` response confirms Aura instance. Check custom path prefixes: `/business/s/sfsites/aura`, `/support/s/sfsites/aura`.

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Object enumeration | POST `getConfigData` action to Aura endpoint | `apiNamesToKeyPrefixes` reveals all accessible objects; `__c` suffix = custom objects |
| 2 | Unauthenticated data access | POST `getItems` with `entityNameOrId` for each discovered object | Response >12KB indicates data exposure; records returned without auth |
| 3 | Custom Apex controller | Search proxy history for `compound://c` and `Apex@` strings | Undocumented business logic accessible to guest users |
| 4 | Record limit bypass | Test `sortBy` parameter against 2,000-record GraphQL limit | Bypass returns full dataset beyond intended limit |
| 5 | Self-registration escalation | Register via guest self-registration if enabled | Elevated access beyond guest-tier permissions |

**Severity:** Critical when PII or CRM data exposed (Salesforce paid $6.2M in bounties in 2025, up to $60K/finding). **Tools:** AuraInspector (Mandiant), Misconfig Mapper (Intigriti), poc_salesforce_lightning. **Key misconfiguration:** "API Enabled" permission on guest user profile opens the attack vector — disabling it closes it.

### ServiceNow-Specific Patterns

```
[] Table ACL bypass — Check sys_* tables via /api/now/table/ for overly permissive ACLs
[] Glide record exposure — Direct table queries returning records across scopes
[] Widget data exposure — Service Portal widgets exposing backend data without auth checks
```
