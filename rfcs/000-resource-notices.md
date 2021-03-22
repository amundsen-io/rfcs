- Feature Name: Resource Notices
- Start Date: 2021-03-22
- RFC PR: [amundsen-io/rfcs#0000](https://github.com/amundsen-io/rfcs/pull/0000) (after opening the RFC PR, update this with a link to it and update the file name)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Resource Notices

## Summary

Resource Notices allows Amundsen administrators to show informational, warning, and alert messages related to the different resources (tables, dashboards, people) we expose in Amundsen.

## Motivation

This feature will help administrators show messages related to deprecation, updates (or lack of), and informational messages related to specific resources. In Lyft's case, we use them to point final users to improved versions of current surfaced tables, so final users employ more trusted data.

## Guide-level Explanation (aka Product Details)

An Amundsen final user can see a notice in the table/dashboard/user detail page whenever their system admin adds one related to it.

A notice is a small box with an icon and a message containing HTML markup (like links and bolded text).

These will come in three flavors:

- Informational: Marked with a blue "i" icon on the right side
- Warning: Marked with an orange exclamation mark icon on the right side
- Alert: Marked with a red exclamation mark icon on the right side

For example, if company X wants to deprecate the use of one table, they can opt to add a new notice in their configuration file:

```
    ... Table Configuration
    notices: {
        "<SCHEMANAME>.<TABLENAME>": {
          severity: NoticeSeverity.ALERT,
          message: `<span>This table is deprecated, please use <a href="<LINKTONEWTABLEDETAILPAGE>">this new table</a> instead.</span>`,
        },
    },
```

This code will show a notice with a red exclamation icon whenever a final user visits the table's Table Detail page.

This feature's ultimate goal is to allow Amundsen administrators to point their users to more trusted/higher quality data without removing the old references.

## Reference-level Explanation (aka Technical Details)

The design we propose is based on the following:

- Using our current frontend configuration files (config-utils.ts, config-default.ts, config-custom.ts, and config-types.ts)
- Allowing a three-level "severity" schema for the alerts (info, warning, and alert)
- Allowing its use in "tables" and "dashboards" (we could add it to people as well, but we'll need designs for it)

What we are not supporting for now:

- Multiple notifications per resource
- Loading of external files (like a JSON)
- Application of a single notice to multiple resources

In the implementation, we'll check the notices object whenever we visit the resource detail page, showing any notification if present.

It is not a breaking change, there won't be any costs for older versions to support this.

## Drawbacks

- This implementation could not be scalable, as maybe users want to have too many notifications. It doesn't mean this work won't be a good start, but it could be not enough. TBD
- With this approach, to add/update/remove a notification, admins would need to make a new deployment of the application. A better approach would be if we use a UI, so a proper feature flag system will be much better for this.
- Notices are on a 1:1 basis. If admins wanted to deprecate a whole schema or dashboard product, they would need to create a notice for each table/dashboard.

## Alternatives

- Integrating a proper feature flag system and feed the configuration from it. This would be ideal not just for this feature but for many others. It is also more complex and costly.
- Using a JSON file to feed the notices was also considered. Given that we are not happy with the current configuration options (JS config + Python configuration), adding yet another one didn't feel like a good move.
- If we don't do anything, we won't be able to show this information in a highlighted way (we would probably add this as a programmatic description, making it less visible)

## Prior art

Nothing that we know of

## Unresolved questions

We think this proposal is a good starting point. It might be lacking in different ways, but we don't know yet in which ways. That's why we think this would be only the first iteration.

## Future possibilities

Out of scope and pending feedback for this stage will be:

- A different way of storing the notices (JSON file, as part of the graph, UI fed)
- Multiple notifications at once and how we would limit those
- Notifications in the User pages
