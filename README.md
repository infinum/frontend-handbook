![](https://infinum.com/handbook/dist/assets/images/books/frontend/header.svg?59a036660281a9152d1e8329f23182fe)

# Frontend handbook

Our handbook based on 10 years of experience in Frontend/JS development. We try to document our way of working and writing code here. As this might change feel free to watch this space regularly.

## Environments

There are currently two environments:

### Staging

This is used for testing purposes to make sure everything is in order before merging a PR. It is hosted at: https://beta.infinum.com/handbook/frontend

To access it you will need a ZG Office VPN setup (contact your TL for help). More information about setting up the VPN can be found [in the general handbook](https://infinum.com/handbook/general/initial-setup/setting-up-vpn/setting-up-vpn).

In order to deploy your work to the staging server, merge your branch to the `staging` branch and it will be automatically deployed. You can do a local `git merge`, followed by a `git push`.

### Production

Production handbook is hosted at https://infinum.com/handbook/frontend.

Once your PR has been approved and you've tested it on the _Staging_ server, you can merge your PR to the `master` branch and it will automatically be deployed to production.

## Contributing

We welcome all contributions from both the public and Infinum members. Feel free to open up the pull request with your ideas/articles/chapters, or what ever you want to see merged.

If you want to add a new chapter, you can do it in [frontend-handbook-private](https://github.com/infinum/frontend-handbook-private) repository `chapters.yml` file.

## Discussions

If you want to discuss something, or propose a more general change feel free to use [Github Discussions](https://github.com/infinum/frontend-handbook/discussions) for this.

## Credits

Infinum Frontend Handbook is maintained and sponsored by [Infinum](https://www.infinum.com).

<p align="center">
  <a href='https://infinum.com'>
    <picture>
        <source srcset="https://assets.infinum.com/brand/logo/static/white.svg" media="(prefers-color-scheme: dark)">
        <img src="https://assets.infinum.com/brand/logo/static/default.svg">
    </picture>
  </a>
</p>

## Licensing

All rights reserved.
