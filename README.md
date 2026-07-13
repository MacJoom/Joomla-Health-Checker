# Health Check Suite for Joomla 6

A backend **Health Check dashboard** for Joomla 6, plus a **PHP Scanner** security add‑on that fills
it with malware / integrity / file‑hygiene findings.

The two parts are independent and shipped as separate installable packages:

| Package | Version | Contains | What it does |
|---|---|---|---|
| **`pkg_healthcheck`** | 6.2.1 | `mod_healthcheck` (admin module) + `plg_system_healthcheckmenu` (system plugin) | The dashboard UI and its **Health Check** sidebar menu entry. Self‑contained — works on stock Joomla 6. |
| **`pkg_phpscanner`** | 1.0.0 | `plg_healthcheck_phpscanner` · `plg_system_phpscannerbaseline` · `plg_task_phpscannermalware` | Feeds the dashboard with security findings and runs the heavy full‑site scans on a schedule. |

The dashboard on its own is empty; the PHP Scanner (or any other `healthcheck`‑group plugin) provides
the data. Install **both** for the full experience.

---

## Requirements

- **Joomla 6.0 or later** (tested on 6.1.x / 6.2).
- **PHP 8.1 or later** (Joomla 6 minimum).
- Backend (administrator) access as a Super User.

---

## Files in this repository

This repository holds the ready‑to‑install `.zip` files. For a normal install you only need the two
**`pkg_…`** bundles; the individual module/plugin zips are provided in case you want to install or
update a single component.

| File | Install this? |
|---|---|
| `pkg_healthcheck-6.2.0.zip` | **Yes** — the Health Check UI (module + menu plugin). |
| `pkg_phpscanner-1.0.0.zip` | **Yes** — the PHP Scanner (all three plugins). |
| `mod_healthcheck-6.2.0.zip` | Optional — just the dashboard module. |
| `plg_system_healthcheckmenu-6.2.0.zip` | Optional — just the sidebar‑menu plugin. |
| `plg_healthcheck_phpscanner-1.0.1.zip` | Optional — just the scanner findings plugin. |
| `plg_system_phpscannerbaseline-1.0.0.zip` | Optional — just the baseline system plugin. |
| `plg_task_phpscannermalware-1.0.1.zip` | Optional — just the scheduled‑scan task plugin. |
| `plg_healthcheck_usermaintenance-6.2.0.zip` | Optional — a standalone check plugin surfacing user‑account hygiene (see below). |

## Installation

Download the package `.zip` files from this repository — click a file and use **Download raw file**,
or use **Code → Download ZIP** and take the packages out of the extracted folder. Then, in the Joomla
backend, go to **System → Install → Extensions → Upload Package File** and drop each package in.

Install the two bundles in this order:

### 1. Health Check UI — `pkg_healthcheck-6.2.0.zip`

Installs the dashboard module and the menu plugin. On install it **configures itself**:

- a published **Health Check** module instance is created in the `cpanel-healthcheck` dashboard
  position, and
- the **System – Health Check Menu** plugin is enabled, adding a **Health Check** entry (heart icon)
  to the left admin sidebar.

After install, click **Health Check** in the sidebar. You'll see the dashboard with an *empty state*
("All good! Nothing to report") until a scanner plugin is added in step 2.

### 2. PHP Scanner — `pkg_phpscanner-1.0.0.zip`

Installs the three scanner plugins. **They install disabled** (Joomla installs plugins disabled by
default), so you must enable them — see configuration below.

---

## Post‑install configuration

### Enable the scanner plugins

Go to **System → Manage → Plugins** and enable:

| Plugin | Required? | Purpose |
|---|---|---|
| **Health Check – PHP Scanner** (`healthcheck` group) | **Yes** | Provides the findings shown on the dashboard (malware, oversized files, files outside a regular install, integrity, extension list, PHP‑in‑content, …). Without it the dashboard stays empty. |
| **System – PHP Scanner Baseline** | Recommended | After any extension install/update or core update, flags the file‑integrity baseline for rebuild so legitimate changes aren't later reported as tampering. |
| **Task – PHP Scanner Malware** | Recommended | Runs the heavy full‑site scans on a schedule (see below) instead of during page loads. |

### Set up the scheduled scans

The full‑site walks are expensive, so run them from Joomla's Scheduler rather than on page render.

1. Go to **System → Manage → Scheduled Tasks → New**.
2. Create a task using **PHP Scanner: filesystem scans** and give it a schedule (e.g. daily). This
   caches the malware / oversized / recent‑files / integrity results for the dashboard.
3. (Optional) Create a second task using **PHP Scanner: rebuild integrity baseline** to establish or
   refresh the tamper‑detection baseline. **Run this only on a known‑clean site or right after
   legitimate updates.**

