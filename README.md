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
