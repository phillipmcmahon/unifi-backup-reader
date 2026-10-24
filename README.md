# UniFi Backup Reader

**Version 1.1.2 | Access edition**

A self-contained, read-only browser application for inspecting supported UniFi PostgreSQL and legacy BSON backups. Open `index.html`, select a backup and browse its saved networks, Wi-Fi settings, devices, ports, clients, firewall rules and services.

This edition uses the Access visual design, including the approved fox logo and sidebar layout from Device Access. It is adapted from the original UniFi Backup Reader by [AyeTech](https://ayetech.au).

Backup processing runs locally in the browser. The application does not connect to your controller or show live network status.

## Supported inputs

Version 1.1.2 reads **PostgreSQL (`postgres_v1`) and legacy BSON Network data** inside supported backup containers:

| Input | Support |
| --- | --- |
| `.unifi` | Supported encrypted UniFi OS console backups containing supported PostgreSQL or BSON Network dumps |
| `.unf` | Supported encrypted UniFi Network backups containing supported PostgreSQL or BSON dumps |
| `.zip` | Extracted backup archives containing PostgreSQL `db-*.jsonl[.gz]` dumps or legacy BSON `db.gz` |
| `.jsonl` | UniFi PostgreSQL JSONL export streams |
| `.gz` | Gzip-compressed PostgreSQL JSONL or BSON streams, or supported backup TAR archives |
| `.bson` | UniFi BSON command streams, BSON documents and supported mongodump archive streams |

A `.unf` or `.unifi` extension alone does not establish compatibility. The reader inspects the payload format. Legacy BSON support includes UniFi collection-selection command streams and the mongodump archive layout. Malformed records are rejected rather than silently skipped.

Use the original database filename, such as `db-config.jsonl.gz`, where possible. If a standalone JSONL file has been renamed, or a raw BSON file has no database/collection marker, and its database cannot be identified, the application asks whether it contains Network configuration or another database. Other databases are listed in Coverage without being presented as an empty network.

The input file limit is **512 MiB**. Archive and decoded-data limits also apply. Browser memory may impose a lower practical limit.

## Use the reader

1. Download this repository's `index.html` and open it in your browser. No package installation, build step or local web server is required.
2. Choose **Open backup**, use the welcome screen's file picker, or drop a file onto the upload area.
3. Confirm the backup filename, capture time and selected site shown above the content.
4. Choose a section from the navigation. Search and filter the current view, then open a record for details and related items.
5. Reveal individual secrets only when needed. Use **Copy table** to paste matching rows into a spreadsheet or ticket, or **Export view** to save CSV or JSON.
6. Choose **Clear session** when finished.

Opening another backup replaces the current data only after the new file has been successfully decoded and, where required, identified. Failed or cancelled opening retains the current backup.

## Views and controls

| Section | Information available when recorded in the backup |
| --- | --- |
| Overview | Site-scoped network counts, local network summary and backup information |
| Networks | Local networks, VLANs, subnets, gateways, DHCP ranges, DNS settings, internet definitions and static DNS records |
| Wi-Fi | SSIDs, security settings, network associations, bands and individually protected credentials |
| Devices | Device inventory, port profiles, effective port configuration, recorded radio and interface details, LLDP neighbours and SSID broadcasts when present |
| Clients | Recorded clients and DHCP reservations |
| Firewall | Zone-based user policies, zones, address and port groups, port forwards, switch rules, NAT rules, separate system rule definitions and legacy rules |
| VPN & Services | VPN networks and peers, recorded VPN sessions, traffic and static routes, dynamic DNS, encrypted DNS, DHCP options, RADIUS, filtering, schedules and device supervision |
| System | Settings, administrators, sites, console/application metadata, archive inventory, All tables and Coverage |

- **Table sorting:** select a sortable column heading to change its order. VLANs sort numerically, with untagged entries first in ascending order. Untagged is not automatically relabelled as VLAN 1.
- **Site and view filters:** counts and exports follow the selected site and applicable filters. Firewall policies also support source and destination zone filters.
- **Record details:** inspect labelled settings, expandable Additional recorded fields, related records and masked troubleshooting data. Additional fields support individually protected values, including nested credentials. Device port and radio details are independent of the parent inventory's search and status filters.
- **Wi-Fi details:** include recorded roaming/FT options, WPA3 transition, MAC filtering, minimum rates, DTIM, multicast, MLO and private pre-shared-key mappings.
- **Network details:** include recorded IPv6, router advertisements, prefix delegation, DHCPv6, DHCP boot settings, WAN addressing/VLAN settings and protected PPPoE credentials.
- **Port configuration:** combines recorded settings, profile inheritance and explicit overrides. Expanded details include 802.1X, port security, STP, storm control, voice VLAN, negotiation, aggregation, PoE and SFP measurements when recorded. Unresolved profiles or network references are labelled as incomplete.
- **Coverage:** distinguishes supported views, tables outside those views, empty tables and information not included in the backup. Select a decoded table to open it in All tables. Selected operator views are not a complete interpretation of every UniFi feature.
- **All tables:** browse every decoded Network or statistics table with site filtering, search, pagination, selectable columns and sorting. Open records for additional fields. Empty tables remain selectable.
- **Console & applications:** inspect recorded application versions, console metadata, included application folders and archive member names, sizes and roles. Archive inventory does not expose raw file contents.
- **System firewall rules:** inspect recorded UniFi rule definitions separately from user policies. A definition in the backup is not proof that it was active.
- **Responsive navigation:** supports desktop and narrow screens, with keyboard focus handling for record drawers and export dialogues.

Technical labels use established casing, including DHCP, DNS, DMZ, VLAN, RADIUS and WireGuard. Empty lists display as **None**, while missing values display as **Not recorded**. Feature-specific labels such as **Disabled** are retained where appropriate. This applies to operator tables, record details and readable copies or CSV exports. Advanced troubleshooting data and All tables JSON exports retain the original array structure.

Wi-Fi security protocol labels combine the recorded authentication type, WPA3 and transition-mode settings. In the supplied backups, UniFi can retain `wpa_mode: "wpa2"` while WPA3 is enabled. Version 1.1.2 uses the WPA3 flags to distinguish WPA3 Personal, WPA3 Enterprise, 192-bit Enterprise and WPA2/WPA3 transition mode. The raw stored WPA mode remains visible in record details with an explicit raw-field label. Missing transition information and conflicting flags are labelled rather than silently assumed. The same resolved protocol is used in tables, summaries, search and exports. A WPA3 flag on an Open record alone is not treated as proof of a WPA3 Personal or Enterprise network.

## Exports and protected information

**Export view** downloads CSV or JSON for all matching rows across every page, using the current filters and displayed sort order. Exports include the backup filename, capture time and selected site. The Overview provides a network summary export. All tables exports contain every field in the matching decoded records, regardless of the columns currently selected for display, with recognised secrets redacted. Other views export their labelled inventory fields. Copy table follows the same row scope and field selection as its view export.

Recognised secret fields are redacted from ordinary view exports, copied tables, copied record summaries and troubleshooting data. Credential/token tables receive additional protection for opaque values. JSON text containing recognised credential fields is also redacted. Individual secret fields have separate reveal and copy controls. Revealed values are remasked when the view or record changes, an export dialogue opens, or the browser tab becomes hidden.

Redaction depends on recognised fields and configuration patterns. Check exported material before sharing it. This version does not provide a reveal-all control or an unredacted full-backup download. **Copy table** uses tab-separated text and prefixes formula-like values for spreadsheet pasting. CSV exports apply spreadsheet-formula protection too.

### WireGuard

Private keys are protected data points and can be revealed when a usable value is available.

| Record type | Configuration download and QR behaviour |
| --- | --- |
| Gateway acting as a VPN client | A complete, valid saved client configuration can be downloaded as `.conf` and encoded as a QR code when its settings are compatible with QR import |
| Remote VPN peer | Recovery is enabled only when a complete saved configuration or sufficient peer and server settings are available, including a client private key that matches the recorded public key |
| WireGuard server | Its recorded private key can be inspected separately. The server private key is not substituted for a client's private key |

Peer reconstruction requires the client tunnel address, explicit client-side routes, server public key and endpoint, and a known pre-shared-key setting. Server-side peer `allowed_ips` are not treated as the client's routing policy. Incomplete data leaves the QR and download controls disabled with an explanation. The reader does not generate replacement identities or invent routing defaults.

A profile exported by UniFi without a private-key value is insufficient on its own to reconstruct a working client configuration. This does not establish that every UniFi version or backup omits private keys.

The reader also inspects supported nested UniFi Users credential archives. It can recognise protected WireGuard fields, but version 1.1.2 does **not** decrypt the protected credential values encountered in the validated backup. Those values are reported as unresolved rather than absent.

WireGuard configuration downloads and QR codes deliberately include available client private keys. Treat them as credentials. QR codes are generated inside the page without an external service.

## Local processing and privacy

- Scripts, styles, icons and required libraries are embedded in `index.html`. The application uses system fonts and does not fetch webfonts.
- The page contains no analytics or telemetry. Its Content Security Policy blocks network connections and external resources.
- The reader does not save uploaded backups to browser storage. Data is held for the active session until it is replaced, cleared or the page is closed.
- Downloads and clipboard copies are explicit user actions. Clearing the session does not remove files already downloaded or clear your clipboard.
- Decoding uses a local Web Worker when available, with a main-thread fallback where worker creation is unavailable.

Treat original backups as sensitive files. Successful decoding exposes saved configuration and may expose credentials through explicit reveal controls.

## Browser compatibility and validation

This version uses modern browser APIs, including module scripts, typed arrays, local Blob URLs and Web Workers. Clipboard availability depends on the browser and execution context, with a fallback copy method provided.

Release validation comprises 91 checks performed in Chromium and the standalone decoder runtime against a real PostgreSQL backup from Network 11.0.97, two real BSON console backups from Network 10.6.106, and generated fixtures. The real legacy files use UniFi collection-selection streams. The older mongodump archive layout is covered by generated fixtures rather than a real backup of that layout.

Checks cover parsing and record preservation, malformed inputs, views, sorting/export consistency, nested secret handling, QR decoding, clipboard copying, new record details, session handling and sidebar layout. No external requests were observed during the browser checks.
Equivalent testing in Firefox, Safari and Edge has not been completed for this edition. No minimum browser versions or fixed parsing-speed guarantees are declared.

## Hosting

Serve `index.html` as a static file if you prefer a hosted copy. The application needs no backend, controller credentials or server-side parser. Local-file use remains available without hosting.

The upstream AyeTech-hosted reader is a separate build and should not be assumed to contain this edition's changes. This README does not establish or configure a deployment for this repository.

## How decoding works

1. Identify the input container and decrypt supported encrypted backup formats locally using the bundled AES implementation.
2. Unwrap supported ZIP, gzip and TAR layers, applying the implemented size and integrity checks.
3. Parse PostgreSQL JSONL table definitions and row records, or supported legacy BSON streams. Decode PostgreSQL typed wrappers and dynamic fields while giving declared relational columns precedence. Normalise legacy collection and field names into the shared operator model while retaining source records. Large BSON integers are preserved as decimal strings when JavaScript cannot represent them exactly.
4. Resolve supported record references, including legacy ID mappings, and build the operator views.
5. Read available archive metadata and inspect supported Users credential archives separately from the Network tables.

No SQL from a backup is executed. The reader does not edit backups, restore configuration, repackage archives, modify devices or query the live network. BSON code values are preserved as data and never evaluated. BSON decimal128 values are retained as hexadecimal bytes, not converted into approximate decimal numbers. Mongodump archive CRC64 values are not verified by this edition, although outer gzip integrity and BSON structural checks still apply. Recorded addresses, health values, sessions and last-seen times describe the backup snapshot.

## Limitations

- Format support is based on the implemented PostgreSQL schemas, BSON layouts and backup containers. Future UniFi changes may require a decoder update.
- A displayed record is not confirmation that the corresponding device or service is currently online or configured that way.
- Information that is absent, unsupported, encrypted or unresolved cannot be recovered simply by rendering the backup. Review Coverage and record-level explanations.
- Custom views do not reproduce the complete UniFi management interface or guarantee complete firewall, service or console coverage.
- Large archives can consume substantial browser memory even when the input file is below the size limit.

## Credits and licence

Original project: [UniFi Backup Reader by AyeTech](https://github.com/ShaunLeslie/unifi-backup-reader). This Access edition changes the interface and decoder behaviour described above.

Bundled libraries and notices are retained in `index.html`:

| Library | Version | Licence | Purpose |
| --- | --- | --- | --- |
| [fflate](https://github.com/101arrowz/fflate) | 0.8.2 | MIT | ZIP and gzip decompression |
| [aes-js](https://github.com/ricmoo/aes-js) | 3.1.2 | MIT | Local AES backup decryption |
| [qrcode-generator](https://github.com/kazuhikoarase/qrcode-generator) | 1.4.4 | MIT | Local WireGuard QR generation |
| [TweetNaCl.js](https://github.com/dchest/tweetnacl-js) | 1.0.3 | Public domain / Unlicense | X25519 private/public key matching |

Prior work credited by the original project includes [zhangyoufu/unifi-backup-decrypt](https://github.com/zhangyoufu/unifi-backup-decrypt) and [Darknetzz/UniFi-Backup-Explorer](https://github.com/Darknetzz/UniFi-Backup-Explorer).

Project licence: [MIT](LICENSE). Retain the applicable original and third-party notices when redistributing.
