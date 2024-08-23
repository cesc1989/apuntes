# Create Athena Named Queries via AWS CLI
Links:

- [create-named-query](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/athena/create-named-query.html)
- [list-named-queries](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/athena/list-named-queries.html)
- [get-named-query](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/athena/get-named-query.html)
- [batch-get-named-query](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/athena/batch-get-named-query.html)
- [update-named-query](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/athena/update-named-query.html)


## Get NamedQueryIds

This command return all named queries ids but without names:

    $ aws athena list-named-queries --profile alpha
    {
        "NamedQueryIds": [
            "9360d50f-3415-4b8e-a251-43f7654d3bc2",
            "5257fbec-f521-4af7-a962-20d89ccf9b80",
            "aa42a5ea-4397-442b-82dd-98c2f1ed3658",
            "b5c03fc6-8d31-4486-bf99-d66ae7b6603f",
        ]
    }


## JSON Skeleton
    $ aws athena create-named-query --generate-cli-skeleton --profile beta
    {
        "Name": "",
        "Description": "",
        "Database": "application-data",
        "QueryString": "",
        "ClientRequestToken": "",
        "WorkGroup": "primary"
    }

About `ClientRequestToken`:

> This token is listed as not required because Amazon Web Services SDKs (for example the Amazon Web Services SDK for Java) auto-generate the token for users. If you are not using the Amazon Web Services SDK or the Amazon Web Services CLI, you must provide this token or the action will fail.


## Queries to Create


