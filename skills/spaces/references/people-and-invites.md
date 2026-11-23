# People, members, and invites

## People (a space's network)
- `search_people` first, every time, before `create_person`. If the person exists, use `update_person`.
- `create_person` returns `possible_duplicates`; if one is the same person, tell the user and use it.
- Check the cheat sheet (`search_cheat_sheet`) for the space's conventions, such as how to spell countries.
- "me" is the user (`search_people` with `person_id` and `update_person` accept it; `is_me` marks them in results).
- Details about a person go on that person (`update_person`), not on the cheat sheet.
- "Barak introduced me to Edo": `set_person_connection` from "me" to Edo, `introduced_by` Barak. Add anyone missing
  first. If the two are already connected, the same call updates the connection.
- `search_people` with `include_connections` or `introduced_by` answers "who do I know in New York?" or "who did
  Barak introduce me to?". With `person_id` it gives one person in full, with their connections.
- `archive_person_or_connection` archives one person or one connection (restorable in the web app).

## Members and roles
- Roles are editor (can change anything, including members) and viewer (view only).
- `get_space_overview` lists a space's members with their user_id and role.
- Before `change_or_remove_space_member`, tell the user exactly who and what, and get their OK.
- To leave a space, the user calls `leave_space`; `change_or_remove_space_member` refuses the user's own id.
- A space always keeps at least one editor.

## Invites
- Read the `add_or_invite_to_space` description before inviting anyone.
- Someone the user already shares a space with (see `list_members_of_my_spaces`) is added right away by user_id or
  email. Anyone else gets an email invite tied to that address, or an open link if you omit both. Ask the user which
  they want. `outcome` in the answer says what happened.
- Spaces emails an email invite to that address, and the answer says `emailed`. When `emailed` is true, tell the user
  it was sent; the url is a fallback if it does not arrive. When it is false (always for an open link, and when an
  email limit was reached), give the user the url to send, and do not say it was emailed.
- Invites waiting for the user come in `get_my_context` (`invites`): tell them, and call
  `accept_or_decline_space_invite` only after they decide. A link the user gives you joins with `invite_link`.

## Newly shared spaces
`get_my_context` marks new spaces in two ways:
- `joined_recently`: the user was invited or added to the space in the last 7 days and has not read it yet. Introduce
  it in 2-3 sentences and suggest one first action (a read if the role is viewer). This is also set when the user
  accepted an invite themselves.
- `is_new`: a co-member added the user and they have not opened the space yet. Tell the user it was shared with
  them.

Reading the space (for example `get_space_overview` or `show_list_items`) clears both. `get_my_context` does not.