> **The Scheduler needs a trigger.** Joomla runs due tasks via the *lazy scheduler* (on regular
> backend page loads) or, better, via a real cron entry calling
> `php /path/to/site/cli/joomla.php scheduler:run`, or a web‑cron hitting the scheduler URL. See
> Joomla's *Scheduled Tasks* documentation.

### Establish the integrity baseline (first run)

The integrity check only reports differences once a baseline exists. Either run the **rebuild
integrity baseline** task once, or use the baseline action offered by the PHP Scanner plugin on a
clean site. Do this **before** relying on integrity findings.

---

## Using it

Open the **Health Check** dashboard from the sidebar (or directly at
`index.php?option=com_cpanel&view=cpanel&dashboard=healthcheck`). Each check loads in the background
and shows a count; click a check to list the matching items. Findings include, among others:

- **Suspicious PHP files (possible malware)** — webshell / obfuscation signatures.
- **Oversized PHP files (not scanned)** — files too large to scan (>5 MB); a very large `.php` is
  unusual and worth reviewing.
- **PHP files outside a regular install** — orphans or files changed after their extension was
  installed.
- **File integrity** — differences against the SHA‑256 baseline.
- Installed extensions list, PHP‑in‑content and deprecated‑API checks, and more.

---

## Configuration options

Most scan toggles, signature lists and exclude lists live in the **Health Check – PHP Scanner** plugin
parameters (**System → Manage → Plugins →** *Health Check – PHP Scanner* **→ Options**). The scheduled
task reuses those same settings so the pre‑warmed caches match the on‑demand scan.

---

## Organising the dashboard into panels

There is **one** Health Check dashboard (the **Health Check** sidebar link, position
`cpanel-healthcheck`). You arrange it into separate **panels** by placing **several Health Check
module instances** in that one dashboard — each module is a panel that shows one or more plugins.
Joomla renders every module assigned to a position, so the modules stack as distinct panels on the
same page.

**How the pairing works.** A Health Check *module* has a **Health Check Group** value; each Health
Check *plugin* has a **Dashboard Context** value. A plugin's checks appear in a module **only when the
two values are identical.** Both default to `general`, so with no configuration a single `general`
module shows every enabled plugin.

### Recipe

1. **Pick a group name per panel** — e.g. `security` (PHP Scanner) and `users` (User Maintenance).

2. **For each panel, add a module.** *System → Manage → Administrator Modules → New → Health Check*:
   - **Title:** the panel heading, e.g. `Security` (keep **Show Title** on).
   - **Position:** `cpanel-healthcheck` — the **same** position for every panel.
   - **Health Check Group** (Basic tab): the panel's group name, e.g. `security`.
   - **Ordering:** controls the panel's vertical order on the dashboard.
   - **Status:** Published — then Save.

3. **Point each plugin at its panel's group.** *System → Manage → Plugins →* open the plugin, set
   **Dashboard Context** to that group, and enable it:
   - **Health Check – PHP Scanner** → `security`
   - **Health Check – User Maintenance** → `users`

   A group can contain **more than one plugin** — just give every plugin you want in a panel the same
   Dashboard Context as that panel's module.

4. Open the **Health Check** dashboard: the `security` module shows the PHP Scanner checks, the
   `users` module shows the User Maintenance checks — two panels, one dashboard.

### Notes

- Installing the Health Check package auto‑creates **one** module (group `general`) at
  `cpanel-healthcheck`. Repurpose it as your first panel (rename it and set its group), or unpublish /
  delete it once you switch to named groups — otherwise it stays as an empty `general` panel.
- All Health Check panels live at position **`cpanel-healthcheck`**; the number of panels is just the
  number of module instances there.
- A plugin's **Dashboard Context** is a single value, so a plugin belongs to one panel at a time. To
  split a plugin's checks across panels you would need the checks in separate plugins.
- Leaving a module's group and the plugins' context at `general` (the defaults) gives a single
  all‑in‑one panel with no configuration.

## Uninstalling

Uninstall from **System → Manage → Extensions**. Uninstall the packages (`Health Check`, `PHP
Scanner`) to remove all their contained extensions. The menu plugin removes its sidebar entry on
uninstall; the module's auto‑created instance is removed with the module.

---

## Notes

- **Self‑contained module.** `mod_healthcheck` ships its own HTMLHelper service and layouts and
  registers them only when Joomla core doesn't already provide them, so it installs and renders on a
  stock Joomla 6 site.
- **Memory‑safe scanning.** The malware scan skips files larger than 5 MB (they would exhaust memory
  and are never legitimate source) and reports them under *Oversized PHP files* instead. Integrity
  hashing streams files, so it is unaffected.
- **License:** GNU General Public License version 2 or later.
