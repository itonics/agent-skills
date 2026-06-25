---
name: itonics-cli
description: ITONICS Innovation CLI. List, get, create, update, or delete elements, element types, files, attachments, watches, and likes via the OData v2 API.
metadata:
  version: "0.1.0"
  author: itonics
  repository: https://github.com/itonics/agent-skills
  tags: itonics,innovation,cli,odata,api
  alwaysApply: "false"
---

# itonics-cli (ITONICS Innovation CLI)

Thin wrapper around the ITONICS Innovation OData v2 API. Use for any task that lists, creates, updates, or deletes elements / element types / files / attachments / watches / likes in an ITONICS tenant.

## No terminal? Use the MCP connector

Prefer not to install the CLI? On **Claude.ai, Claude Desktop, Cursor, or ChatGPT**, the hosted itonics-mcp connector exposes the same surface as self-describing MCP tools — no local config needed. Add a custom connector / remote MCP server and paste the Streamable HTTP endpoint `https://itonics-mcp.com/mcp` (include the `/mcp` path), or for Claude Code run `claude mcp add --transport http itonics https://itonics-mcp.com/mcp`. Auth runs through a browser OAuth flow. The [property-write encoding](#property-writes) below applies the same way.

## Prerequisites

Install the CLI using the [setup instructions](https://github.com/itonics/itonics-cli#install), then log in once:

```bash
itonics login   # prompts for domain, space, and API key; saves to ~/.config/itonics/config.toml (chmod 0600)
itonics whoami  # confirm what's resolved
```

Env vars still work and override the saved config:

```bash
export ITONICS_DOMAIN=https://your-tenant.itonics.io
export ITONICS_API_KEY=your-api-key
export ITONICS_SPACE=your-space-uri
```

If any command fails with `missing credentials: …`, prompt the user to run `itonics login` rather than guessing values.

## Quick Reference

| Task | Command |
|------|---------|
| Log in / rotate key | `itonics login` |
| Show effective creds | `itonics whoami` |
| Log out | `itonics logout` |
| List elements | `itonics elements list -n 20 --format table` |
| Filter by type | `itonics elements list -f "elementTypeUri eq 'XXX'"` |
| Full-text search | `itonics elements list -s "keyword"` |
| Get one element | `itonics elements get URI --raw` |
| Create element | `itonics elements create --type URI --label LBL --created-by EMAIL` |
| Update element | `itonics elements update URI --updated-by EMAIL --label "New"` |
| Delete elements | `itonics elements delete URI [URI...] --yes` |
| List element types | `itonics types list --format table` |
| Upload file | `itonics files upload ./logo.png --created-by EMAIL` |
| Upload + attach | `itonics files upload ./logo.png --created-by EMAIL --attach-to ELEMENT_URI` |
| File details | `itonics files get FILE_URI` |
| List attachments | `itonics attachments list ELEMENT_URI` |
| Attach by URI | `itonics attachments attach ELEMENT_URI FILE_URI --attached-by EMAIL` |
| Detach | `itonics attachments detach ELEMENT_URI FILE_URI --detached-by EMAIL` |
| List watches | `itonics watches list ELEMENT_URI` |
| Add watch | `itonics watches add ELEMENT_URI USER_URI` |
| Remove watch | `itonics watches remove ELEMENT_URI USER_URI` |
| List likes | `itonics likes list ELEMENT_URI --orderby "likedOn desc"` |
| Add like | `itonics likes add ELEMENT_URI USER_URI` |

## Required Input Resolution

For commands that need specific values (`URI`, `EMAIL`, `USER_URI`, etc.), resolve in this order:

1. Check the conversation context for prior values.
2. If missing, run a discovery command first (e.g., `itonics elements list` filtered to the target type, `itonics types list`).
3. If still ambiguous, ask the user to confirm.
4. Then run the target command.
5. Never run commands with unresolved placeholders like `<URI>` or `<EMAIL>`.

## Commands

### Elements

```bash
# List
itonics elements list [-f ODATA_FILTER] [-s SEARCH] [-n LIMIT] [--orderby FIELD] [--select FIELDS] [--raw] [--format json|table]

# Get one
itonics elements get URI [--expand NAV] [--raw] [--format json|table]

# Create
itonics elements create \
  --type URI --label LABEL --created-by EMAIL \
  [--summary TEXT] [--status published|draft|archived|deleted] \
  [--tag TAG]... [--prop URI=VALUE]...

# Update
itonics elements update URI --updated-by EMAIL \
  [--label TEXT] [--summary TEXT] [--status STATUS] \
  [--tag TAG]... [--prop URI=VALUE]...

# Delete (prompts unless --yes)
itonics elements delete URI [URI...] [--yes]
```

### Element types

```bash
itonics types list [-f ODATA_FILTER] [--orderby FIELD] [--format json|table]
itonics types create --label LABEL --created-by EMAIL [--icon NAME] [--prop LABEL:TYPE]...
```

### Files

Two-step under the hood: requests a pre-signed URL from `POST /files`, then PUTs the binary. The CLI strips the `x-amz-acl: private` header advertised by the API because the underlying bucket has ACLs disabled.

```bash
# Upload (prints fileUri). Optionally attach to an element in the same call.
itonics files upload PATH --created-by EMAIL [--content-type MIME] \
  [--attach-to ELEMENT_URI [--attached-by EMAIL]]

# Metadata + short-lived pre-signed download URL
itonics files get FILE_URI
```

### Attachments

```bash
itonics attachments list ELEMENT_URI [--top N] [--skip N] \
  [--orderby "createdOn|fileName|mimeType|size [asc|desc]"]
itonics attachments attach ELEMENT_URI FILE_URI [FILE_URI...] --attached-by EMAIL
itonics attachments detach ELEMENT_URI FILE_URI [FILE_URI...] --detached-by EMAIL
```

`attach` is idempotent per `fileUri`. Not every element type accepts generic attachments — when it doesn't, the API returns `400 BadRequestException: Element type 'X' does not expose the default Attachments field`. In that case set the file as a property value on a `file` / `headerImage`-typed property instead.

### Watches and Likes

Both are managed explicitly by user URI — there is no implicit "current user" from the API key. `userUris` accepts 1–100 entries per write.

```bash
# Watches
itonics watches list ELEMENT_URI [--top N] [--skip N] [--orderby "UserURI asc|desc"]
itonics watches add ELEMENT_URI USER_URI [USER_URI...]
itonics watches remove ELEMENT_URI [USER_URI...]   # empty list = no-op

# Likes ($filter supports UserURI eq|ne ..., likedOn gt|lt 'ISO-8601')
itonics likes list ELEMENT_URI [--filter ODATA] [--orderby "likedOn desc"] [--top N] [--skip N]
itonics likes add ELEMENT_URI USER_URI [USER_URI...]
itonics likes remove ELEMENT_URI [USER_URI...]     # empty list = no-op
```

## Property writes

Properties are passed as `--prop URI=VALUE` (repeatable). The value shape depends on the field type:

- **text / url / number** — send the literal value.
- **select** — send the option URI (look it up with `--raw` reads). Sending the human label is not accepted.
- **rich_text** — send plain HTML (`<p>hello</p>`) or `base64:<base64-of-HTML>`. The `tiptap:<base64>` storage encoding is read-only and rejected on writes.
- **file / headerImage** — set to a `fileUri` produced by `itonics files upload`.
- **relation** — send the related element URI.

## Reading with `--raw` (`rawFieldValues=1`)

With `--raw`, each property is `{ uri, type, label, rawValue: [...] }` instead of a rendered string. `rawValue` is always an array, even for single-value fields, and `rich_text` carries the `tiptap:` storage form. Use `--raw` when you need the canonical option URI of a `select`, the user URI of a `user` field, or to round-trip values.

## Programmatic usage

```python
from itonics_cli import ItonicsClient

c = ItonicsClient(domain, api_key, space)
elements = c.list_elements(filter="elementTypeUri eq 'XXX'", raw=True)
file_uri = c.upload_file("/path/to/logo.png", "user@example.com")
c.attach_files("ELEMENT_URI", [file_uri], "user@example.com")
c.add_watches("ELEMENT_URI", ["USER_URI"])
c.list_likes("ELEMENT_URI", filter="likedOn gt '2026-01-01T00:00:00Z'")
```

## OpenAPI reference

Source of truth: <https://apidocs.itonics-innovation.com/bundled-openapi.yaml>. Schemas worth keeping in mind: `CreateElement`, `UpdateElement`, `ElementProperty`, `CreateFileUploadUrlRequest`, `UploadFileResponse`, `AttachFilesToElementBody`, `PostElementWatchesBody`, `PostElementLikesBody`.

## Instructions

When the user invokes `/itonics $ARGUMENTS`:
1. Parse the user's intent from `$ARGUMENTS`.
2. Resolve missing inputs via discovery commands (see [Required Input Resolution](#required-input-resolution)).
3. Run the appropriate CLI command.
4. Present the results clearly.

If no arguments are given, ask what the user wants to do.
