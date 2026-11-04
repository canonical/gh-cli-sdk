# GitHub CLI SDK for Workshop

This SDK provides the official GitHub CLI (`gh`) for working with pull
requests, issues, releases and other GitHub resources from the terminal. A
GitHub token can be supplied through the `github-token` secret plug, and
`gh`'s configuration and login state are persisted on the host across
workshop updates.

---

## Reference workshop

A minimal workshop:

```yaml
# workshop.yaml
name: gh-demo
base: ubuntu@24.04
sdks:
  - name: gh-cli
    channel: latest/stable
```

This demonstrates a workshop with `gh` available on the `PATH`.

---

## Using the SDK

### Prerequisites, project layout

1. No other SDKs are required.
2. `gh` works inside any project directory; it picks up the repository from
   the project's git remotes.
3. Launch and refresh don't change your git configuration. To let `git` use
   `gh` for HTTPS authentication, run `gh auth setup-git` inside the
   workshop.

### Authenticate

Use the `github-token` secret plug, so the token is never stored in the
workshop. It reads the login that `gh auth login` keeps in your host keyring
(`gh auth status` shows `(keyring)`).

Add a `secret` slot to the `system` SDK in your workshop definition:

```yaml
# workshop.yaml
name: gh-demo
base: ubuntu@24.04
sdks:
  - name: system
    slots:
      github-token:
        interface: secret
        attributes:
          service: "gh:github.com"
          username: ""
  - name: gh-cli
    channel: latest/stable
```

Then, after `workshop launch`, connect the plug:

```bash
workshop connect gh-demo/gh-cli:github-token gh-demo/system:github-token
```

- `username: ""` selects the account `gh` is currently using, so the same
  definition works for everyone on the project. Without it the slot matches
  several keyring items and fails with `multiple secrets match`.
- For a narrower token, store a
  [fine-grained personal access token](https://github.com/settings/personal-access-tokens)
  with `secret-tool store --label="GitHub token" --collection=default service github account gh-cli`
  and use `service: github` and `account: gh-cli` as the slot attributes.
- The connection survives workshop updates unless you change the slot's
  attributes. A `GH_TOKEN` or `GITHUB_TOKEN` you pass in (for example with
  `workshop exec --env GH_TOKEN`) takes precedence over the plug.

#### Fallback: log in inside the workshop

Without the plug, `gh` uses its own login, which is kept across workshop
updates:

```bash
workshop exec -- gh auth login
```

### Use GitHub from the workshop

```bash
workshop exec -- gh pr list
workshop exec -- gh issue create
```

---

## Plugs (resources this SDK consumes)

### `gh-config`

- Interface: `mount`
- Workshop target: `/home/workshop/.config/gh` (mode `0700`, since it can
  hold a login token)
- Purpose: Persists `gh` configuration and login state between workshop
  updates.
- Connection: automatic.

### `gh-data`

- Interface: `mount`
- Workshop target: `/home/workshop/.local/share/gh`
- Purpose: Persists installed `gh` extensions between workshop updates.
- Connection: automatic.

### `github-token`

- Interface: `secret`
- Purpose: Supplies a GitHub token to `gh` as `GH_TOKEN` on each
  invocation. If the plug is not connected, `gh` falls back to its own
  login state.
- Connection: manual; see [Authenticate](#authenticate) for the keyring item,
  the `system` slot and the `workshop connect` command. The
  [Workshop secrets documentation](https://ubuntu.com/workshop/docs/how-to/develop-with-workshops/provide-secrets)
  covers the slot definition in full.
- Exposure: while connected, any process in the workshop can request the
  token. The token is also visible to `gh` and every process `gh` starts.

## Slots (resources this SDK provides)

This SDK doesn't define any slots.

---

## Documentation and guidance

- [GitHub CLI manual](https://cli.github.com/manual/)

---

## Community and support

- [GitHub CLI discussions](https://github.com/cli/cli/discussions)
- Please review our [Code of Conduct](https://ubuntu.com/community/ethos/code-of-conduct)
  before participating.

---

## Contributions

All contributions, including code, documentation updates, and issue reports,
are welcome!

- Open issues or pull requests on the [official repository](https://github.com/canonical/gh-cli-sdk).

---

## License and copyright

Copyright 2026 Canonical Ltd.

This program is free software: you can redistribute it and/or modify it under
the terms of the
[GNU Lesser General Public License version 2.1 (LGPLv2.1)](https://www.gnu.org/licenses/old-licenses/lgpl-2.1.html)
as published by the Free Software Foundation.

This program is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE. See the GNU Lesser General Public License for more details.

The [GitHub CLI](https://github.com/cli/cli) is licensed under the
[MIT License](https://github.com/cli/cli/blob/trunk/LICENSE).
