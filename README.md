# devicerts-mcp

[![Endpoint](https://img.shields.io/badge/MCP-devicerts.com%2Fmcp-1f6feb)](https://devicerts.com/mcp)
[![Transport](https://img.shields.io/badge/transport-Streamable%20HTTP-555)](https://modelcontextprotocol.io)
[![Sources](https://img.shields.io/badge/sources-FDA%20%C2%B7%20EUDAMED%20%C2%B7%20GUDID-555)](https://devicerts.com/methodology)

Ask your AI assistant about medical devices and get answers from FDA and EUDAMED records. Free, read-only, no login.

This is the public MCP server of [deviCERTS](https://devicerts.com), a catalogue of medical devices built from official registries: EUDAMED, FDA 510(k), PMA, De Novo and GUDID.

**Endpoint:** `https://devicerts.com/mcp` (Streamable HTTP). The same address shows the docs page in a browser.

## Connect

**Claude Code**

```bash
claude mcp add --transport http devicerts https://devicerts.com/mcp
```

**Cursor** (`.cursor/mcp.json`)

```json
{ "mcpServers": { "devicerts": { "url": "https://devicerts.com/mcp" } } }
```

**VS Code** (`.vscode/mcp.json`)

```json
{ "servers": { "devicerts": { "type": "http", "url": "https://devicerts.com/mcp" } } }
```

**Claude (web and desktop):** Settings, Connectors, add a custom connector, paste the endpoint, leave authentication empty.

**ChatGPT:** turn on developer mode in Settings, create a connector with the endpoint and no authentication. Plan availability depends on OpenAI.

Menu names change between versions. Your client's MCP documentation is the final word.

## Tools

| Tool | Input | Returns |
| --- | --- | --- |
| `search_devices` | `query`, `registry` (`eudamed`, `fda_510k`, `fda_pma`, `fda_udi`), `manufacturer`, `device_class`, `limit` (1 to 20, default 10) | Matching devices with manufacturer, class, registries, status and page link |
| `find_device_by_identifier` | `identifier` | Device for a 510(k), PMA or De Novo number, a UDI-DI or a Basic UDI-DI (a group) |
| `get_device` | `slug`, `detail` (`summary` or `full`) | One device record with registrations, identifiers, risk class and sources |
| `search_manufacturers` | `query`, `limit` (1 to 20, default 10) | Matching manufacturers |
| `get_manufacturer` | `slug` | One manufacturer with country, registrations, up to 10 devices and 10 recalls |
| `get_data_sources` | none | Registries, their terms and last sync date |
| `search`, `fetch` | `query`; `id` | Aliases in the shape ChatGPT expects |

Every result carries `canonical_url`, `cite_as`, `retrieved_at` and a `terms` block.

## Example prompts

- What is FDA 510(k) number K110287, and who is the manufacturer?
- Find Class IIa syringes in EUDAMED and list their UDI-DIs.
- Which devices from this manufacturer appear in both EUDAMED and FDA records?
- Compare the registrations of two infusion pumps and say which registry each fact comes from.

## Data, licence and limits

- Read-only. At most 20 results per call, 16 KB per request, 5 calls per batch. Results are cached for up to an hour. Requests share a global queue, with a safety cap of 120 calls a minute per network address.
- The deviCERTS compilation, layout and text are licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Attribution, as in every result: *DevicErts, https://devicerts.com*.
- Registry data stay under each source's own terms: EUDAMED under the European Commission reuse terms, U.S. FDA and GUDID data in the U.S. public domain. `get_data_sources` lists them.
- The server returns registry facts only, no AI-written text.
- A device can be in several registries, so counts overlap and do not add up.

## Disclaimer

deviCERTS is not an official register and gives no medical, legal or regulatory advice. The registry record is the record. Our copy can be incomplete or out of date: check the original before you rely on it.

## Links

- Docs page (same address as the endpoint, in a browser): https://devicerts.com/mcp
- Methodology (sources, update schedule): https://devicerts.com/methodology
- For AI readers: https://devicerts.com/llms.txt
- Contact: contact@devicerts.com
