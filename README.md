# ORM DO - Unlimited SQLite DBs Directly In Your Worker

Functionality

- 🔥 Abstracts away from the DO so you can just perform SQL queries to state from unlimited SQLite DBs, directly from your workers.
- 🔥 Compatible and linked with @outerbase to easily explore the state of the DO or DOs
- 🔥 query fn promises json/ok directly from the worker. This makes working with it a lot simpler.

> [!IMPORTANT]
> New (more opinionated) version at [janwilmake/dorm](https://github.com/janwilmake/dorm)

# Demo

See https://ormdo.wilmake.com for the `example.ts` example, which demonstrates it works using a users management API and HTML for that.

X Post: https://x.com/janwilmake/status/1912146275597721959

# Contribute

(still testing this button! lmk if it worked)

[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/janwilmake/orm-do)

# Usage

In your `wrangler.toml`

```toml
[[durable_objects.bindings]]
name = "MY_EXAMPLE_DO"
class_name = "ORMDO"

[[migrations]]
tag = "v1"
new_sqlite_classes = ["ORMDO"]
```

In your worker:

```ts
import { ORMDO, createDBClient, DBConfig, DBClient } from "./queryState";
import { adminHtml } from "./adminHtml";
export { ORMDO };

const dbConfig: DBConfig = {
  /** Put your CREATE TABLE queries here */
  schema: [
    `
    CREATE TABLE IF NOT EXISTS users (
      id TEXT PRIMARY KEY,
      name TEXT NOT NULL,
      email TEXT UNIQUE,
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )
    `,
  ],
  /** Updating this if you have breaking schema changes. */
  version: "v1",
  // Optional: used for authenticating requests
  authSecret: "my-secret-key",
};

type Env = {
  MY_EXAMPLE_DO: DurableObjectNamespace;
};

export default {
  fetch: async (request: Request, env: Env, ctx: any) => {
    const client = createDBClient(env.MY_EXAMPLE_DO, dbConfig);

    // First try to handle the request with the middleware
    const middlewareResponse = await client.middleware(request, {
      prefix: "/api/db",
      secret: dbConfig.authSecret,
    });

    // If middleware handled the request, return its response
    if (middlewareResponse) {
      return middlewareResponse;
    }

    // Get URL and method for routing
    const url = new URL(request.url);
    const method = request.method;

    ///... YOUR ENDPOINTS HERE USING DB CLIENT
  },
};
```

# Why?

I'm looking for a simpler way to create stateful workers with multiple DBs. One of the issues I have with DOs is that they're hard to work with and your code becomes verbose quite easily. Also it's not yet easy to explore multiple databases. This is an abstraction that ensures you can perform state queries directly from your worker, queue, schedule, etc, more easily.

My ultimate goal would be to be able to hook it up to github oauth and possibly [sponsorflare](https://sponsorflare.com) and have anyone explore their own data.

I'm still experimenting. Hit me up if you've got ideas!

Made by [janwilmake](https://x.com/janwilmake).
