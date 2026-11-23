# Lists, formulas, scripts, and sources

## Reading a list before changing it
`get_space_overview` gives each list's id, title, the first line of its description and its fields (key, name, type).
`get_list_schema` gives the whole schema: `record.description` (purpose and rules), `record.fields` with options,
the hidden fields, `record.display`, `record.sort`, saved totals, formulas, `record.has_status_field` and the version.
Use those field keys in `values` in `add_list_items` and `update_list_items`. quantity, unit, status, notes, and url
work only when that field is on the list. An unknown key is rejected with the list's fields.

## Making and changing a list
- `create_list` makes a list in one call: title, description, fields (each with a key), display, sort, totals and
  first items. Give an amount, a date or a quantity its own number or date field, not text in the title.
- `add_list_items` never makes a list. A title that matches no list fails and gives the items back.
- `update_list_schema` changes what a list is: title, description, fields (an entry with no key adds one, with an
  optional `new_field_key` for its key; with a key, rename, hide, restore, or change a text field to number or date),
  field order, display, sort and totals. It needs `expected_version` from `get_list_schema`. Hiding keeps the values;
  no tool erases them.

## Checklist vs table
- Only `display.style` checklist is a checklist. Logs and records (meals, weights, readings, journals) use table.
- On other lists, do not filter by status or set it.
- Setting status on a list without a status field is a validation error.
- When the user names one part of a list ("I'm at the supermarket"), call `show_list_items` with `filter` on the field
  that holds it, so that only those items are shown. Do not add other items unless the user asks.

## Numbers
A number field takes a JSON number or a plain numeric string ("450"). "450 kcal" is rejected. Null clears a value and
is not 0. `create_list`, `add_list_items` and `update_list_items` may return `cheat_sheet_warnings` when a number
differs from a saved one:
check those with the user.

## Totals
`totals` in `create_list` and `update_list_schema` saves sum, avg, min, max, or count of a number field, optionally
grouped; `update_list_schema` takes the whole list of totals the list should keep. Values in `summaries` from
`show_list_items` say how many values were included and skipped. Null and text are skipped, not counted as 0. For a
total of your own, without saving it, use `totals` in `query_list_items`.

## Formulas
`set_list_formula` saves or previews (`preview: true` saves nothing) a numeric CEL column such as `qty * price`, or a
rolling calendar-day window. Results are in `items[].computed` by formula key; definitions, timezone, and provenance
are in `formulas[]`.
- Null is not zero; each null has a `reason` (for example lookup_no_match or budget_exceeded, which is partial).
- Rolling windows report coverage k/N. Missing days are not zero. Today is excluded unless include_partial_day.
- Replacing or deleting a formula needs its key and the current `expected_version`.

## Lookups
A formula can look up a number in another list or the cheat sheet of the same space (lookup, lookup_range).
No match is an explicit null, not 0.

## Scripts
- `set_script` saves a new version of a JavaScript script (`function main(input, ctx)`). It has no network, files,
  clock, or randomness. Editors only.
- A script bound to a list runs by itself each time a row of that list is added or changed, and its result is saved
  with the row. The `add_list_items` or `update_list_items` summary says which fields it filled, or that it failed.
- Output types: number shows on the list in `items[].computed` under the script's key; fields fills the row's fields
  named in `writes`; json is read with `get_scripts` and the run's `run_id`.
- A script value in `items[].computed` has `provenance.script_key`, `script_version`, `status`, and `computed_at`.
  It is current only when `status` is ready. When it is pending, stale, or error, say so; an error value is null and
  `provenance.error` says why.
- `run_script` runs a script now. Saving a row already runs the list's scripts, so use it after a new script version
  or a source refresh.

## Locked and external sources
- `set_lookup_source_lock` locks a list or the cheat sheet to a snapshot. Unlocking first reports dependent formulas;
  confirm with the user before retrying with `confirm_dependents`. An optional `source.key` names the column that
  keeps row identities when a refresh is accepted (title for a cheat sheet).
- External sources are read-only and cannot be unlocked.
- `stage_lookup_source_update` stages a new snapshot and changes nothing. There is no automatic acceptance; an
  editor accepts it with `accept_lookup_source_update`.
- With `source.key` set, accepting matches rows on that column: matching rows keep their ids and are updated, new keys
  are added, and missing keys are archived. The result reports added, updated, removed, and unchanged. Only without a
  key are all rows replaced with new ids.
- Formulas reading a locked or external source use its accepted snapshot. Tell the user the data-as-of date shown on
  the value (source, date, computed_at), not a dataset version.
