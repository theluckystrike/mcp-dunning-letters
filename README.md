# mcp-dunning-letters

<!-- mirror-seo:start -->

**MCP server for dunning letters and overdue invoice payment reminders: reminder, final notice and aging.** Chase overdue invoices on a ladder anchored to the due date: reminder 1, reminder 2, the final notice, with the aging and the day's chase list.

Works with Claude Desktop, Claude Code, Cursor and any Model Context Protocol client. Runs on your own machine, or hosted with no install.

Product page: <https://mcp.zovo.one/s/dunning-letters> — what it does, the tools it exposes, and a live token endpoint.

## Install

**Hosted, nothing to install.** Get a token from <https://mcp.zovo.one/mcp/connect> (the connect page) or <https://mcp.zovo.one/mcp/token> (the same token as JSON); a free anonymous one is issued on the spot and a Pro key works the same way. Then point an MCP client at `https://mcp.zovo.one/mcp/dunning-letters` over streamable-http and send the token as `Authorization: Bearer <token>`.

If your client cannot set headers, put the token in the path instead: `https://mcp.zovo.one/mcp/dunning-letters/t/<token>`. Both forms work. The bare URL with no token answers 401 on `tools/call`, so the token is not optional.

**Claude Desktop, one click.** Download `dunning-letters.mcpb` from the [latest release](https://github.com/theluckystrike/mcp-servers/releases/latest) and double-click it.

**From source.** The mirror is self-contained: every `@theluckystrike/*` dependency is vendored, so a fresh clone builds with no extra setup.

```sh
git clone https://github.com/theluckystrike/mcp-dunning-letters.git
cd mcp-dunning-letters
npm install && npm run build
```

Then point your client at the built entry point:

```json
{
  "mcpServers": {
    "dunning-letters": {
      "command": "node",
      "args": ["/absolute/path/to/mcp-dunning-letters/dist/index.js"]
    }
  }
}
```

> `@theluckystrike/mcp-dunning-letters` is **not published on npm yet**, so an `npx -y @theluckystrike/mcp-dunning-letters` command will fail. The three paths above are the working ones and each is exercised by CI.

![dunning-letters demo](https://raw.githubusercontent.com/theluckystrike/mcp-servers/main/assets/demo-dunning-letters.gif)

Read-only mirror of [mcp-servers/servers/dunning-letters](https://github.com/theluckystrike/mcp-servers/tree/main/servers/dunning-letters). See [MIRROR.md](MIRROR.md).

<!-- mirror-seo:end -->

**In the [official MCP Registry](https://registry.modelcontextprotocol.io/v0.1/servers/io.github.theluckystrike%2Fdunning-letters/versions/latest)** (`io.github.theluckystrike/dunning-letters`).
Chase overdue invoices without losing the thread. Register an unpaid invoice -- client, invoice reference, amount in integer cents, currency, due date -- and the server runs the escalation ladder: reminder 1 (polite, due + 7 days), reminder 2 (firm, with the late fees note, due + 14), and the final notice (before-action wording, due + 21; the gaps are configurable per invoice). For each stage it generates the letter, as Markdown or as a self-contained printable HTML page. It records what you sent and when, lists everything overdue with days late and stage, ages the whole register into current/30/60/90+ buckets, and answers "what do I need to send today". **Nothing is emailed or sent anywhere: this server produces the letter text, and sending it is your act.**

Built by theluckystrike.

npm publish for `@theluckystrike/mcp-dunning-letters` is pending, so `npx -y @theluckystrike/mcp-dunning-letters` returns 404 today. Until then, a clone+build is the working path.

## Install

### Claude Desktop

macOS `~/Library/Application Support/Claude/claude_desktop_config.json`, Windows `%APPDATA%\Claude\claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "dunning-letters": {
      "command": "npx",
      "args": ["-y", "@theluckystrike/mcp-dunning-letters"]
    }
  }
}
```

### Claude Code

```sh
claude mcp add dunning-letters -- npx -y @theluckystrike/mcp-dunning-letters
```

### Cursor

`~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (project), same entry as Claude Desktop.

## Tools

| tool | what it does |
| --- | --- |
| `invoice_register` | Start chasing an unpaid invoice: client, reference, amount, currency, due date. Returns `DUN-YYYY-NNNN` and the three escalation dates |
| `payment_record` | Record money received, full or partial. A part payment lowers what the next letter asks for; covering the balance closes the ladder |
| `letter_render` | Generate the chase letter for the current stage -- reminder 1, reminder 2 or the final notice -- as Markdown or self-contained printable HTML. Nothing is emailed: you send the text it produces |
| `letter_sent` | Record that a letter actually went out, with its date, so the ladder advances. Letters go out in order |
| `overdue_list` | Every unpaid invoice past its due date: days late, outstanding, letters sent, what is due next, worst first |
| `aging_summary` | The whole register in current/1-30/31-60/61-90/91+ buckets, counts and totals in integer cents, per currency |
| `chase_today` | The day's chase list: which invoices cross an escalation threshold today, which letter to send each, what falls due next |
| `invoice_status` | One chased invoice in full: payments, letters, schedule, next action |
| `invoice_delete` | Delete an invoice entered wrongly. The id is not reissued |
| `license_status` / `license_activate` | Free or Pro, and the key |

## Free vs Pro

| | Free | Pro |
| --- | --- | --- |
| Unpaid invoices chased at once | 3 | Unlimited |
| All three letters, both formats | Yes | Yes |
| Aging summary and the day's chase list | Yes | Yes |
| Late fee accrual in the letters | Yes | Yes |
| Payment recording and history | Yes | Yes |

The cap is on how many chases run at once, never on the letters or the aging: three late payers taken from first reminder to final notice is a real chase list, and an invoice that gets paid frees its slot. What is metered is breadth -- a fourth concurrent chase is a collections workload, not a freelancer's month.

**Get Pro:** https://mcp.zovo.one/buy/dunning-letters -- $19 one-time for this server, or $39 for the bundle.

## A measured insight

**The ladder is anchored to the due date, not to the last letter, and the two drift apart exactly when chasing is going badly.**

The worked chase in `test/_client.mjs`: USD 1,250.00 due 2026-06-01, gaps 7/14/21, so the letters fall due on 06-08, 06-15 and 06-22 whether or not anything was sent. If reminder 1 actually goes out late -- say 06-20, twelve days after its date -- a previous-letter-anchored ladder would push the final notice to 07-11. This one does not: 06-22 stands, because the client's obligation was fixed by the due date, not by when you got around to writing. Recording a sending moves only which stage is next; it never moves the schedule. The letters are also strictly sequential: an invoice 60 days late with nothing sent is still owed reminder 1, because a final notice that no polite letter preceded reads as a threat, not a chase.

The late fee is measured the same way, once: simple interest, pro-rata on a 30-day month, on the amount outstanding on the day the letter is written, rounded once to the minor unit. On the worked chase that is 125,000 minor units at 2% over 30 days late -- 2,500 exactly -- and after a 50,000 part payment it is 1,500 on the 75,000 that remains, so the final notice asks for 76,500 and no invented figure more.

## Privacy

All data stays local, in `${XDG_DATA_HOME:-~/.local/share}/mcp-servers/dunning-letters/`. Two files: `invoices.json`, `counter.json`. Nothing is sent anywhere -- no account, no API key, no network call in this server at all -- and the letters are rendered to text for you to send yourself; this server holds no mail credentials and wants none. License keys are verified offline.

Built by theluckystrike. https://github.com/theluckystrike
