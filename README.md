# Klardaten DATEV MCP Server

Connect AI assistants and automation tools to DATEV data through Klardaten.

The Klardaten DATEV MCP Server provides access to DATEV data via the Klardaten
DATEVconnect Gateway, with read access by default and optional creation of
uncommitted accounting sequences (Buchungsstapel). It is built for German tax
firms, accounting teams, and finance workflows that want to use DATEV data inside
AI assistants, automation platforms, and MCP-compatible clients.

## What You Can Do

Use the server to ask practical questions about DATEV data and combine the
answers with the other tools in your assistant or workflow platform.

Examples:

- Find clients, contact data, responsibilities, and master data.
- Review bookings, balances, open receivables, and open payables.
- Create uncommitted accounting sequences for review in DATEV when the additional
  permission is granted.
- Search documents and receipts in DATEV document management.
- Look up invoices, orders, fees, cost centers, and order values.
- Inspect payroll clients, employees, salaries, working time, tax, and social
  insurance data.
- Prepare reporting inputs, email drafts, task lists, or workflow steps from
  DATEV data.

## Supported DATEV Areas

- Accounting
- Master data
- Document management
- Order management
- Payroll (DATEV Lohn und Gehalt and LODAS)

Availability depends on the DATEV modules enabled for your Klardaten account
and connected DATEV environment. LODAS read access requires the
Lohnaustauschdatenservice option to be enabled for the selected client.

## Document Search Add-on

Choose the DMS tool by the information needed:

| Request | Tool |
| --- | --- |
| Current DATEV inventory or filtering by document number, client, date, state or folder | Existing `datev_dms_list`; no text query required |
| Find words in filenames, descriptions or keywords, or list indexed/pending files | Optional `datev_dms_search` with `mode: "metadata"`; literal contains matching, with no query required for listing |
| Find text or facts inside documents, such as a contract's termination terms | Optional `datev_dms_search` with `mode: "content"`; a nonempty query, with date/client filters when needed |

Metadata mode includes stored files awaiting or failing extraction.
Stored metadata can lag DATEV and covers only locally stored files; current or
complete live inventory requests use `datev_dms_list`. Metadata matches do not
prove facts inside a document or replace evidence after a failed content search.

A query without `mode` defaults to `content`, preserving body retrieval and relevance scores. Explicit `mode: "hybrid"` combines body retrieval with
metadata matching. Omitting both lists metadata by filters alone. Content and
hybrid modes require a query and use `topK` (default 5, maximum 20). Metadata
mode uses `pageSize` (default 10, maximum 100) and `cursor`. These pagination
arguments cannot be mixed. The `filename` filter remains exact and case-insensitive.

Metadata matching finds fragments in the middle of words, including German
compounds, across filenames, descriptions and keywords. NFC-normalized Unicode
alphanumeric terms of at least three characters are ANDed across fields in any
order, without case or diacritic sensitivity. Shorter terms are ignored. A
metadata query without eligible terms is rejected; hybrid keeps its content
lane and reports `metadata_query_applied: false`. Query terms are literal;
SQL wildcards, regex and FTS operators are not supported.

Hybrid combines both ranked lanes with equal-weight reciprocal-rank fusion
(k=60), deduplicating stored files while retaining separate attachments.
`minScore` filters only the content lane. `match_sources` identifies content,
filename, description and/or keywords. Metadata-only matches have no snippet,
chunks or page reference and cannot establish body facts. Hybrid fails if a
component or permission check fails. Results report effective mode and whether
metadata matching was applied. A non-null `next_cursor` continues metadata
pagination with the same profile, query and filters, even after an empty page;
null ends the scan.

Metadata listings return newest DATEV filing times first, unknown dates last, then ascending stored file ID. Content and hybrid searches retain relevance ordering.

Creation and modification ranges on the search tools use inclusive DATEV local
calendar days, separate from receipt dates and UTC indexing time. Results
respect the selected DATEV profile's access rights. Incomplete metadata
enrichment is reported as `search_meta.metadata_complete: false`.

Example MCP arguments:

```json
{"query": "Pharmacy", "topK": 5}
```

This defaults to content search. Use `"mode": "hybrid"` to combine content and metadata. A contains-only search for fragments in names and labels:

