---

title: Environments
weight: 3

---

## Usage

The most common use case for environments is deploying to a staging subdomain before your production environment. `wrangler publish` will look at your top level configuration, and you can specify other environments beneath it. Each of these environments will inherit the values from the top level configuration if they are not specified, with the following caveats.

- `type` will always be inherited from the top-level configuration; you cannot specify different types for different environments.
- Fields that can be inherited from the top level are `account_id`, `zone_id`, `workers_dev`, and `webpack_config`. `kv_namespaces` and `route` must be defined for each environment and will not be inherited.
- `name` is inherited. If left out of the environment configuration, a Worker project named `my-worker` with an environment `[env.dev]` would become `my-worker-dev`.

### Naming 

You cannot specify multiple environments with the same name. If this were allowed, publishing each environment would overwrite your previously deployed worker, and the behavior would not be clear. For this reason wrangler appends the environment name to the top-level name to publish a Worker script.

### Examples

#### Top level configuration

##### Routes

This `wrangler.toml` has no environments defined and will publish `my-worker` to `example.com/*`

```toml
type = "webpack"
name = "my-worker"
account_id = "12345678901234567890"
zone_id = "09876543210987654321"
route = "example.com/*"
```

```console
$ wrangler publish
✨  Built successfully, built project size is 523 bytes.
✨  Successfully published your script to example.com/*
```

##### workers.dev

This `wrangler.toml` has no environments defined and will publish `my-worker` to `my-worker.<your-subdomain>.workers.dev`

```toml
type = "webpack"
name = "my-worker"
account_id = "12345678901234567890"
workers_dev = true # this field specifies that the worker should be deployed to workers.dev
```

```console
$ wrangler publish
✨  Built successfully, built project size is 523 bytes.
✨  Successfully published your script to https://my-worker.<your-subdomain>.workers.dev
```

In order to use environments with this configuration, you can pass the name of the environment via the `--env` flag.

With this configuration, Wrangler will behave in the following manner:

```console
$ wrangler publish
✨  Built successfully, built project size is 523 bytes.
✨  Successfully published your script to dev.example.com/*
```

```console
$ wrangler publish --env staging
✨  Built successfully, built project size is 523 bytes.
✨  Successfully published your script to staging.example.com/*
```

```console
$ wrangler publish --env production
✨  Built successfully, built project size is 523 bytes.
✨  Successfully published your script to example.com/*
```

#### Staging Environment with workers.dev

In order to deploy your code to workers.dev, you must include `workers_dev = true` in the desired environment. Your `wrangler.toml` may look like this:

```toml
name = "my-worker"
type = "webpack"
account_id = "12345678901234567890"
zone_id = "09876543210987654321"
route = "example.com/*"

[env.staging]
workers_dev = true
```

With this configuration, Wrangler will behave in the following manner:

```console
$ wrangler publish
✨  Built successfully, built project size is 523 bytes.
✨  Successfully published your script to example.com/*
```

```console
$ wrangler publish --env staging
✨  Built successfully, built project size is 523 bytes.
✨  Successfully published your script to https://my-worker-staging.<your-subdomain>.workers.dev
```

## Concepts






Here is an example `wrangler.toml` to illustrate



#### orkers.dev as a first class target

If you only want to deploy to workers.dev you can configure Wrangler like so:

```toml
name = "my-worker-dev"
type = "webpack"
account_id = "12345678901234567890"
workers_dev = true

[env.production]
name = "my-worker"

[env.staging]
name = "my-worker-staging"
```

With this configuration, Wrangler will behave in the following manner:

```console
$ wrangler publish
✨  Built successfully, built project size is 523 bytes.
✨  Successfully published your script to https://my-worker-dev.<your-subdomain>.workers.dev
```

```console
$ wrangler publish --env staging
✨  Built successfully, built project size is 523 bytes.
✨  Successfully published your script to https://my-worker-staging.<your-subdomain>.workers.dev
```

```console
$ wrangler publish --env production
✨  Built successfully, built project size is 523 bytes.
✨  Successfully published your script to https://my-worker.<your-subdomain>.workers.dev
```

### Same name for multiple environments



```toml
name = "my-worker"
type = "webpack"
account_id = "12345678901234567890"
zone_id = "09876543210987654321"
route = "example.com/*"

[env.staging]
name = "my-worker"
workers_dev = true
```

```console
$ wrangler publish
Error: ⚠️  Each name in your `wrangler.toml` must be unique, this name is duplicated: my-worker
```

```console
$ wrangler publish --env staging
Error: ⚠️  Each name in your `wrangler.toml` must be unique, this name is duplicated: my-worker
```

### Defining workers_dev and route

```toml
name = "my-worker"
type = "webpack"
account_id = "12345678901234567890"
zone_id = "09876543210987654321"
route = "example.com/*"
workers_dev = true

[env.staging]
workers_dev = true
route = "staging.example.com/*"
```

Wrangler will fail to publish to an environment where `route` is defined alongside `workers_dev = true`.

```console
$ wrangler publish
Error: ⚠️  Your environment should only include `workers_dev` or `route`. If you are trying to publish to workers.dev, remove `route` from your wrangler.toml, if you are trying to publish to your own domain, remove `workers_dev`.
```

```console
$ wrangler publish --env staging
Error: ⚠️  Your environment should only include `workers_dev` or `route`. If you are trying to publish to workers.dev, remove `route` from your wrangler.toml, if you are trying to publish to your own domain, remove `workers_dev`.
```

## Invalid configurations

### Multiple types

You cannot specify a type for each environment, type must be specified at the top level of your `wrangler.toml`.

```toml
name = "my-worker"
type = "webpack"
account_id = "12345678901234567890"
zone_id = "09876543210987654321"
route = "example.com/*"
workers_dev = true

[env.staging]
type = "rust"
```

Wrangler will not error with this configuration, it will build with the `webpack` type.
