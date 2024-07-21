[![Action Tests](https://github.com/alhankeser/looker-janitor/actions/workflows/action_tests.yml/badge.svg)](https://github.com/alhankeser/looker-janitor/actions/workflows/action_tests.yml)

[![Python Tests](https://github.com/alhankeser/looker-janitor/actions/workflows/tests.yml/badge.svg)](https://github.com/alhankeser/looker-janitor/actions/workflows/tests.yml)

# Looker Janitor

The GitHub Action to clean your Looker LookML view files.

## Quick Start

Add this as a step to a workflow file:
```--yaml
- name: Run Looker Janitor
  uses: alhankeser/looker-janitor-action@v0.3.7
```

## What it does

- Groups and sorts field types (filters, parameters, dimensions and measures).
- Puts primary key at top.
- Sorts fields within their parent groups by field name or label (even if it's localized).
- Sorts parameters of each field.

## Inputs

|name|required|type|description|default|
|--|--|--|--|--|
files | false | string | List of files to clean. If omitted, then any changed view files will be cleaned. | ''
type_order | false | string | Order of field types. | filter<br>parameter<br>dimension<br>dimension_group<br>measure<br>set<br>
param_order | false | string | Order of field parameters. | hidden<br>type<br>view_label<br>group_label<br>group_item_label<br>label<br>description<br>sql<br>sql_start<br>sql_end<br>filters<br>value_format_name<br>value_format<br>drill_fields<br>
primary_key_first | false | boolean | Should primary key be ordered first. | true
order_types | false | boolean | Should fields be ordered by their type. | true
order_fields | false | boolean | Should fields be ordered. | true
order_field_parameters | false | boolean | Should field parameters be ordered. | true
order_fields_by_label | false | boolean | Should fields be ordered by their labels. | true
localization_file_path | false | string | Localization file path to use for label sorting | None

## Example usage

