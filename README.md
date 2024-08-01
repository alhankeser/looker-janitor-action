[![Action Tests](https://github.com/alhankeser/looker-janitor/actions/workflows/action_tests.yml/badge.svg)](https://github.com/alhankeser/looker-janitor/actions/workflows/action_tests.yml) [![Python Tests](https://github.com/alhankeser/looker-janitor/actions/workflows/tests.yml/badge.svg)](https://github.com/alhankeser/looker-janitor/actions/workflows/tests.yml)

# Looker Janitor

The GitHub Action to clean your Looker LookML view files.

- [Quick Start](#quick-start)
- [Features](#features)
- [Usage Examples](#usage-examples)
- [Inputs](#inputs)

## Quick Start

Add this as a step to a workflow file:
```yaml
- name: Run Looker Janitor
  uses: alhankeser/looker-janitor-action@v0
```

See more advanced usage examples below

## Features

- Order field types (e.g. filters, dimensions, measures)
- Order fields (including by label and localized labels if need be)
- Order field parameters (e.g. type, label, description, sql...)
- Move primary key to top of dimensions list
- Check for missing field parameters and report in `%f:%l: %m` format

## Usage Examples
End-to-end usage example:
```yaml
name: Looker Janitor
on:
    pull_request:
        types:  [opened, synchronize]

jobs:
    run-looker-janitor:
        permissions:
            contents: write
        runs-on: ubuntu-latest
        steps:
            - name: Run Looker Janitor
              id: looker_janitor
              uses: alhankeser/looker-janitor-action@v0
              with:
                check_required_params: false
                required_dimension: |
                  hidden
                  type
                required_measure: |
                  value_format_name
            - name: Get Changed Files
              id: changed_files
              uses: tj-actions/changed-files@v44
              with:
                files: |
                    **.view.lkml
                    **.view.lookml
            - name: Commit Cleaned Files
              shell: bash
              run: |
                for file in ${{ steps.changed_files.outputs.all_changed_files }}; do
                    git add "${file}"
                done
                git commit -m "Run Looker Janitor" || continue
                git push
            - name: Setup Review Dog
              uses: reviewdog/action-setup@v1
            - name: Run Review Dog
              shell: bash
              env:
                REVIEWDOG_GITHUB_API_TOKEN: ${{ secrets.GITHUB_TOKEN }}
              run: |
                cat looker-janitor/output.txt | reviewdog -efm="%f:%l: %m" -reporter=github-pr-annotations -level warning -filter-mode=nofilter
```

By default, any changed view.lookml/lkml files will be updated by Looker Janitor.
You can choose to override that by passing a custom list of files:
```yaml
- name: Run Looker Janitor
  uses: alhankeser/looker-janitor-action@v0
  with:
    files: |
      customers.view.lkml
      orders.view.lkml
```

Override the default field type ordering and put dimensions above filters:
```yaml
- name: Run Looker Janitor
  uses: alhankeser/looker-janitor-action@v0
  with:
    type_order: |
      dimension
      filter
      measure 
```

Turn off type ordering entirely:
```yaml
- name: Run Looker Janitor
  uses: alhankeser/looker-janitor-action@v0
  with:
    order_types: false
```

Provide a localization filepath to order labels by localized label value:
```yaml
- name: Run Looker Janitor
  uses: alhankeser/looker-janitor-action@v0
  with:
    localization_file_path: samples/en.strings.json
```

Don't order fields by their labels and instead use field names:
```yaml
- name: Run Looker Janitor
  uses: alhankeser/looker-janitor-action@v0
  with:
    order_fields_by_label: false
```

Turn off field ordering entirely:
```yaml
- name: Run Looker Janitor
  uses: alhankeser/looker-janitor-action@v0
  with:
    order_fields: false
```

Set an order in which field parameters should be sorted:
If only a subset of paraters are provided, all remaining parameters will remain in their current order.
```yaml
- name: Run Looker Janitor
  uses: alhankeser/looker-janitor-action@v0
  with:
    param_order: |
      type
      label
      description
      sql
```

Override the default and don't put primary key first in list of dimensions:
```yaml
- name: Run Looker Janitor
  uses: alhankeser/looker-janitor-action@v0
  with:
    primary_key_first: false  
```

Check for required parameters and specify what parameters are required for measures:
```yaml
- name: Run Looker Janitor
  uses: alhankeser/looker-janitor-action@v0
  with:
    check_required_params: true
    required_dimension: |
      type
      label
    required_measure: value_format_name
- name: Print Output
  shell: bash
  run: |
    cat looker-janitor/output.txt
```
The contents of looker-janitor/output.txt is in the following format `%f:%l: %m` ([errorformat](https://github.com/reviewdog/reviewdog/blob/master/README.md#errorformat)):
```
samples/example_input.view.lkml:29: dimension 'customer_id' missing label
samples/example_input.view.lkml:34: dimension 'order_date' missing label
samples/example_input.view.lkml:39: dimension 'order_status' missing label
samples/example_input.view.lkml:52: measure 'average_order_value' missing value_format_name
samples/example_input.view.lkml:59: measure 'count' missing value_format_name
samples/example_input.view.lkml:65: measure 'total_sales' missing value_format_name
```

When combined with [reviewdog](https://github.com/reviewdog/reviewdog), you can get something that looks like this:
![image](https://github.com/user-attachments/assets/35d2dc9f-2bcf-414d-a17c-01bbd1712c71)

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
check_required_params | false | boolean | Should required field parameters be checked and output to looker-janitor/output.txt | false
required_filter | false | string | List of required filter parameters. If missing, reported in output. | type
required_parameter | false | string | List of required parameter parameters. If missing, reported in output. | type
required_dimension | false | string | List of required dimension parameters. If missing, reported in output. | type
required_dimension_group | false | string | List of required dimension_group parameters. If missing, reported in output. | type
required_measure | false | string | List of required measure parameters. If missing, reported in output. | type
required_set | false | string | List of required set parameters. If missing, reported in output. | fields
