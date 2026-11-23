# Spaces tools

Every tool the Spaces connector offers, grouped by job. The connector's tool descriptions have the full rules.

## Finding your way
- `get_my_context`: call first. Who you act for, their spaces (id, name, description, role, `can_change`), how many
  lists, notes and cheat sheet topics each holds, `is_new`, `joined_recently`, and `invites` (the invites that wait
  for the user's answer). For choosing a space: it names nothing inside one. `has_instructions: true` marks a space
  whose editors wrote instructions. Pass `include_account` for the plan, usage and connection restrictions,
  `include_archived` for archived spaces, and `cursor` past 200 spaces.
- `get_space_overview`: what one space holds: its instructions (null when it has none), its members (user_id, name,
  role), its lists (id, title, first line of the description, field keys), its notes (id and title) and its cheat sheet
  topics. Headers only. Call it in the chosen space before you add, change or create anything. When `next_cursor` is
  not null, call again with `cursor`. `space_version` is the `expected_version` for `set_space_instructions`.
- `get_list_schema`: one list's full schema, without items: description, every field with options, hidden fields,
  display, sort, saved totals, formulas and version. Read it before `update_list_schema`, and before adding items when
  the description holds rules (`description_more` in the overview).

## Lists
- `create_list`: a new list in one call: title, description, fields (with keys), display, sort, totals and first
  items. Fails, and gives that list's id, when a live list already has the title.
- `update_list_schema`: change what a list is: title, description, fields (add, rename, hide, restore, change a text
  field to number or date), field order, display, sort and totals. Needs `expected_version` from `get_list_schema`.
  Never adds or removes items. Editors only.
- `show_list_items`: show a list's items to the user (a card in hosts with cards), with filter, status, query, sort and
  limit. Summaries and computed values come with it.
- `query_list_items`: rows and totals of a list for your own reasoning: a small table as text, nothing shown. Totals
  per group (for example per day) in one call; `with_history` adds who added and changed each item.
- `add_list_items`: add 1 to 50 items to a list that exists. A title that matches no list fails and gives your
  items back: create the list with `create_list`.
- `update_list_items`: change 1 to 50 items: titles, ticks, values. All-or-nothing.
- `archive_list_items`: archive 1 to 50 items (restorable in the web app).
- `set_list_formula`: numeric computed columns and rolling windows, with a preview. Editors only.
- `upload_image`: add a photo (only when the server has photos turned on).

## Notes
- `get_note`: a note's Markdown text and its version.
- `create_note`: a new note with a title and an optional body.
- `append_to_note`: add text to the end of a note. It cannot lose text; prefer it.
- `update_note`: replace a note's title or its whole body. A new body needs `expected_version`: read with `get_note`
  first and keep what should stay.
- `archive_list_or_note`: archive one whole list or one note (restorable).

## Cheat sheet
- `search_cheat_sheet`: free-text search in any language, in one space or every space. Use before stating a value
  or asking. Pass `topic` or `item_id` for one exact entry, or only `space_id` to page through every entry.
- `set_cheat_sheet_entry`: save one entry. Short facts only (text up to 200 characters, numbers, true or false);
  `script_key` links the script that computes it. A rule goes in the space's instructions, long text in a note.
- `archive_cheat_sheet_entry`: archive one entry, by `topic`, `item_id` or `entry_id`.

## Spaces
- `create_space`: a new space with its description and lists; each list takes `create_list`'s inputs. Ask what it
  should hold first when the user has not said.
- `set_space_description`: set or clear what a space is for (one or two sentences, read by assistants to choose
  between spaces). Only when the user asks or agrees.
- `set_space_instructions`: set or clear the space's instructions (Markdown, at most 2,000 characters). Every member's
  assistant reads them, so only when the user asks or agrees. Editors only.

## Sharing
- `list_members_of_my_spaces`: the people the user already shares a space with, to find someone by name.
- `add_or_invite_to_space`: add a co-member at once, invite another email, or make an open invite link. `outcome`
  says which. Read its description first. An email invite is emailed to its address when `emailed` is true.
- `change_or_remove_space_member`: change another member's role (editor or viewer), or remove them. Editors only.
- `leave_space`: the user leaves a space themself.
- `accept_or_decline_space_invite`: answer an invite from `get_my_context` (`invite_id` and `response`), or join with
  a link the user gives you (`invite_link`). Only after the user decides.

## People and network
- `search_people`: find people by words or filters; one person in full with `person_id` ("me" is the user); who knows
  whom with `include_connections`; introductions with `introduced_by`.
- `create_person`, `update_person`: add or change a person. `search_people` first.
- `set_person_connection`: record or change who knows whom, and who introduced them.
- `archive_person_or_connection`: archive one person or one connection (restorable).

## Scripts
- `get_scripts`: the scripts in a space (newest version of each, with description, warning, list, sources, the cheat
  sheet entries it computes and its last run); with `key`, one script's card with its code and last runs; with
  `run_id`, one run with its output and provenance.
- `set_script`: save a new version of a JavaScript script. A script bound to a list runs by itself each time a row of
  that list is added or changed. Editors only.
- `run_script`: run a script now, for one input or for items on its list. Use it after a new script version or a
  source refresh. Editors only.

## Lookup sources
- `set_lookup_source_lock`, `stage_lookup_source_update`, `accept_lookup_source_update`: locked and external sources.
  Editors only.

## History, undo, and support
- `get_change_history`: who changed what and when, including which assistant. Filter by list, note, item, assistant
  (`client`), words, or time. Each change has `group_id` and `undoable`. With `at`, a list, note, item or the whole
  space as it was at that time, shown as what changed since, with the `expected_versions` that `restore_version` needs.
- `undo_change`: take back one change (`change_id`) or one whole call (`group_id`). Fails with `undo_conflict` if the
  same thing changed again later.
- `restore_version`: put back what changed since a time: everything, or only `entity_ids`. Things added since are
  archived, not deleted. Take it back with `undo_change` and its `group_id`.
- `report_bug`: a bug report about Spaces itself, with steps, expected, and actual. Get the user's OK first.
