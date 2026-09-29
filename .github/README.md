# pw-module-jumplinks

Castus copy of [Jumplinks](https://gitlab.com/rockettpw/seo/jumplinks-one) by Mike Rockett, a ProcessWire module for managing permanent and temporary redirects.

Upstream is on **GitLab**, so this isn't a GitHub "fork". It's a GitHub repo whose `source` branch carries the GitLab history.

> This README lives in `.github/` so that upstream's own [`README.md`](../README.md) stays untouched and never conflicts when we merge upstream changes.

## What this is

- **Used by:** [castusdesign/intertrain](https://github.com/castusdesign/intertrain), as the redirect manager (admin: Setup > Jumplinks). Redirects are stored in the database, not in this repo.
- **Composer package:** `rockett/jumplinks` (type `processwire-module`).
  - We **keep upstream's package name** on purpose. That way Dependabot in Intertrain matches any advisory published for `rockett/jumplinks` on Packagist.
  - Intertrain lists this repo as a `vcs` repository, which takes precedence over the Packagist copy of the same name.
- **Installs to:** `public_html/site/modules/ProcessJumplinks/` (set by `extra.installer-name`)
- **Pulls in:** [`league/csv`](https://packagist.org/packages/league/csv) `^7.2` from Packagist, for the CSV import.

## Branches

| Branch | Contains | Rule |
|---|---|---|
| `source` | Upstream GitLab history, exactly as published | **Never commit our changes here.** It only ever fast-forwards to an upstream tag. |
| `main` (default) | `source` + our `composer.json` changes, `.gitattributes`, this README, and our patches | Every change is its own commit, listed below |

## Current base

- **Upstream:** https://gitlab.com/rockettpw/seo/jumplinks-one
- **Version:** tag `1.5.64`, commit `683572f`
- **Our release tag:** `1.5.64-patch1`

We **don't push upstream's own tags** to this repo. Upstream's tagged `composer.json` uses the `pw-module` type and a different installer, so Composer would treat those tags as installable versions of `rockett/jumplinks`. Only our `-patchN` tags should exist here.

## Our patches

| Commit on `main` | What it changes | Why |
|---|---|---|
| "Use league/csv from Composer instead of the vendored copy" | Deletes `Classes/LeagueCsv/` (league/csv 7.2.0) and the `require_once` of its autoloader in the CSV import path of `ProcessJumplinks.module.php` | Puts league/csv under Composer, where Dependabot can see it. The vendored copy also shipped league/csv's own dev tooling manifests, which Dependabot kept flagging. Originally applied in intertrain `869c500`. |
| "Package for Composer" | `composer.json`: type `pw-module` becomes `processwire-module`; `wireframe-framework/processwire-composer-installer` is replaced by `composer/installers`; adds `league/csv ^7.2` and `extra.installer-name` | Intertrain installs every module with `composer/installers`. Mixing in the wireframe installer would need a second installer plugin. |

Keep league/csv on `^7.2`: version 9 changed the `Reader` API that the import code uses (`Reader::createFromString()`, `setDelimiter()`, `setEnclosure()`).

**Can they be dropped?**

- **The league/csv patch:** only if upstream switches to requiring league/csv through Composer.
- **The composer.json changes:** only if Intertrain switches to the wireframe installer.

## How to update

1. **One-time setup** in your clone:

   ```sh
   git remote add upstream https://gitlab.com/rockettpw/seo/jumplinks-one.git
   ```

2. **Fetch upstream and move `source` to the new release:**

   ```sh
   git fetch upstream --tags
   git log --oneline source..1.5.65      # review what's new
   git switch source
   git merge --ff-only 1.5.65            # the new upstream tag
   git push origin source                # push the branch only, not upstream's tags
   ```

3. **Merge into `main`:**

   ```sh
   git switch main
   git merge source
   ```

   - **`composer.json` conflicts:** keep our version. Copy across any genuinely new upstream requirement, such as a new PHP minimum.
   - **Upstream reintroduced `Classes/LeagueCsv/` or the `require_once`:** remove them again.

4. **Tag and push only our tag:**

   ```sh
   git tag 1.5.65-patch1
   git push origin main 1.5.65-patch1
   ```

   Don't use `git push --tags`, because it would publish upstream's tags too (see "Current base").

## Rolling it out in Intertrain

In the Intertrain repo:

1. Run `docker compose exec app composer update rockett/jumplinks`.
2. Commit the changed `composer.lock`.
3. In the admin, go to Modules > Refresh. Then open Setup > Jumplinks and check that existing redirects are listed.
4. Smoke-test:
   - A known redirect still redirects.
   - Importing a small CSV of redirects works.
5. Open a PR. Deploying is done by the team as usual.

## Licence

ISC, as upstream (see [`LICENSE.md`](../LICENSE.md)). Our patches are released under the same licence.
