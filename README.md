# blog.squirrelcave.com

Static site for blog.squirrelcave.com

To regenerate the static site, run `pnpm run generate`. This will blow away the existing static-site folder, and rerun gssm with preconfigured options.

# infra notes

- Private ghost instance on docker on tailscale
- Static site generation via [Fried-Chicken/ghost-static-site-generator](https://github.com/Fried-Chicken/ghost-static-site-generator)
- TODO: automatically republish static site via github actions
- hosted on cloudfare pages

Private ghost instance-on-docker-on-tailscale is documented in code and version-controlled (though not publicly published yet).

Static site generation is documented in code and version controlled in this repo.

Automatic republishing via github actions and cloudfare pages are currently manually configured.

## Automatic republishing via github actions

TODO: set up github actions to
- respond to `site.changed` webhook
- run `pnpm run generate`
- automatically commit the result
- bob's your uncle; once master is updated, cloudflare pages will automatically take it from there to published

## Cloudflare pages

I think this is free (at least for now).

Currently manually configured. Can we get this configuration into code?

- squirrelcave-blog cloudflare static site pointing at NAR8789/blog.squirrelcave.com github repo
  - no build command
  - output under `/static`
  - everything else default
- custom domain: currently preview.squirrelcave.com (to be changed to blog.squirrelcave.com)

# TODO

- [ ] swap domains: move private ghost instance to write.squirrelcave.com or somesuch, and host the static site on blog.squirrelcave.com.
- [ ] set up github actions to automatically publish static site upon ghost `site.changed` webhook.
- [ ] move cloudflare pages configuration and github actions configuration to code
