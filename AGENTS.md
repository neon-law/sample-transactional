# Working in a Navigator Project repository

## Critical: email is draft-only

**Every email must remain a draft. An agent never sends, replies to, or forwards email.** The lawyer reviews each draft
and sends it personally. A request to send, reply, or forward means prepare a draft for lawyer review instead.

This repository is the source of one Navigator Project, named in `navigator.yaml`. Git is its history: every change to
the matter's pages, Drafts, and document pointers is a reviewed commit. Declare the change in YAML or Markdown; do not
write application code. When a declaration cannot say what the matter needs, open a Linear issue for a new Section or
gate rule rather than working around it.

## Find the term first

Before naming a table, field, state, or UI concept, run `navigator glossary list` to find a term and its plain-language
description, then `navigator glossary show "<Term>"` for its full definition. Use the glossary's term when one fits; if
none fits, add a term under `docs/glossary/` in the same Navigator change rather than inventing a synonym.

## The loop

1. `navigator project sync` downloads the Project documents visible to you.
2. Edit `navigator.yaml`, `drafts/`, or `documents/` pointers.
3. `navigator project preview` renders the Matter Pages from the working tree.
4. `navigator project gate` before every commit; it is the required check.
5. Open a pull request. Merging it publishes.

For architecture or process questions, convene the `council` skill; for how a client will read a page or letter,
`client-council`; for the legal substance, `legal-council`. Keep each answer terse.

`navigator project onboard` and `navigator project offboard` draft or open the engagement and offboarding letters from
`navigator.yaml`. `navigator project repository sync-skills` restores missing or drifted canonical Agent Skills and this
`AGENTS.md`; keep custom skills beside them.

## Branches

* `main` is the only long-lived branch. Change it only through a pull request; never push to it directly.
* Pull requests are squash-merged, and the head branch is deleted on merge.
* Open pull requests ready for review by default; use a draft only when deliberately holding work for more changes.
  On GitHub, run `gh pr merge <number> --auto --squash` if auto-merge was not armed automatically, then confirm
  `autoMergeRequest` is non-null or the pull request is already merged. On another forge, use its equivalent
  ready-for-review and merge-when-checks-pass controls, and verify the resulting pull-request state and merge policy
  there.
* Before merging, review every changed document pointer (`documents/**/*.yaml`): retain `visibility: internal` unless
  the client is meant to read that document. Treat every change to `visibility: client` as a deliberate client-release
  decision. Review every change to a `client:` lens under `pages:`; it is what the client sees. For each Deadline,
  choose deliberately whether it is internal or client-visible.
* The required check is `ci`, which runs `navigator project gate`.
* The forge enforces branch and merge settings; client-visibility decisions remain part of human review. A pre-commit
  hook is a convenience only.

## Folders

* `navigator.yaml` names the Project and declares its Matter Pages under `pages:`: each page has a slug, a kind, and
  the Sections each lens (`lawyer`, `client`) shows. Navigator renders them at `/app/projects/<code>/<slug>`.
* `documents/` holds YAML pointers only; document bytes stay in Navigator-managed storage.
  * `documents/onboarding/` and `documents/offboarding/` hold the letters that open and close the Project. A signed
    copy is named `<name>_signed.pdf` and its pointer carries `docusign_envelope_id`.
  * `documents/invoices/` holds invoices named `INV-<digits>.<ext>`; every pointer carries `xero_invoice_id`.
* `drafts/` holds Drafts, the notation blueprints, directly under it; `templates/` is an alias the gate warns about.
  Memo notations use `kind: memo`. Use placeholders for client data and matter facts; answers and work product stay in
  Navigator. A Notation is a database record, never a file here.
* `seeds/` holds seed data that `navigator site import` creates or updates.
* `notes/` is git-ignored. Write call notes there for `navigator project communication create --body-file`.

## Final documents only

Every document this repository produces for a client, a court, a counterparty, or a signer is final.

* Never deliver a fill-in-the-blank document: no blank lines, brackets, `[INSERT]` markers, unfilled `{{placeholders}}`,
  TODOs, or notes to the drafter or the reader.

* A `{{placeholder}}` belongs only in a Draft; Navigator fills it from answers when it renders the document.
* When a fact is missing, ask for it through a notation questionnaire and render again. Never invent it.
* Signature, initial, date-of-signing, and notary blocks are the only spaces a person completes, and only at signing.
* Read the rendered pages before calling a document ready to send, sign, or file.

## Tools

Use available MCP servers and the `navigator` CLI, preferring them whenever they provide the needed operation. Never
substitute browser or native-app automation for a server that is available.

* **Navigator CLI** — the system of record.
  Document bytes, archived authorities, and answers go through Navigator and never into Git. Preview a memo with
  `navigator notation preview drafts/<code>.md`. A direct upload from this checkout writes only its pointer under
  `documents/`; review and commit that `.yaml` file.
* **Notion** — writing: handbooks, guides, and intake notes.
* **Gmail** — client instructions and correspondence. Read messages and create drafts only; never send, reply to, or
  forward an email.
* **Google Calendar** — meetings and deadlines.
* **Google Drive** — meeting notes, transcripts, and large document intake.
* **Slack** — private channel for updates.
* **Linear** — CLI feedback, named by mechanism and never by matter.
* **CourtListener** — dockets, filings, parties, and judges.
* **Midpage** — statutes and rules.
* **Descrybe** — case law.
* **Exa** — the open web, never primary law.

The last four leave the firm. Keep client identities and matter facts out of those queries.

For a client email:

* Run `navigator mail draft --body-file` to create an unsent draft of the substantive message body.
* Add nothing else: do not add a complimentary close, typed name, title, bar admission, contact block, or signature.
* The attorney reviews the complete draft in Gmail before any separate sending action.
* The CLI strips the `--` signature delimiter and an exact trailing `NAVIGATOR_GMAIL_SIGNATURE`, but not other prose, so
  inspect the draft.

## Feedback

When Navigator's CLI is missing or wrong, open a Linear issue on the Lawyers team rather than documenting a CLI
workaround here.
