- Feature Name: extended_search_filters
- Start Date: 2025-01-06
- RFC PR: [amundsen-io/rfcs#0000](https://github.com/amundsen-io/rfcs/pull/0000) (after opening the RFC PR, update this with a link to it and update the file name)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Extended Search Filters

## Summary

We're looking to improve the advanced search experience in Amundsen by providing two new ways to filter data in the UI, namely the radio select filter and dropdown select filter.

## Motivation

### Why are we doing this?

One pain point we've found with Amundsen's advanced search is that for new users, the experience can be quite daunting, and asking even simple questions requires some specific knowledge of the cluster-database-schema-table naming scheme. Some of this pain stems from the fact that all current search filters are free text - users have to enter in the exact (or wildcarded) expression they are filtering on.

### What use cases does it support?

In our experience we've found this useful for fields that are sparse in value, and so showing all options to the user is more effective than providing them the ability to filter down with free text.

Depending on implementation, this can also support dynamic fields that can fill options from the react state. In our use-case, this was used to provide context dependant dropdown options for table tags of the form `KEY = VALUE`.

### What is the expected outcome?

The expected outcome is that amundsen developers are provided with more configuration to customise their advanced search experience.

## Guide-level Explanation (aka Product Details)

Amundsen's advanced search page contains multiple filters on the left hand side of the page. Deciding what particular filters to show is decided via configuration provided in `frontend/amundsen_application/static/js/config/`. In particular the AppConfig key `resourceConfig[<resource>][filterCategories]`.

This is an array of `FilterConfig` objects which determine what specific filters can be shown and how they are rendered.

Two of these filters focused on removing friction between your users and the search page are the **radio select** and **dropdown options**.

### Radio Select

The radio select will render multiple options of your choosing, allowed you to select/deselect these options, and have the page automatically rerender the search results based on these options. For example, suppose we've got many tables in our system, but these all live in a few main database locations, then a radio select is probably the correct filter category to use:

![Example of radio filter](/assets/000/radio-example.png)

Radio selects can support multiple selections, or just a single selection at a time. The options rendered is also completely configurable, and can show an icon alongside the option. If you'd like to provide an option for 'everything else' that will simply match anything not covered by the visible options, you can include that too (This is the 'Other' option in the image above).

### Dropdown Select

The dropdown select will render a searchable dropdown which users can type into and select their preferred options. The contents of this dropdown are also configurable.

For example, suppose we've got about 300 schemas in our system, which is too much for a radio button scheme, but not enough to warrant avoiding a better experience than freetext entirely. We can show the schemas in a searchable dropdown, allowing the users to see all schemas available.

![Example of Dropdown search filter](/assets/000/search-example.png)

This could also rerender the search options based on the database filter values mentioned above! So when the snowflake checkbox isn't checked, snowflake schemas would not be shown in the dropdown.

One more example of dynamic rendering for search is in table tags. Users might have tagged resources with a `KEY=VALUE` format, so  `my_schema.my_table` might have tags `ETL_OWNER=hsimpson`, `YEAR_INTRODUCED=2022`, etc. In order to adequately filter by this in our search page, you might have two dropdowns; One for selecting the tag key, and another for selecting the tag value.

![Example of tag key options](/assets/000/tag-key.png)
![Example of tag value options](/assets/000/tag-value.png)

The tag value dropdown can then render it's search options based on the option selected for tag key.

## UI/UX-level Explanation

Largely the same as the Guide-level explanation - The advanced search page has the ability to show more search options. See the guide level explanation for example images of UI.

## Reference-level Explanation (aka Technical Details)

Adds two more filter categories in the config types:

* `RadioFilterCategory`
* `DropdownFilterCategory`

### RadioFilterCategory

`RadioFilterCategory` has the following extra configuration keys:

```ts
interface RadioFilterCategory extends BaseFilterCategory {
  getOptions: (state: GlobalState) => RadioOption[]; // Function to determine what options to render
  wildcard: boolean;                                 // Whether to show a catch-all wildcard option
  wildcardOption?: RadioOption;                      // What to render the wildcardOption as
  defaultOptions: string[];                          // The default selection of options on page load (in terms of patterns)
  multiple?: boolean;                                // Whether to allow multiple box selections (default: true)
}
```

`RadioOption` has the following schema:

```ts
interface RadioOption {
  label: string;                                     // The visible label for this option
  pattern: string[] | string;                        // What string/strings to include in the filter parameters when this checkbox is clicked
  icon?: string;                                     // Icon class, if you want it rendered (For example, `icon-snowflake`)
  wildcard?: boolean                                 // True only for the wildcard option
}
```

The selected Radio Options are then combined to form a search query.

#### Example

Suppose we had the following options:

```ts
[
  { label: 'Snowflake', pattern: 'snowflake', icon: 'icon-snowflake' },
  { label: 'Data Sources', pattern: ['kafka', 's3'] },
  { label: 'Data Sinks', pattern: ['looker', 'tableau']},
  { label: 'Other', pattern: [], wildcard: true }
]
```

And the `defaultOptions` `['snowflake', 'looker', 'tableau']`. Then on page load the selection of 'Snowflake' and 'Data Sinks' would be made, and the filter value sent off to the search endpoint would be `['snowflake', 'looker', 'tableau']` with the `OR` `filterOperation`.

#### Extra required feature - NOR filter.

If instead 'Snowflake' and 'Other' were selected, then we want to show all results *except* the Data Sources and Data Sinks.
Then the following value would be sent to the search endpoint: `['looker', 'tableau', 'kafka', 's3']` with the new filterOperation `NOR`.

`NOR`'s implementation on the search side is very simple:

```python
# Current OR implementation
if filter.operation == 'OR':
  filter_queries.append(Q(BOOL_QUERY, should=queries_per_term, minimum_should_match=1))
# NEW: NOR filter operation
elif filter.operation == 'NOR':
  filter_queries.append(~Q(BOOL_QUERY, should=queries_per_term, minimum_should_match=1))
```

### DropdownFilterCategory

`DropdownFilterCategory` has the following extra configuration keys:

```ts
interface DropdownFilterCategory extends BaseFilterCategory {
  getOptions: (state: GlobalState) => DropdownOption[]; // Function to determine what options to render
  multiple?: boolean;                                // Whether to allow multiple dropdown selections (default: false)
}
```

`DropdownOption` has the following schema:

```ts
interface DropdownOption {
  label: string;                                     // The visible label for this option - also the unique key
  pattern: string[] | string;                        // What string/strings to include in the filter parameters when this dropdown is selected
  icon?: string;                                     // Icon class, if you want it rendered (For example, `icon-snowflake`)
}
```

The selected Dropdown Options are then combined to form a search query, similar to the RadioFilter.

The dropdown is implemented using the `react-select` library.

## Drawbacks

| Drawback | Comment |
| -------- | ------- |
| Upfront implementation cost | This is small, as this feature has been implemented in our fork (although not up to standard.) |
| Impact on onboarding of Amundsen | Slightly complicates the experience since we're adding more stuff. |
| Complicates the filter definition configuration by allowing dynamically rendered results | Somewhat of a departure from the current filter design in pursuit of a more configurable / streamlined user experience. |

## Alternatives

A similar experience could be achieved by doing some of the following:

* Altering the search bar to allow 'flags' to be added and have this turn into filters ( Search: `my_table tag:<value>` )
* Have tags/databases in search results be clickable to filter down results based on this.

Both of these, while powerful, are less intuitive for users and less controlled by the maintainer, and so they weren't favoured over the new filters.

## Prior art

This radio select feature is present in acryl/datahub:

![Example of search view in Acryl](/assets/000/acryl-filter.png)

Although I am not involved with other data communites and am unaware of the opinions / experiences of them.

## Unresolved questions

Not many unresolved questions since this is already implemented and in use for us, but some questions relating to wider community adoption:

* How much configuration of these filters is preferred vs. unnecessarily complicating the process?
* What other-use cases are available for such filters, and are there changes necessary to make this work?

## Future possibilities

There is the possibility for other, more adventurous, filters:

* Filter by tables upstream/downstream of those fitting the other filters
* Filter by clustering of users (most of my table's users are in compliance, marketing, engineering, etc.)

There is also the possibility to streamline the process of rendering dynamic results as well, since implementing many of the examples above would require more than just configuration (You need to generate the ducks functionality to refresh things like tag values and schemas).

So a feature to make this refresh functionality available via configuration would also be neat.
