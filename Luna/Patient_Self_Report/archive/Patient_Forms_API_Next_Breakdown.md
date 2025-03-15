# Patient Forms API: Next Breakdown

Pre items:
- ~~Refactor: change numbers in JSON response status code to status code Rails symbols~~
- ~~Send patient signature to an S3 bucket~~
- Send patient agreed terms (PDF) + signature to S3 bucket

Main items:
1. Authentication: POSTPONED.
2. ~~Upgrade to Rails 6~~
3. ~~Webhooks~~
4. ~~Dashboard: Done in separate project.~~

## Authentication

- Define if authentication will only be for mobile app or also for web app
    - Making it only required for mobile will let web form work without changing so many things
- Define items(user id, secret key base, ?) to generate JWT
    - JWT is more secure and doesn’t need to be saved to a database column
- Implement token verification step in controllers
- Update test suite to run with token in requests

## ~~Upgrade to Rails 6~~

- ~~Research how to upgrade from Rails 5 to Rails 6~~
- ~~Review and/or increase test coverage~~
- ~~List gems that might break when upgrading~~
- ~~Upgrade to Rails 6~~

## Webhooks

- ~~Define list of webhooks: Form is completed.~~
- Make Webhooks retriable in case of failures
- What is exactly a Webhook?

[What are Webhooks? by Zapier](https://zapier.com/blog/what-are-webhooks/)
> There are two ways your apps can communicate with each other to share information: polling and webhooks. As one of our customer champion's friends has explained it: **Polling is like knocking on your friend’s door and asking if they have any sugar. Webhooks are like someone tossing a bag of sugar at your house whenever they buy some.**

[What’s a Webhook?](https://sendgrid.com/blog/whats-webhook/)
> Webhooks are sometimes referred to as “Reverse APIs,” as they give you what amounts to an API spec, and you must design an API for the webhook to use. The webhook will make an HTTP request to your app (typically a POST), and you will then be charged with interpreting it.


- Resources:
    - [REST Hooks](https://resthooks.org/)
    - [Ngrok](https://ngrok.com/)
    - [Webhook.site](https://webhook.site/)

