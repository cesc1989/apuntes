# AWS Athena Integration Design

Relevant Links

- [Query data from S3 files using Amazon Athena](https://towardsdatascience.com/query-data-from-s3-files-using-aws-athena-686a5b28e943)
- [AWS SDK Athena Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/Athena/Client.html)
- [Athena Presto SQL Functions](https://docs.aws.amazon.com/athena/latest/ug/presto-functions.html)

To get the needed information from Athena we need to do multiple steps. The steps are:

- Get work group name
- Get named queries for that work group
    - Find the required named query for the chart
- Get the query string for the selected named query
- Request execution of query in Athena
    - It might take some time to finish
- Get results of the query execution
    - This can turn into an asynchrony process. In an initial background-like step we request Athena to execute the queries. In the final step, we read the results using the `execution_id` provided by Athena.
- Process the result from the query execution
    - Needed because the result from Athena API has its own structure or it’s a CSV file in S3

As we have the need to do some synchronous and asynchronous steps to get the data from Athena, the design around this integration aims to have independent classes that cover each or many steps in isolation.

## Named Query Execution Request in Athena

The first step to get the data for all charts is to request Athena to run the queries in a background job or a rake task(run by a cron task). Ideally, they’d run every hour or so.

**Services - Requester**
To ask Athena to execute a query we’ll use services suffixed *Requester*. So, the class `CaseDistributionRequesterService` will talk to Athena for the execution we need.

After finding the named query and starting the execution of it, the service will be save the `execution_id` as a record in the `ExecutionId` model.

**Query Finder**
In `app/lib/athena_named_query_finder` we place classes in charge of finding the right named query for a given chart.

So the class `AthenaNamedQueryFinder::CaseDistribution` will find the query for the **Case Distribution chart**.

## Fetch and Process Results from Athena Query

The second step is to now collect the results from the query, process them, convert them to an understandable JSON structure for the frontend integration to work with.

**Controllers**
The front face of all this. They kind of orchestrate everything. The controller/endpoint has the important task of using the service to get the results, call the converter to make them a JSON object and then send it back to the frontend client to fill in the chart.

There should be one controller per chart to keep all code small and well spread.

**Services - Fetcher**
In the first step the services request execution of queries. In this step they fetch the results of that run query. The service `CaseDistributionChartService` does this for the **Case Distribution chart**.

They’ll read the last record in the `ExecutionId` table to get the data for the chart. Also could use the related records to have kind of an history of the chart.

**Converters**
These guys located in the folder `app/lib/athena_results_converter` have the task of making sense out of the results of the Athena query by filtering and applying expected formulas on the dataset. They will return a JSON representation that fits the data required for the chart.

A class like `AthenaResultsConverter::CaseDistributionConverter` will process the results from Athena for the **Case Distribution chart**.

# Errors Found

## Not authorized to perform

    is not authorized to perform: athena:ListNamedQueries on resource: arn:aws:athena:us-east-1:811996165463:workgroup/primary)

Can be fixed by giving permissions on API key to perform actions on Athena service. 

## InvalidRequestException

    Aws::Athena::Errors::InvalidRequestException (No output location provided. An output location is required either through the Workgroup result configuration setting or as an API input.)

In the Work Group settings in Athena, set the **Results location** and enable **Override client-side settings**.

## Query has not finished error

    Aws::Athena::Errors::InvalidRequestException (Query has not yet finished. Current state: QUEUED)
    
    Query has not yet finished. Current state: RUNNING
    false

Message is pretty clear. If one request results from a running query, this happens.

## No implicit conversion of Seahorse error

    no implicit conversion of Seahorse::Client::Response into Array

When trying to concat a response rows in a loop.

Solution: I was trying to `concat` an Athena response with a `response.result_set.rows` from a previous response.

# Permissions Needed

- ListWorkGroups
- ListNamedQueries
- GetNamedQuery
- StartQueryExecution
- GetQueryExecution
- GetQueryResults
    - S3 → GetObject

## Work Groups - Relevant Stuff

See

- [SDK Docs](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/Athena/Client.html#list_work_groups-instance_method) → method `list_work_groups`.
- [AWS API Docs](https://docs.aws.amazon.com/athena/latest/APIReference/API_ListWorkGroups.html) → endpoint `ListWorkGroups`.

```ruby
Class:0x0000562eb3e84e90>:Aws::Athena::Types::ListWorkGroupsOutput:0x562eb7237468
        next_token = nil,
        work_groups = [
            [0] #<#<Class:0x0000562eb3eab040>:Aws::Athena::Types::WorkGroupSummary:0x562eb72373f0
                creation_time = 2020-02-12 21:39:34 +0000,
                description = "",
                name = "primary",
                state = "ENABLED"
            >
        ]
    >
```

## Named Queries - Relevant Stuff

> Provides a list of available query IDs only for queries saved in the specified workgroup. Requires that you have access to the workgroup.

Returns IDs alone. There’s no way to distinguish what ID belongs to a given named query.

See

- [Docs](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/Athena/Client.html#list_named_queries-instance_method) → method `list_named_queries`.
- [AWS API Docs](https://docs.aws.amazon.com/athena/latest/APIReference/API_ListNamedQueries.html) → `ListNamedQueries`.

```ruby
Class:0x0000562eb3e86e98:Aws::Athena::Types::ListNamedQueriesOutput:0x562eb71b64a8
	named_query_ids = [
		[0] "ce077fbe-c044-4336-b336-c88ffbf7b064"
	],
	next_token = nil
```

## Named Query - Relevant Stuff

> Returns information about a single query. Requires that you have access to the work group in which the query was saved

Here we have the query to be run against Athena.

See

- [Docs](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/Athena/Client.html#get_named_query-instance_method) → method `get_named_query`.
- [AWS API](https://docs.aws.amazon.com/athena/latest/APIReference/API_GetNamedQuery.html) → endpoint `GetNamedQuery`.

```ruby
<Class:0x0000562eb3e80868>:Aws::Athena::Types::GetNamedQueryOutput:0x562eb7196ce8
        named_query = #<#<Class:0x0000562eb3e84990>:Aws::Athena::Types::NamedQuery:0x562eb7196c98
            database = "pfdatabase",
            description = "primera query",
            name = "first-query",
            named_query_id = "ce077fbe-c044-4336-b336-c88ffbf7b064",
            query_string = "select * from pfdata;",
            work_group = "primary"
        >
    >
```

Extract the info from the named_query instance of `Aws::Athena::Types::NamedQuery`.

## Start Query Execution - Relevant Stuff

```ruby
Class:0x0000562eb3e9fc90>:Aws::Athena::Types::StartQueryExecutionOutput:0x562eb6cf5748
        query_execution_id = "c9eaebe2-80fc-444d-a361-bf6eaa9db9f7"
    >
```

## Get Query Execution - Relevant Stuff

```ruby
Class:0x0000562eb3e80138>:Aws::Athena::Types::GetQueryExecutionOutput:0x562eb6b964d8
        query_execution = #<#<Class:0x0000562eb21e2da0>:Aws::Athena::Types::QueryExecution:0x562eb6b96460
            query = "select * from pfdata",
            query_execution_context = #<#<Class:0x0000562eb3e8fa70>:Aws::Athena::Types::QueryExecutionContext:0x562eb6b963c0
                database = nil
            >,
            query_execution_id = "c9eaebe2-80fc-444d-a361-bf6eaa9db9f7",
            result_configuration = #<#<Class:0x0000562eb3e8dce8>:Aws::Athena::Types::ResultConfiguration:0x562eb6b961e0
                encryption_configuration = nil,
                output_location = "s3://pfdataset/results/c9eaebe2-80fc-444d-a361-bf6eaa9db9f7.csv"
            >,
            statement_type = "DML",
            statistics = #<#<Class:0x0000562eb3e8f700>:Aws::Athena::Types::QueryExecutionStatistics:0x562eb6b96168
                data_manifest_location = nil,
                data_scanned_in_bytes = 0,
                engine_execution_time_in_millis = 868,
                query_planning_time_in_millis = 858,
                query_queue_time_in_millis = 61389,
                service_processing_time_in_millis = 35,
                total_execution_time_in_millis = 62292
            >,
            status = #<#<Class:0x0000562eb3e8ea08>:Aws::Athena::Types::QueryExecutionStatus:0x562eb6b960f0
                completion_date_time = 2020-02-14 15:26:58 +0000,
                state = "FAILED",
                state_change_reason = "SYNTAX_ERROR: line 1:15: Table awsdatacatalog.default.pfdata does not exist",
                submission_date_time = 2020-02-14 15:25:56 +0000
            >,
            work_group = "primary"
        >
    >
```

The **state** of the query is in `resp.query_execution.status.state` (one of "QUEUED", "RUNNING", "SUCCEEDED", "FAILED", "CANCELLED")

## Get Query Results - Relevant Stuff

> To stream query results successfully, the IAM principal with permission to call `GetQueryResults` also must have permissions to the Amazon S3 `GetObject` action for the Athena query results location.

