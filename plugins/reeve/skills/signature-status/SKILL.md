---
name: signature-status
description: "Checks a Reeve e-signature envelope: its status, which recipients have opened and signed it, and a page-by-page preview of the document with its signature fields. Use when someone asks whether a contract or agreement has been signed, who still needs to sign, the status of an envelope, or to see where the signature fields are."
---

# E-signature status through Reeve

## Tools

| Tool | How to call it | What it does |
| :- | :- | :- |
| `sign_get_envelopes` | directly | One envelope by `envelope_id`: its status and its recipients, with each recipient's read and signing status |
| `sign_preview_envelope` | `read` | For an envelope made from an uploaded PDF, renders its pages as images with the signature fields drawn as boxes |

`sign_preview_envelope` runs as `read` with
`{"tool": "sign_preview_envelope", "args": {"envelope_id": "..."}}`.

## Workflow

1. **Get the envelope id.** Both tools need the `envelope_id`. If the user
   hasn't given one, ask for it.
2. **Check the status.** Call `sign_get_envelopes` with the `envelope_id`.
   Report the envelope's status, then each recipient: who has signed, who
   has opened it but not signed, and who hasn't opened it.
3. **Preview, when asked.** To show the document or check where the fields
   sit, call `sign_preview_envelope`. The preview is always of the original
   uploaded file, so a completed envelope shows the unsigned document with
   its field boxes.

A "not found" result means the envelope doesn't exist or belongs to another
account. Ask the user to check the id.
