# Looker Janitor

The GitHub Action to tidy those dirty Looker LookML view files.

## What it does

These are all default behaviors that can be turned off or customized:
- Groups filters, parameters, dimensions and measures and orders them in any order you'd like.
- Puts primary key at top.
- Sorts fields within their parent groups by field name.
- Warns when a field is missing a specific paramater(s) that you provide.
- Sorts parameters of each field.

### Future:

- Sort fields by label instead of field name, as option.
- Groups fields by group_item_label.

## Inputs

|name|required|type|default|description|
|--|--|--|--|--|
|files|no|string|""|List of space-delimited relative file paths. Example: `views/orders.view.lkml views/customers.view.lkml`. If empty, will clean any changed files that end in `.view.lkml` or `.view.lookml`.|

## Outputs

List of warnings formatted as `{file_path}:{line_number}: {message}\n`
