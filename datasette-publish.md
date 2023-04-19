# Publishing a database to Vercel

First, install both Vercel and the `datasette-publish-vercel` plugin.

<https://vercel.com/docs/cli> has documentation for installing the Vercel CLI.

On macOS:

    brew install vercel-cli

Or use one of these:

    npm i -g vercel

Or:

    pnpm i -g vercel

Now run this command to login:

    vercel login

Install the plugin:

    datasette install datasette-publish-vercel

And deploy the database:

    datasette publish vercel /tmp/peps.db --project python-peps

## Other publishing options

Datasette can publish to the following providers:

- Heroku (`datasette publish cloudrun`)
- Google Cloud Run (`datasette publish heroku`)
- Vercel (with [datasette-publish-vercel](https://datasette.io/plugins/datasette-publish-vercel))
- Fly (with [datasette-publish-fly](https://datasette.io/plugins/datasette-publish-fly))

Further deployment options are described [in the documentation](https://docs.datasette.io/en/stable/deploying.html).