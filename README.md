# devicerts-mcp

[![Endpoint](https://img.shields.io/badge/MCP-devicerts.com%2Fmcp-1f6feb)](https://devicerts.com/mcp)
[![Transport](https://img.shields.io/badge/transport-Streamable%20HTTP-555)](https://modelcontextprotocol.io)
[![Auth](https://img.shields.io/badge/auth-none-555)](https://devicerts.com/mcp)
[![Sources](https://img.shields.io/badge/sources-FDA%20%C2%B7%20EUDAMED%20%C2%B7%20GUDID-555)](https://devicerts.com/methodology)

Ask your AI assistant about medical devices. It gets answers from FDA and EUDAMED records. Free, read-only, no login.

This is the public MCP server of [deviCERTS](https://devicerts.com), a catalogue of medical devices built from official registries: EUDAMED, FDA 510(k), De Novo, PMA, GUDID and FDA recalls.

**Endpoint:** `https://devicerts.com/mcp` (Streamable HTTP, no authentication). In a browser, the same address shows the docs page.

## Quick start

**Claude (web and desktop):** open Settings, then Connectors. Add a custom connector, paste the endpoint and leave authentication empty.

**Claude Code**

```bash
claude mcp add --transport http devicerts https://devicerts.com/mcp
```

**ChatGPT:** turn on developer mode in Settings. Create a connector with the endpoint and no authentication. The `search` and `fetch` tools follow the ChatGPT connector and deep research format. Plan availability depends on OpenAI.

**Cursor** (`.cursor/mcp.json`)

```json
{ "mcpServers": { "devicerts": { "url": "https://devicerts.com/mcp" } } }
```

**VS Code** (`.vscode/mcp.json`)

```json
{ "servers": { "devicerts": { "type": "http", "url": "https://devicerts.com/mcp" } } }
```

Menu names change between versions. Your client's MCP documentation is the final word.

## Tools

All tools are read-only (`readOnlyHint: true`, `destructiveHint: false`, `idempotentHint: true`) and return structured content with an output schema.

| Tool | Use it when | Input |
| --- | --- | --- |
| `search_devices` | A device is described by name, type, model or company, or you need a filtered list | `query`; `registry` (`eudamed`, `fda_510k`, `fda_pma`, `fda_udi`); `region` (`EU`, `US`); `device_class` (17 stored values, such as `Class IIa`); `manufacturer`; `limit` (1 to 20); `offset` |
| `find_device_by_identifier` | You have a 510(k), PMA or De Novo number, a UDI-DI or a Basic UDI-DI | `identifier` |
| `get_device` | You know the device slug and need its record | `slug`; `detail` (`summary`, `full`) |
| `get_device_recalls` | You need the FDA recalls of one device | `slug`; `status` (`ongoing`, `completed`, `terminated`) |
| `get_device_certificates` | You need the EU notified-body certificates of one device | `slug` |
| `search_manufacturers` | You look for a company by name | `query`; `country` (ISO code, such as `DE`); `limit` (1 to 20); `offset` |
| `get_manufacturer` | You know the manufacturer slug and need its profile | `slug` |
| `get_data_sources` | You ask how current the data are, or how you may reuse them | none |
| `search`, `fetch` | Your client supports only the ChatGPT search and fetch pair | `query`; `id` |

Search results carry `total`, `offset` and `next_offset`. Pass `next_offset` back to get the next page. Paging reaches the first 1,000 results.

Every result carries `canonical_url`, `cite_as`, `retrieved_at`, `status_as_of` and a `terms` block.

## Example prompts

- What is FDA 510(k) number K110287, and who is the manufacturer?
- List Class III insulin pumps in EUDAMED and give their UDI-DIs.
- Does the HeartStart FRx defibrillator have ongoing FDA recalls? Give the recall numbers.
- Which notified body certified this device, and when does the certificate expire?
- Find manufacturers named Philips based in the Netherlands.

## Limits

- Read-only. The server cannot change any record.
- At most 20 results per page. Requests up to 16 KB, at most 5 calls per batch.
- Results are cached for up to an hour.
- Requests share a global queue, with a safety cap of 300 calls a minute per network address. A busy answer says when to retry.

## Data sources and licences

| Source | Region | Terms |
| --- | --- | --- |
| EUDAMED (devices, certificates) | European Union | European Commission reuse terms |
| FDA 510(k), De Novo, PMA, recalls (via openFDA) | United States | U.S. public domain |
| FDA GUDID (AccessGUDID) | United States | U.S. public domain |

- Registries are synced monthly. `get_data_sources` gives the last sync date of each.
- The deviCERTS compilation, layout and text are licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Attribution, as in every result: *DevicErts, https://devicerts.com*.
- The server returns registry facts only, no AI-written text.
- A device can be in several registries, so counts overlap and do not add up.

## Privacy

The tools request no personal data and need no account. For each request the server logs one line: the tool name, the client name and version, the status, timing, result count, cache use, the page asked for, search terms cut to 80 characters, and a daily salted hash of the IP address (never the address itself). The lines stay in rotated server logs. Details: https://devicerts.com/privacy

## Disclaimer

deviCERTS is not an official register and gives no medical, legal or regulatory advice. The registry record is the record. Our copy can be incomplete or out of date: check the original before you rely on it.

## Changelog

### 1.1.0 (2026-11)

- New tools: `get_device_recalls` (FDA recalls of one device, with a status filter) and `get_device_certificates` (EU notified-body certificates of one device).
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
