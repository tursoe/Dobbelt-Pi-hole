# Double Pi-hole dashboard

**English** | [Dansk](README.DA.md)

**v1.5.011**

A lightweight dashboard for managing two, three or four Pi-hole v6 instances from one
interface. The solution consists of a single self-contained `index.html` with
no backend, database or build process. The browser communicates directly with
the active Pi-hole instances' REST APIs through an nginx reverse proxy.

The dashboard includes:

- Dashboard with combined statistics and charts
- Overview with instance status and configuration comparisons
- A combined query log with filtering and sorting
- Management of allowed and blocked domains
- Local DNS and CNAME records with differences between instances clearly shown
- Known clients, suggestions for unknown devices and Pi-hole network table cleanup
- Group management and group memberships for domains, clients and adlists
- Adlist management
- Gravity updates with a sequential queue, logs and the time of the last update
- Teleporter backup downloads from selected instances
- Configurable start tab under Settings
- Actions that can be applied to one or all active instances
- Danish, English, Norwegian Bokmål and Nynorsk user interfaces

The Pi-hole instances remain independent DNS servers. This project does not
provide failover, a virtual IP address, a primary or secondary server, or
automatic background replication. Its purpose is to make it easy to maintain
the relevant settings consistently from a single dashboard.

![Pi-hole dashboard](images/dashboard.png)

## Lightweight by design

The dashboard is deliberately simple:

- One HTML file containing HTML, CSS and JavaScript
- No backend
- No database
- No build process
- No Node.js, PHP or Python
- Server count and default names shared across browsers are configured in the HTML file; custom names and interface preferences are stored locally in the browser
- Direct communication with the Pi-hole v6 APIs through nginx

Chart.js is loaded as a separate local file and is only used for charts.

## Requirements

- Two to four Pi-hole **v6** instances with the REST API. The dashboard does not work
  with the old Pi-hole v5 PHP API.
- nginx or a similar reverse proxy in front of the dashboard. Requests to
  `/api1/...` and `/api2/...`, plus `/api3/...` and `/api4/...` when used,
  must be forwarded to their corresponding Pi-hole instances. This avoids CORS issues.
- An app password for each Pi-hole. Create it under **Settings → API** in each
  Pi-hole's own web interface.

### Installation with Docker

Download `compose.yml` and place it in an empty directory. Set the addresses and display names for your Pi-hole instances. Choose HTTP or HTTPS to match your setup and adjust the certificate verification settings.

Open a terminal in that directory and start the dashboard:

```bash
docker compose up -d
```

Docker automatically downloads the prebuilt image containing the dashboard, nginx and Chart.js. Then open `http://YOUR-SERVER-IP:8080`.

When a newer version is available, the Docker edition shows these update
commands under Settings. Run them in the directory containing your `compose.yml`:

```bash
docker compose pull
docker compose up -d
```

