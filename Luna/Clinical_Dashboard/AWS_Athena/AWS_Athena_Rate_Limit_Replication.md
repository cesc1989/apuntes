# AWS Athena Rate Limit Replication
Gem to preven rate limit: https://github.com/kamui/retriable

Trying to replicate this error found in production environment:

    Query execution requested(Change in PSFS Scale) with Execution ID: ce7642c2-34c3-4256-b446-275a3e6f48e7
    
    Requesting query execution(Age Distribution) in Athena
    Rate exceeded
    missing required parameter params[:query_string]
    Query execution requested(Age Distribution) with Execution ID: 
    
    Requesting query execution(Patients Treated) in Athena
    Rate exceeded
    missing required parameter params[:query_string]
    Query execution requested(Patients Treated) with Execution ID: 
    
    Requesting query execution(Body Parts) in Athena

In development is very complicated and haven’t been able to do so. This is what I tried.

## Running Multiple Resque Workers

As in production the application is deployed to several containers, this mean that a background job could run in several containers at once.

In development, I added this line to the `Procfile.dev` file:

    worker: COUNT=5 QUEUE=athena rake environment resque:workers
    
    # Instead of
    worker: QUEUE=athena rake environment resque:work

And I could launch 5 Resque workers. However, if I ran `Resque.enqueue_in(2.seconds, RequestQueryExecutionJob)` it would execute in just one worker.

## Run Job in Several Workers

To have all workers running the same job I wrote this:

    5.times do
      Resque.enqueue_in(2.seconds, RequestQueryExecutionJob)
    end

And all workers picked a job and executed it. The only thing was that seems they do it one at a time.

