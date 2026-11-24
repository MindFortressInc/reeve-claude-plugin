---
name: crm-contacts
description: "Finds and updates contacts in the user's Reeve CRM: look a person up by name, email, phone, tag or lifecycle stage, read their full record and interaction history, change their name, tags, lifecycle stage or owner, and add a note. Use when someone asks to find a contact or customer, look someone up, see what we last discussed with someone, tag or re-stage a contact, or log a note about a person."
---

# CRM contacts through Reeve

## Tools

| Tool | How to call it | What it does |
| :- | :- | :- |
| `crm_list_contacts` | directly | Lists contacts, filtered by `q` (name, email or phone), `tags_any`, `tags_all` or `lifecycle`, sorted and paginated |
| `crm_list_contacts_search` | directly | Searches contacts by what is known about them (`q`), including notes and interactions |
| `crm_get_contacts` | `read` | One contact's full record: identifiers, extensions and relationships |
| `crm_list_contacts_interactions` | `read` | One contact's message and call history, newest first |
| `crm_update_contacts` | `act` | Changes a contact's `display_name`, `tags`, `lifecycle_stage` or `owner_user_id`. Only the fields passed change |
| `crm_create_contacts_notes` | `act` | Adds a note (`body`) to a contact's timeline |

Tools marked `read` run as `read` with `{"tool": "<name>", "args": {...}}`.
Tools marked `act` run the same way through `act`.

## Workflow

1. **Find the contact.** For a name, email or phone, call `crm_list_contacts`
   with `q`. For a description ("the caterer we spoke to in March"), call
   `crm_list_contacts_search` with `q`. If several contacts match, show them
   and ask which one.
2. **Read it.** Call `crm_get_contacts` with the `contact_id` for the full
   record. For "what did we last talk about", call
   `crm_list_contacts_interactions` with the `contact_id`.
3. **Update it, once the user has confirmed the change.** Show the contact
   and exactly what will change, then call `crm_update_contacts` with the
   `contact_id` and only the fields that change. Passing `tags` replaces the
   contact's whole tag list, so include the tags it already has that should
   stay. When the change includes `tags`, call `crm_get_contacts` again just
   before `crm_update_contacts` and build the new list from the tags it
   returns. If they differ from the tags you showed, show the revised change
   and ask again before writing. For a note, call `crm_create_contacts_notes`
   with the `contact_id` and the `body`.
4. **Report the result.** Confirm what changed from the tool's response. If
   the call returns `pending_approval` with a `gate_id`, tell the user and
   call `resolve_gate` with that `gate_id` after they approve.