```json
{"mode": "metadata", "query": "chnung ller", "pageSize": 10}
```

Listing ten successfully indexed files needs no query:

```json
{"indexed": true, "pageSize": 10}
```

### Document periods and business filters

`datev_dms_search` returns DMS assessment year/month, additional years,
receipt/document date, receipt number, amount, document class, correspondence
partner firm number and order assignment. Results also include readable class,
order, domain, folder, register, state, employee and DATEV user names when present.
Missing source values remain unknown; these properties are not inferred from text.

All three modes accept these structured filters:

| Filter | Meaning |
| --- | --- |
| `year`, `yearFrom`, `yearTo`, `month` | Primary assessment year, inclusive year bounds and exact month |
| `relatedYear` | Primary year or one of the additional `more_years` values |
| `receiptFrom`, `receiptTo` | Inclusive receipt/document days in DATEV local time |
| `receiptNumber` | Exact case-insensitive match, preserving leading zeros and punctuation |
| `amount`, `amountMin`, `amountMax` | Exact decimal strings with inclusive bounds; no currency or grouping separators |
| `classId`, `correspondencePartnerFirmNumber` | Exact DMS document class or correspondence partner firm number |
| `orderId`, `orderNumber`, `orderCreationYear`, `orderAssessmentYear` | Order assignment; number plus creation year identifies an order precisely |
| `className`, `orderName`, `domainName`, `folderName`, `registerName`, `stateName`, `employeeName`, `datevUserName` | Exact whole-name matching without case or diacritic sensitivity |

Assessment periods, receipt dates and filing/modification dates are separate
properties. An order number can repeat across creation years. Name labels are
cached and can repeat across IDs; supplied filters combine with AND. General
contains queries continue to cover filenames, descriptions and keywords.

Amounts are returned as decimal strings, such as `"150.95"`, retaining exact value.
The source amount does not establish a currency. Existing documents receive the
new properties through metadata backfill without content reindexing. During that
process `search_meta.metadata_complete: false` indicates incomplete coverage.

## Read and Write Access

Connections remain read-only unless the user explicitly grants the additional
permission to create accounting sequences. Hosted clients, including Microsoft
Cowork, can use `datev_accounting_create_sequence` with the OAuth scope
`datev:accounting-sequences:create`. Existing read-only connections must reconnect
and opt in to this permission; refreshing a token does not add it. Local stdio
connections remain read-only.

This is the only supported write operation. It creates uncommitted
financial-accounting sequences for review and finalization in DATEV. The server
cannot finalize sequences or edit or delete existing records.

Sequence creation is not automatically retried. After a timeout or uncertain
result, check DATEV for an already-created sequence before trying again to avoid
duplicates.

## Supported Clients

The server uses MCP Streamable HTTP and can be used with MCP-compatible clients
that support OAuth-based authentication.

Setup guides are available for:

- Open WebUI
- n8n
- Langdock
- Claude Code
- Codex

Other MCP-compatible clients can use the same connection details when they
support MCP Streamable HTTP with OAuth.

## Connection Details

MCP endpoint:

```text
https://api.klardaten.com/mcp
```

Authentication:

```text
OAuth 2.1 with dynamic client registration
```

Full setup instructions:

```text
https://klardaten.com/docs/mcp-setup
```

## Example Questions

```text
Which receivables are still open for this client?
```

```text
Show me the revenue and account balances for the current fiscal year.
```

```text
Find documents for this client that still need review.
```

```text
Which invoices and orders exist for this customer?
```

```text
Summarize relevant payroll data for this employee.
```

```text
Create a short reporting summary from the available DATEV data.
```

With the additional sequence-creation permission:

```text
Create an uncommitted accounting sequence from these records for review in DATEV.
```

## Access

To use the Klardaten DATEV MCP Server, you need:

- An active Klardaten account with access to the DATEV MCP Server.
- Access to a connected DATEV environment through Klardaten.
- An MCP-compatible client.

Start here:

```text
https://klardaten.com/docs/mcp-setup
```

## About Klardaten

Klardaten builds automation infrastructure for German tax firms. The DATEV MCP
Server helps assistants and workflow tools work with DATEV data through a
standardized MCP interface.

Learn more:

```text
https://klardaten.com
```
