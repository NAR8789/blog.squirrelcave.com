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

Github action is [documented in code](https://github.com/NAR8789/squirrelcave.com-static-site/blob/main/.github/workflows/publish-static-site.yml) and version controlled in this repo. However, to trigger this action I need to transform ghost's webhook into a [repository dispatch](https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event), and I do that via pipedream, which is currently manually configured.

Automatic via cloudfare pages is currently manually configured.

## Event transformation via pipedream

1. ghost webhooks to pipedream
2. pipedream filters and authenticates events (authentication is just a string match on key, so this is probably vulnerable to timing attacks??)
3. pipedream triggers a repositority dispatch event to start the publish flow in github actions.

Pipedream's security and namespacing are pretty janky unfortunately
- secrets live in a global ENVIRONMENT
- no built-in authentication helpers or time-invariant filter action. I could self-implement, but then how much complexity is pipedream really saving me?

Pipedream so far only really saves me the complexity of self-hosting a node app.
- no built-in ghost integration, so my filtering is custom-rolled
- because pipedream is "no-code", custom-rolling means form-based-programming-and-custom-tooling
- no built-in trigger github action step, so I need to self-roll that
- pipedream's built-in github integration requires full admin access to github, which... no.

## Cloudflare pages

I think this is free (at least for now).

Currently manually configured. Can we get this configuration into code?

- squirrelcave-blog cloudflare static site pointing at NAR8789/blog.squirrelcave.com github repo
  - no build command
  - output under `/static`
  - everything else default
- custom domain: squirrelcave.com

# security notes

## pipedream

- ghost webhooks just support a static url, so I secure the pipedream endpoint with a static key embedded in the url
- pipedream also needs a github access token to trigger the repository dispatch
- TODO: replace pipedream with a self-hosted application that connects ghost webhooks to github, for one less
  compromisable third-party service.
- TODO: make the application a github app, so I can restrict it to just the squirrelcave.com-static-site repo.
  - is there a lighter-weight way to do this...?

# TODO

- [ ] replace pipedream with a self-hosted application that connects ghost webhooks to github
  - [ ] make this a github app so I can restrict it to just the squirrelcave.com-static-site repo.
- [ ] move cloudflare pages configuration and github actions configuration to code