See the [Docker project](https://github.com/tursoe/Dobbelt-Pi-hole-Docker) for additional settings and documentation.

## Setup with nginx

1. Copy `index.html` to the directory served by nginx, for example
   `/srv/web/pihole-dashboard/index.html`.
   Also copy `icon.svg`, `favicon-*.png`, `android-chrome-*.png`,
   `apple-touch-icon.png` and `site.webmanifest` from the release to the same directory.
   Copy `language.js` to the same directory to enable German and other contributed languages.
2. Download Chart.js to the same directory:

   ```bash
   cd /srv/web/pihole-dashboard
   curl -o chart.umd.min.js https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js
   ```

3. Copy `nginx.conf.example` to your nginx configuration.
4. Adjust the domain, Pi-hole addresses, certificate paths and web root.
   Set `DEFAULT_SERVER_COUNT` in `index.html` to `2`, `3` or `4`.
   The nginx example enables `/api1/` and `/api2/` by default. Uncomment and
   adjust the `/api3/` block for three instances. For four instances, enable
   and adjust both `/api3/` and `/api4/`.
5. Test and reload nginx:

   ```bash
   nginx -t && systemctl reload nginx
   ```

6. Open the dashboard URL and log in to each active Pi-hole instance with its
   app password.

## Server count and names

The number of active Pi-hole instances is set **only in `index.html`**:

```js
const DEFAULT_SERVER_COUNT = 2; // 2, 3 eller 4
```

The first two, three or four entries in `DEFAULT_SERVERS` are used. The API
paths are `/api1`, `/api2`, `/api3` and `/api4`; configure nginx with a proxy
for each active instance. Actual host names or IP addresses belong in nginx,
not in the browser settings.

Default display names shared across browsers are set using `label` in `DEFAULT_SERVERS`:

```js
const DEFAULT_SERVERS = [
    { id: "1", label: "Pi-hole 1", base: "/api1", color: "c-1", chip: "s1" },
    { id: "2", label: "Pi-hole 2", base: "/api2", color: "c-2", chip: "s2" },
    { id: "3", label: "Pi-hole 3", base: "/api3", color: "c-3", chip: "s3" },
    { id: "4", label: "Pi-hole 4", base: "/api4", color: "c-4", chip: "s4" },
];
```

Server colours are blue, yellow, purple and orange, respectively. The layout
adapts to the configured number and the screen width, up to a maximum page
width of 1900 px.

Names can also be changed under **Settings** for the active instances. These
names are saved in the browser's `localStorage` and override the file's default
names in that browser. The server count and API paths cannot be changed under
Settings. Changing the defaults in the file does not overwrite locally saved
names.

## Features in v1.5.011

### Overview

Overview shows instance status and compares domains, local DNS records,
CNAMEs, clients, groups and adlists. It highlights missing entries and
differences in values or group memberships. Incomplete comparisons are
identified separately. You resolve differences yourself on the relevant tab.

Clicking Overview or Groups fetches data again, even when the tab is already
selected. The view reflects the most recently fetched data and does not
monitor changes in the background.

### Groups

Create groups on one or all active instances using one shared form.
Missing groups can be added to individual instances, and existing groups
can be edited or deleted. The default group cannot be renamed or deleted.
Domains, clients and adlists have their own group membership actions.
Groups are matched by name across instances; their numeric IDs may differ.
Copying a group copies its definition, not its members.

Group membership text and edit buttons are hidden automatically on Domains,
Clients and Lists when all active instances have only the default group (ID 0)
and every entry on the current tab belongs to it. Extra groups, entries without
the default membership or incomplete data keep the group controls visible.
The check covers the complete fetched list, including entries on other pages.
The Groups tab is always visible. Visibility is recalculated when the tab's
data is fetched; hiding the controls does not change group memberships.

### Local DNS and CNAME

Local DNS also supports creating, editing and deleting CNAME records with an
optional TTL and validation for invalid names and CNAME loops. CNAME changes
restart the Pi-hole DNS service. The interface explains the requirements for
the target domain. A direct link ending in `#cnames` opens the CNAME section.

### Gravity and backups

Run Gravity on one or more selected instances. The queue updates one instance
at a time and displays progress, errors and warnings. Follow the DNS checks
and any confirmations shown by the queue. Keep the page open until the queue
finishes. Closing it may leave the current update running, but subsequent
updates will not start.

Overview can download Teleporter backups from selected instances. Each backup
is saved as a separate file; your browser may request permission for multiple
downloads. Restore backups through Pi-hole's own web interface.

### Cleaning up seen clients

Delete beside a seen, unknown device removes its network table entry from the
selected instances. The confirmation lists every affected IP address because
Pi-hole deletes the entire device entry and its associated addresses.
Client configuration and query history are preserved. The device may reappear
if Pi-hole detects it on the network again.

### Start tab

Choose a start tab under Settings. The choice is saved in the current browser.
Dashboard is the default, and a direct link to a tab takes precedence.

## Languages

The user interface is available in **English**, **Dansk**, **Norwegian Bokmål** and **Nynorsk**.
The language selector at the top of the dashboard changes the interface
immediately, including dialog text, dynamic messages, tables and charts.
Numbers, percentages, dates and times follow the selected language.

Roy-Magne Mo contributed the Norwegian Bokmål and Nynorsk translations,
based on commit `8e90275`. His name appears beside the language selector
when either Norwegian language is selected.

The initial language is selected in this order:

1. A supported language previously selected and saved in the browser's
   `localStorage`.
2. The first supported language in the browser's ordered language preferences.
   Regional variants such as `de-DE`, `de-AT`, `en-GB` and `da-DK` are supported.
3. English if no supported language is found.

Danish, English, Norwegian Bokmål and Nynorsk are built into `index.html`. The optional [language.js](language.js)
adds German and can register further contributed languages. Place it beside
`index.html` to enable them. If the file is missing or cannot be loaded, the
dashboard continues with the built-in languages. Unavailable languages are omitted
from the selector. An unavailable saved language falls back to a supported
browser language, then English.

Each language in `window.PIHOLE_LANGUAGES` supplies `label` (the language's native
name), `locale`, `author` and `messages`. Copy the complete English catalog from
`I18N.en` in `index.html`, translate the values and retain every key and named
parameter. Valid contributions are added automatically. Older v1.4.010 packs
remain supported; missing new messages fall back to English. The original
required keys must still be present, and supplied messages must preserve the
correct named parameters. Invalid contributions are skipped. This file cannot
replace the built-in languages. New contributions should include every current key. The template identifies
compatibility-only messages that can retain their English text.

Use [language.template.js](language.template.js) as the template for a new language.
The v1.5.011 template contains all 368 message keys and instructions for
translators. It can also be downloaded under Settings. Keep the keys and
translate only the text values. `language.template.js` is not loaded automatically;
add the completed language to `language.js`. Catalogs use
`.one` for the language's singular category and `.other` for all remaining
numbers, including zero, two, few and many plural categories.

Translations use stable message keys and named parameters. Additional languages
need no backend, build process or extra dependencies. User-supplied values such
as domains, names and comments are never translated.

**Credits:** Daniel Dietz contributed the key-based localization engine and
German translation. This version builds on his work through commit `4a32733`.
His name is also shown beside the language selector when German is selected.
Other translators can be credited through the language's `author` field. Daniel's
existing text in the supplied `language.js` is preserved; the 115 new German
messages for v1.5.011 were added with Google Translate and have not been reviewed by him.

Setup instructions and nginx examples are available in Danish and English:

| Language | Documentation | nginx example |
| --- | --- | --- |
| English | [README.md](README.md) | [nginx.conf.example](nginx.conf.example) |
| Dansk | [README.DA.md](README.DA.md) | [nginx.conf.DA.example](nginx.conf.DA.example) |

## Security

- API and login requests reject HTTP redirects. Configure nginx to proxy directly to the final Pi-hole API URL, including the correct HTTP or HTTPS scheme.
- The dashboard is intended for personal use or for people you trust. Anyone
  who can log in receives administrative access to the connected Pi-hole
  instances.
- Login sessions are stored in the browser's `sessionStorage` and disappear
  when the tab is closed. Passwords are not stored.
- Untrusted values such as client names, domains and comments are escaped
  before being displayed.
- JavaScript is embedded directly in the HTML file. This limits how restrictive
  a Content Security Policy can be. See the comment in
  `nginx.conf.example`.
- When hosting the dashboard on a Synology NAS, a DSM access control profile
  can restrict access to selected IP addresses or local networks.

The dashboard should not be exposed directly to the internet without suitable
access control in front of it.

## Version history

- **v1.1.005:** Complete Danish and English interfaces and documentation
- **v1.2.006:** Support for two, three or four Pi-hole instances
- **v1.3.007:** Language engine rewritten to use message keys, with additional languages in language.js
- **v1.4.008:** Docker image and simplified installation
- **v1.4.010:** Added Norwegian Bokmål and Nynorsk and prepared update checking for Docker installations. Query log time ranges are now limited to 1, 6 and 24 hours.
- **v1.5.011:** Overview, groups and group memberships, CNAME records, Teleporter backups, a sequential Gravity queue, seen client cleanup and a configurable start tab. Expanded language template with English fallback for older language packs.

The dashboard supports two to four active instances. Additional instances
require both the matching server count in `index.html` and a proxy route in nginx.

## Contributing

**Development language:** Source comments and fallback HTML text are maintained in Danish. Translation catalogs keep their respective languages. Existing function names, API fields and message keys are retained for compatibility.

Issues and pull requests are welcome. Contributions may include bug fixes,
suggestions and translations into additional languages. Keep message keys and
named parameters consistent across all language catalogs, and include
translations for static text, dynamic states and accessibility labels.

## License

Copyright © 2023–2026 Christian Kragh Jessen  
https://christiankragh.dk

The project was originally developed for Pi-hole v5 in 2023 and updated for
Pi-hole v6 in 2026.

The project is released under the MIT License and may be used, copied, modified
and redistributed in accordance with its terms. See `LICENSE`.

For manual installations, download Chart.js separately as described in the
setup instructions. The Docker image includes Chart.js. Chart.js is an independent third-party project with its
own license.
