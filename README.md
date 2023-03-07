# blog.squirrelcave.com

Static site for blog.squirrelcave.com

To regenerate the static site, run `pnpm run generate`. This will blow away the existing static-site folder, and rerun gssm with preconfigured options.

# infra notes

- Private ghost instance on docker on tailscale
- Static site generation via [Fried-Chicken/ghost-static-site-generator](https://github.com/Fried-Chicken/ghost-static-site-generator)
- automatically republish static site via github actions
- hosted on cloudfare pages

Private ghost instance-on-docker-on-tailscale is documented in code and version-controlled (though not publicly published yet).

Static site generation is [documented in code](https://github.com/NAR8789/squirrelcave.com-static-site/blob/main/package.json#L4) and version controlled in this repo.

Github action is [documented in code](https://github.com/NAR8789/squirrelcave.com-static-site/blob/main/.github/workflows/publish-static-site.yml) and version controlled in this repo. However, to trigger this action I need to transform ghost's webhook into a [repository dispatch](https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event), and I do that via nginx.

Automatic via cloudfare pages is currently manually configured.

## Event transformation via nginx

Ghost can webhook out on `site-updated` to a static url.
This will be a GET request.

Triggering github actions requires POSTing to the repository dispatch endpoint.

To join the two, we use nginx as a request transformer.

```nginx
location /to-github/site-changed {
  set $ghtoken $arg_ghtoken;
  set $args '';

  # github repository dispatch documentation: https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event
  proxy_pass https://api.github.com/repos/NAR8789/squirrelcave.com/dispatches;
  proxy_method POST;

  proxy_set_header Accept 'application/vnd.github+json';
  proxy_set_header Authorization 'Bearer $ghtoken';
  proxy_set_header X-Github-Api-Version '2022-11-28';
  proxy_set_body '{"event_type":"site-changed"}';
}
```

1. Now ghost can webhooks to nginx, passing along authentication token in a `ghtoken` parameter.
2. nginx transforms this into POST with bearer authentication and the correct body

## Cloudflare pages

I think this is free (at least for now).

Currently manually configured. Can we get this configuration into code?

- squirrelcave-blog cloudflare static site pointing at NAR8789/blog.squirrelcave.com github repo
  - no build command
  - output under `/static`
  - everything else default
- custom domain: squirrelcave.com

# security notes

## ghost webhook to nginx to github

ghost webhooks just support a static url, so I include my github access token as a query parameter.
Not sure how well ghost secures the storage for this.

This token needs to be refreshed every 90 days.

## tailscale

Since the ghost instance is self-hosted and only accessible through tailscale, the github action needs to be on the tailnet.

Conveniently, the tailscale team have built a [Connect Tailscale](https://github.com/marketplace/actions/connect-tailscale) action. This does require a [reusable authkey](https://tailscale.com/kb/1085/auth-keys/), but github actions' secrets store seems reasonably trustworthy.

The auth key is a little inconvenient, as it needs to be manually refreshed every 90 days.

# TODO

- [ ] move cloudflare pages configuration to code
