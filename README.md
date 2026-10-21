# Klardaten DATEV MCP Server

Connect AI assistants and automation tools to DATEV data through Klardaten.

The Klardaten DATEV MCP Server provides access to DATEV data via the Klardaten
DATEVconnect Gateway, with read access by default and optional writes for
uncommitted accounting sequences (Buchungsstapel), DMS documents, and client and
addressee master data. It is built for German tax
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
- Create and update clients and addressees when the relevant master-data
  permission is granted.
- Search documents and receipts in DATEV document management.
- Upload original files and create DMS documents through supported native file
  inputs, including Claude's interactive upload picker.
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

## Read and Write Access

Connections remain read-only unless the user explicitly grants additional write
permissions. Hosted clients, including Claude, ChatGPT, Langdock, and Microsoft Cowork,
can opt in to these permissions independently:

| Capability | OAuth scope |
| --- | --- |
| Create uncommitted accounting sequences | `datev:accounting-sequences:create` |
| Upload original files and create DMS documents through supported native file inputs | `datev:dms-documents:create` |
| Create clients and addressees | `datev:master-data:create` |
| Update clients and addressees | `datev:master-data:update` |

Existing connections must reconnect and explicitly select newly added
permissions. Refreshing a token cannot add a permission. Local stdio
connections remain read-only.

Sequence creation produces uncommitted financial-accounting sequences for
review and finalization in DATEV; the server cannot finalize them.

DMS document creation uses the original bytes supplied through a supported file
input. In Claude, the interactive MCP App lets the user select the original
file, verifies its upload, and returns a file reference for document creation.
The selected destination and metadata are resolved separately. Claude does not
need to reconstruct or transform the file.

Master-data creation requires the linked addressees to exist before a client
is created. Updates fetch the complete record first, preserve omitted fields,
and merge nested addresses, communications, bank accounts, tax offices, and
contacts by ID. Unmentioned items remain. Historical edits include the complete
history and matching current value together. `clearFields` explicitly clears
optional nonhistorical scalar fields. Immutable fields cannot change.
`clearFields` cannot erase history or collections, and updates do not delete
collection items.

Master-data writes apply directly without a custom per-change approval
workflow. Client-native confirmation controls may apply. Read–merge–PUT cannot
atomically protect against concurrent edits in DATEV.

Writes are not automatically retried. After a timeout or uncertain result,
verify DATEV through read-only lookups before another write attempt to avoid
duplicates or unintended overwrites.

## Supported Clients

The server uses MCP Streamable HTTP and can be used with MCP-compatible clients
that support OAuth-based authentication.

Setup guides are available for:

- Open WebUI
- n8n
- Langdock
- [Claude](https://klardaten.com/en/docs/mcp-setup/claude)
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
