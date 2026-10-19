# Working in a Navigator Project repository

## Critical: email is draft-only

**Every email must remain a draft. An agent never sends, replies to, or forwards email.** The lawyer reviews each draft
and sends it personally. A request to send, reply, or forward means prepare a draft for lawyer review instead.

This repository holds source-only material for one Navigator Project defined in `navigator.yaml`. Project identity and
matter coordinates belong in `navigator.yaml` and Navigator's live Project row, not in this contract.

## Find the term first

Before naming a table, field, state, or UI concept, run `navigator glossary list` to find a term and its plain-language
description, then `navigator glossary show "<Term>"` for its full definition. Use the glossary's term when one fits; if
none fits, add a term under `docs/glossary/` in the same Navigator change rather than inventing a synonym.

Run `navigator project sync` to download Project documents visible to you into this checkout.

Run `navigator project onboard` to draft or open this Project's onboarding letter, and `navigator project offboard` to
draft or open its offboarding letter. Each command reads the Project code and client DRI from `navigator.yaml`.

Run `navigator project gate` before every commit; it is the required gate for every proposed repository change.

Run `navigator project repository sync-skills` from an existing Project repository to restore missing or drifted
canonical Agent Skills and this `AGENTS.md`. This offline repair changes only those canonical files; keep custom skills
beside them. `navigator project gate` verifies the result and does not repair them.

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
  decision. Confirm portal copy under `apps/portal/` is final and client-safe. For each Deadline, choose deliberately
  whether it is internal or client-visible and review that choice.
* The required check is `ci`, which runs `navigator project gate`. Publishing a page or a Draft means merging its pull
  request.
* The forge enforces branch and merge settings; client-visibility decisions remain part of human review. A pre-commit
  hook is a convenience only.

## Folders

* `documents/` holds Project documents. Only their YAML metadata files (`.yaml`) are stored in Git; document bytes
  remain in Navigator-managed storage and are downloaded to ignored paths.
  * `documents/onboarding/` holds the engagement letter that opens the Project; `documents/offboarding/` holds the one
    that closes it. A signed copy is named `<name>_signed.pdf` and its pointer carries `docusign_envelope_id`.
  * `documents/invoices/` holds the firm's invoices to the client, named `INV-<digits>.<ext>`; every pointer carries
    `xero_invoice_id`.
* `apps/` holds the firm's front-end applications.
  * `apps/portal/` holds the Project's Portal, the default client experience, served at `/app/projects/<code>/portal/`.
* `seeds/` holds seed data that `navigator site import` creates or updates.
* `drafts/` holds Drafts, the notation blueprints; `templates/` is its alias spelling, and the gate warns until it is
  renamed. Files sit directly under it. Memo notations use `kind: memo`; their reusable source belongs here and is
  reviewed in Git. Use placeholders for client data and matter facts, whose answers and produced work product stay in
  Navigator. A Notation is a database record, never a file here.
* `notes/` is git-ignored. Write call notes and memos there for `navigator project communication create --body-file`;
  the Communication's body lives in Navigator, never in Git.

## Final documents only

Every document this repository produces for a client, a court, a counterparty, or a signer is final. Each fact is filled
in and each choice is made before the document leaves Navigator. Never deliver a fill-in-the-blank document: no blank
lines, brackets, `[INSERT]` markers, unfilled `{{placeholders}}`, TODOs, or notes to the drafter or the reader.

* A `{{placeholder}}` belongs only in a Draft under `drafts/`; Navigator fills it from answers when
  it renders the document.
* When a fact is missing, ask for it through a notation questionnaire or a Portal form and render again. Do not invent
  it, and do not ask the reader to write it in by hand.
* Signature, initial, date-of-signing, and notary blocks are the only spaces a person completes, and only at signing.
* Read the rendered pages before calling a document ready to send, sign, or file.

## Tools

Use available MCP servers and the `navigator` CLI, preferring them whenever they provide the needed operation. Never
substitute browser or native-app automation for a server that is available.

* **Navigator CLI** — the system of record; run `navigator project gate` before every commit.
  Use `navigator project sync` to download Project documents.
* Document bytes, archived authorities, and answers go through Navigator and never into Git. Memo notations are
  authored source: review and commit their reusable templates, then run `navigator notation preview drafts/<code>.md`.
* A direct document upload from this matching checkout writes only its source pointer under `documents/`; review and
  commit that `.yaml` file.
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

When a task requires a client email, use `navigator mail draft --body-file` to create an unsent draft. Draft only the
substantive message body: do not add a complimentary close, typed name, title, bar admission, contact block, or
signature. The attorney reviews the complete draft in Gmail before any separate sending action. The CLI removes the `--`
signature delimiter and an exact trailing signature configured in `NAVIGATOR_GMAIL_SIGNATURE`, but preserves common
sign-offs. Arbitrary undelimited prose cannot be identified reliably as a signature, so inspect the draft before relying
on this cleanup.

## Feedback

When Navigator's CLI is missing or wrong, open a Linear issue on the Lawyers team rather than documenting a CLI
workaround here.
