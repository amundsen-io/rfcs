- Feature Name: table_data_quality_checks
- Start Date: 2021-07-16
- RFC PR: [amundsen-io/rfcs#39](https://github.com/amundsen-io/rfcs/pull/41)
# Table Checks Frontend Component

## Summary
This RFC proposes a simple frontend component to display data quality checks on the table details page.
The APIs and models that back this component will be stubbed. This proposal is not a full-fledged integration with 
data quality checks.

## Terms
- Data Quality Check - Like unit tests for code, data quality checks are assertions that try to guarantee certain 
  characteristics of data. For example a data quality check could be as simple as asserting that a column is not null,
  or implement some complex custom logic.


## Motivation
While Amundsen is not currently building unit tests for data, we can empower our users by allowing integrations 
with a generic external data quality service. Exposing the status of data quality checks in Amundsen can give users 
an at-a-glance summary of the table's quality. This can help a user to quickly decide if they want to use this table.

While we would like to eventually incorporate Data Quality Checks as a first-class  

## Product Details
Users will see a new section on the left panel of the Table Details page titled `Data Quality Checks`. There will be
a message and icon for the current status of these checks, if they exist. A passing table might say 
`✅ 10 of 10 checks passed` and a failing table might say `❌ 3 of 10 checks failed`. Additionally, the user may see 
the last run timestamp, and a link to the external data quality service, if available.

## UX Explanation
A new section will be added to the left side of the Table Details Page.

![Data Quality Preview](../assets/039/data_quality.png)

## Technical Details
Table Data Quality Checks will be implemented as an optional feature on Amundsen Frontend. This feature will be hidden 
behind a config flag and will use a generic client much like the existing data preview client. The frontend will define
a placeholder API `/api/quality/v0/table` but will not implement it.
 
The frontend will expect a response that includes the following:
```  
{
  num_checks_total: Int;
  num_checks_success: Int;
  num_checks_failed: Int;
  external_url: Optional[Str];
  last_run_timestamp: Optional[Int];
}
```

## Drawbacks
Granted that we are trying to support a generic model, it's difficult to support a deeper integration given that we 
are not endorsing a data quality service provider.

## Alternatives
There are several alternatives: 
  - Ingest data quality checks as part of the databuilder and display the status as part of the table metadata API. 
The displayed status in this case may be stale.
  - We could build data quality checks as an Amundsen service and provide a much deeper level of service, but this 
is not really Amundsen's primary purpose.
