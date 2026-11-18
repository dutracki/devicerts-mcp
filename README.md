# devicerts-mcp

[![Endpoint](https://img.shields.io/badge/MCP-devicerts.com%2Fmcp-1f6feb)](https://devicerts.com/mcp)
[![Transport](https://img.shields.io/badge/transport-Streamable%20HTTP-555)](https://modelcontextprotocol.io)
[![Auth](https://img.shields.io/badge/auth-none-555)](https://devicerts.com/mcp)
[![Sources](https://img.shields.io/badge/sources-FDA%20%C2%B7%20EUDAMED%20%C2%B7%20GUDID-555)](https://devicerts.com/methodology)

[![deviCERTS – medical device regulatory data MCP connector – tool definition quality and endpoint health on Glama](https://glama.ai/mcp/connectors/com.devicerts/medical-devices/badges/score.svg)](https://glama.ai/mcp/connectors/com.devicerts/medical-devices)

Ask your AI assistant about medical devices. It gets answers from FDA and EUDAMED records. Free, read-only, no login.

This is the public MCP server of [deviCERTS](https://devicerts.com), a catalogue of medical devices built from official registries: EUDAMED, FDA 510(k), De Novo, PMA, GUDID and FDA recalls.

**Endpoint:** `https://devicerts.com/mcp` (Streamable HTTP, no authentication). In a browser, the same address shows the docs page.

## Quick start

**Claude (web and desktop):** open Settings, then Connectors. Add a custom connector, paste the endpoint and leave authentication empty.

**Claude Code**

```bash
claude mcp add --transport http devicerts https://devicerts.com/mcp
```

**ChatGPT:** turn on developer mode in Settings. Create a connector with the endpoint and no authentication. The `search` and `fetch` tools are the standard pair ChatGPT connectors and deep research use. Plan availability depends on OpenAI.

**Cursor** (`.cursor/mcp.json`)

```json
{ "mcpServers": { "devicerts": { "url": "https://devicerts.com/mcp" } } }
```

**VS Code** (`.vscode/mcp.json`)

```json
{ "servers": { "devicerts": { "type": "http", "url": "https://devicerts.com/mcp" } } }
```

Menu names change between versions. Your client's MCP documentation is the final word.

Operated by [Digitaly](https://digitaly.site), publisher of deviCERTS.

## Tools

All tools are read-only (`readOnlyHint: true`, `destructiveHint: false`, `idempotentHint: true`) and return structured content with an output schema.

| Tool | Use it when | Input |
| --- | --- | --- |
| `search_devices` | A device is described by name, type, model or company, or you need a filtered list | `query`; `registry` (`eudamed`, `fda_510k`, `fda_pma`, `fda_udi`); `region` (`EU`, `US`); `device_class` (17 stored values, such as `Class IIa`); `manufacturer`; `limit` (1 to 20); `offset` |
| `find_device_by_identifier` | You have a 510(k), PMA or De Novo number, a UDI-DI or a Basic UDI-DI | `identifier` |
| `get_device` | You know the device slug and need its record | `slug`; `detail` (`summary`, `full`) |
| `get_device_recalls` | You need the FDA recalls of one device | `slug`; `status` (`ongoing`, `completed`, `terminated`) |
| `get_device_certificates` | You need the EU notified-body certificates of one device | `slug` |
| `search_recalls` | You need FDA recalls across devices: by product code, firm, class, status or date | `query`; `product_code`; `firm`; `submission_number`; `pathway` (`510k`, `pma`, `denovo`); `date_from`; `date_to`; `recall_class` (`Class I`, `Class II`, `Class III`); `status`; `limit` (1 to 20); `offset` |
| `search_certificates` | You have a certificate number, or ask what a notified body issued or a manufacturer holds | `number`; `notified_body` (number or name); `manufacturer` (name or SRN); `legislation` (`MDR`, `IVDR`); `status`; `valid_on`; `include_removed`; `limit` (1 to 20); `offset` |
| `search_manufacturers` | You look for a company by name | `query`; `country` (ISO code, such as `DE`); `limit` (1 to 20); `offset` |
| `get_manufacturer` | You know the manufacturer slug and need its profile | `slug` |
| `get_data_sources` | You ask how current the data are, or how you may reuse them | none |
| `search_gudid_listings` | You look for FDA GUDID listings by UDI-DI, brand name or labeler | `query`; `company`; `limit` (1 to 20); `offset` |
| `get_recall` | You have an FDA recall number such as Z-0079-2022 | `recall_number` |
| `search_establishments` | You look for an EU operator by name or SRN (role, country), or an FDA establishment by FEI number | `query`; `registry` (`eudamed`, `fda`); `actor_type`; `country`; `limit` (1 to 20); `offset` |
| `search`, `fetch` | ChatGPT connectors and deep research (standard search/fetch pair). Hidden from the tool list for clients recognised as not ChatGPT; any client can still call them | `query`; `id` (`device:<slug>` or `manufacturer:<slug>`) |

Search results carry `total`, `offset` and `next_offset`. Pass `next_offset` back to get the next page. Paging reaches the first 1,000 results.

`tools/list` shows all 15 tools by default. It leaves out `search` and `fetch` (13 tools) only when the request's `clientInfo.name` (from `initialize`, or the 2026-07-28 per-request `_meta`) or its `User-Agent` is recognised as another client: node/undici, python-httpx, aiohttp, python-requests, Claude/Anthropic, Cursor, VS Code, Glama, Windsurf, Zed, Continue or Cline. The server is stateless, so it decides per request; an empty or unknown `User-Agent`, `curl`, or anything containing `openai` or `chatgpt` always gets all 15. Calls to `search` and `fetch` work for every client.

Every result carries `canonical_url`, `cite_as`, `retrieved_at`, `status_as_of` and a `terms` block.

## Example prompts

- What is FDA 510(k) number K110287, and who is the manufacturer?
- List Class III insulin pumps in EUDAMED and give their UDI-DIs.
- Does the HeartStart FRx defibrillator have ongoing FDA recalls? Give the recall numbers.
- Which notified body certified this device, and when does the certificate expire?
- List ongoing Class I FDA recalls by Medtronic started since 2024.
- Which MDR certificates has notified body 0344 issued that are valid today?

## Limits

- Read-only. The server cannot change any record.
- Paged search tools (`search_devices`, `search_recalls`, `search_certificates`, `search_manufacturers`, `search_gudid_listings`, `search_establishments`): `limit` 1 to 20 (default 10), `offset` 0 to 999; offset plus limit stays within the first 1,000 results. `search` returns up to 10, with no `limit`.
- Every POST to the endpoint, whatever its JSON-RPC method, is limited to 16 KB: a larger body gets HTTP 413 with JSON-RPC error `-32600`. A JSON-RPC batch (2025-03-26 clients) holds 1 to 5 messages of any method; otherwise HTTP 400, `-32600`.
- Results are cached for up to an hour. Each request has a 10-second deadline.

### Busy and retry answers

None of these is a JSON-RPC error carrying retry data; read the HTTP status and headers.

| When | Answer |
| --- | --- |
| More than 300 requests a minute from one network address (burst 300). A request counts once, a batch too | HTTP `429`, `Content-Type: application/problem+json`, body `{"type":"about:blank","title":"Too Many Requests","status":429,"detail":"Rate limit for this client reached."}`, `Retry-After` in seconds |
| Server busy: 2 MCP requests run at a time, up to 16 wait at most 3 seconds | HTTP `503`, `text/plain` `Service busy, retry later.`, `Retry-After: 30`, `X-Admission: mcp/overload`, `mcp/queue_full` or `mcp/queue_timeout` |
| The backend budget for MCP, shared by all assistants together (10 calls a second, burst 50; 2 at a time, 8 waiting), is used up, or the backend fails | HTTP `200`, tool result with `isError: true` and the text `DevicErts is rate limiting this client. Retry after N seconds.` (N from the backend's `Retry-After`) or `DevicErts is temporarily unavailable. The call can be retried.` |
| The backend is slow (10-second deadline per request) | After about 10 s: HTTP `200`, tool result with `isError: true` and `DevicErts did not answer in time. The call can be retried.` Only if the answer still has not left after about 11 s: HTTP `504`, JSON-RPC error `-32603` `The request ran past its deadline. It can be retried.` No `Retry-After` on either |
| Edge flood guard: POST `/mcp` counts with page views toward 60 requests per 10 seconds per address | Cloudflare answers `429` and blocks the address for 10 seconds |

## Data sources and licences

Data returned by the server follow the terms of each source registry, and the deviCERTS compilation needs the CC BY 4.0 attribution below. The files of this repository are under the MIT licence (see [Licence](#licence)).

| Source | Region | Terms |
| --- | --- | --- |
| EUDAMED (devices, certificates) | European Union | European Commission reuse terms |
| FDA 510(k), De Novo, PMA, recalls (via openFDA) | United States | U.S. public domain |
| FDA GUDID (AccessGUDID) | United States | U.S. public domain |

- EUDAMED certificates are synced weekly, the other registries monthly. `get_data_sources` gives the last sync date of each.
- The deviCERTS compilation, layout and text are licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Attribution, as in every result: *DevicErts, https://devicerts.com*.
- The server returns registry facts only, no AI-written text.
- A device can be in several registries, so counts overlap and do not add up.

## Licence

- The files in this repository (`server.json`, examples, docs code) are licensed under the [MIT licence](LICENSE).
- Data returned by the server are not covered by the MIT licence. Registry data follow each source's terms (see above). The deviCERTS compilation, layout and text are under [CC BY 4.0](LICENSE-CC-BY-4.0), with the attribution *DevicErts, https://devicerts.com*.

## Privacy

The server needs no account and its tools ask for no personal data, but search text is free text and can contain personal data, such as a person's name. Avoid putting personal data in a search unless you need to.

For each request the server logs one line: the tool name, the client name and version, the status, timing, result count, cache use, the page asked for, the search text or identifier cut to 80 characters, and a hash of the IP address (SHA-256 of the address, a secret salt and the UTC date, so it changes every day; the address itself is never logged).

- **Where and how long:** only in the server's rotated container logs. Retention is set by size, not by date: at most 5 files of 20 MB (about 100 MB, usually a few days of lines), cleared whenever the server is redeployed. Usage summaries may be kept in internal operations reports for up to 12 months: aggregated counts per tool and client and of the most frequent search terms, without IP hashes.
- **Legal basis:** legitimate interest in running the server, seeing how it is used and protecting it against abuse (GDPR Art. 6(1)(f)). You can object at any time: contact@devicerts.com.

Details: https://devicerts.com/privacy#p-mcp

## Disclaimer

deviCERTS is not an official register and gives no medical, legal or regulatory advice. The registry record is the record. Our copy can be incomplete or out of date: check the original before you rely on it.

## Changelog

### 1.2.0 (2026-11-18)

- New tool `search_gudid_listings`: FDA GUDID listings by UDI-DI, brand name or labeler. Returns listing fields (`primary_di`, brand or device name, version or model, labeler, device class, commercial status); an input that looks like a GTIN or HIBCC code matches the DI exactly first.
- New tool `get_recall`: one FDA recall by its number, such as `Z-0079-2022`.
- New tool `search_establishments`: EU operators by name or SRN (role, country) and FDA establishments by FEI number. `identifier_type` tells an SRN, a notified-body number and an authority ID apart.
- Publisher: [Digitaly](https://digitaly.site), shown in the server metadata and instructions.
- 15 tools in total; the 12 earlier tools are unchanged.

### 1.1.0 (2026-11)

- New tools: `get_device_recalls` (FDA recalls of one device, with a status filter) and `get_device_certificates` (EU notified-body certificates of one device).
- New tools: `search_recalls` (FDA recalls across all devices by words, product code, firm, submission, pathway, date range, class and status) and `search_certificates` (EUDAMED certificates by number, notified body, holder, legislation, status and validity date).
- `search_devices`: new `region` filter (`EU`, `US`), `offset` paging and `device_class` as an enum of the 17 stored values.
- `search_manufacturers`: new `country` filter and `offset` paging.
- Search results carry `offset` and `next_offset`.
- Every tool description now says when to use it, when not to, and which tool to use instead.
- Every tool has a title, the four annotation hints and an output schema.

### 1.0.0 (2026-11)

- First public release: `search_devices`, `find_device_by_identifier`, `get_device`, `search_manufacturers`, `get_manufacturer`, `get_data_sources`, `search`, `fetch`.

## Links

- Docs page (same address as the endpoint, in a browser): https://devicerts.com/mcp
- Methodology (sources, update schedule): https://devicerts.com/methodology
- Privacy: https://devicerts.com/privacy
- For AI readers: https://devicerts.com/llms.txt
- Support: contact@devicerts.com
