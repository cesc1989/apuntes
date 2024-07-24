# Webpacker not reading from process.env in Production

Relacionado [[Detalle_con_Webpacker_y_el_ENV]]

The Physician Portal application is a Rails 6 monolith app. We use [Webpacker gem](https://github.com/rails/webpacker) for JavaScript assets bundling and be able to have React and all modern frontend goodies in the same project.

For our first staging release, we thought we’d need an environment variable holding the URL for the internal API. In development, that was achieved setting the value in an `.env` file.

    # .env
    API="http://localhost:3000/"

For staging environment, we asked DevOps guy to setup an env var named `API` and we were supposed to read it from the system using `process.env` in the config service.

    // app/javascript/congig/index.ts
    export let configService = new ConfigService({
      API: process.env.API
    });

But turns out that it doesn’t work. API value remains undefined. So, in order to proceed, what we did was unignoring the `.env` file and follow [steps suggested in the Webpacker](https://github.com/rails/webpacker/blob/master/docs/env.md) documentation to be able to read that file in a production-like environment (staging).

And it worked. We pushed the `.env` file to staging and lived happily with 🙂 

Until a production release came in and the hardcoded API value in the `.env` file wouldn’t worked because it pointed to the staging URL.

These are all the steps I tried to help Webpacker read the `API` env var when compiling(`rake assets:precompile`) so that `process.env` could be able to read it.

1. Ignored `.env` and removed it from the project with `git rm` `--``cached .env`.
2. Removed `dotenv` package configuration from `config/webpack/environment.js` file.
3. Used `Webpacker::Compiler.env` to [add env vars](https://github.com/rails/webpacker/blob/5-x-stable/docs/env.md) when compiling in production.
4. Compiled assets in production-mode in my dev environment. [Seen here](https://github.com/rails/webpacker/issues/855#issuecomment-331807289).
5. `console.log`ged values set in `config/initializers/webpacker.rb`. They showed up.
6. Sent to production. Nothing worked. `API` still `undefined`.
7. Logged in JavaScript console `console.log(process.env.NODE_ENV)`. It shows up. Still needed env vars not showing.

In the end, I noticed that having an `API` env var wouldn’t be necessary as the API and the frontend client are in the same folder path and requests can be done just appending the URL path without specifying the HTTP schema and domain.

    # API and frontend in different apps/servers
    API="https://api.example.com/v1"
    Frontend request -> fetch(`${API}/comments`)
    
    # API and frontend in same app/server
    # API="https://api.example.com/v1" is not necessary
    Frontend request -> fetch("/comments") 

I applied the change:

    // app/javascript/config/config.service.ts
    export class ConfigService {
      //...
      get(key: string) {
        // return `${this.envConfig[key]}/v1`;
        return 'v1'
      }
    }

And it worked in staging environment but still I don’t know why Webpacker is not catching the env vars from system when compiling.

## Main Suspicion

I think this isn’t working because Webpacker (Webpack) need an `.env` file to exist to read from when COMPILING the frontend application.

I think `process.env` can read from system environment variables but only when doing a compilation not in the JavaScript interpreter.

## Related Issues

When looking for answers, these two issues got my attention:

- [Pass custom environment variables to the compiler](https://github.com/rails/webpacker/issues/691)
    - [Related PR](https://github.com/rails/webpacker/pull/694)
    - [Comment that mentions](https://github.com/rails/webpacker/issues/691) this wouldn’t help `process.env` read ENV vars. The conversation goes the issue below.
- [Environment variables set with Webpacker::Compiler aren't available via process.env](https://github.com/rails/webpacker/issues/1641)
    - [Related PR](https://github.com/rails/webpacker/pull/1658)

From the PR description I can infer that using `Webpacker::Compiler.env['API'] = ENV['API']` is helpful when using the binstubs in the project  `bin/` folder **BUT NOT** when compiling using `rake assets:precompile`.

That’s why I was able to see the variables(`console.log`) when compiled the assets in production-mode(`NODE_ENV=production ./bin/webpack --watch --colors --progres`) in my development environment. Also explains why I can’t see them in production environment.

