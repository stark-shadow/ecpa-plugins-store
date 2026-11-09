# ECPA Plugins Store

The curated plugin registry for ECPA / EasyCLIProxyAPI. It follows the same
format and release contract as
[`router-for-me/CLIProxyAPI-Plugins-Store`](https://github.com/router-for-me/CLIProxyAPI-Plugins-Store),
so any entry can move between the two stores without rewriting.

This repository maintains **only `registry.json`**. Plugin binaries, checksums
and release notes stay in each plugin author's own GitHub repository.

## Maintenance Policy

**This registry is maintained by hand. It is not a mirror of anyone else's
store.**

`omc-community/omc-plugins` publishes its own combined registry; treat it as a
source of facts about *their* entries (name, description, locales, tags, latest
version), never as a file to copy over this one. A wholesale copy silently drops
every entry that upstream does not carry — that is exactly how `aistudio2api`
and `freetier` disappeared on 2026-11-09 (`aea2464`).

When syncing an upstream entry:

1. Compare per `id`, never replace the `plugins` array.
2. Update `version` only when the latest release tag is confirmed and the
   release carries `checksums.txt` plus platform zips.
3. Leave every entry upstream does not know about untouched.

`eps-push --from <dir>` mirrors a whole directory with `rsync --delete`, which
has the same blast radius. Prefer plain `eps-push "<message>"` unless you
really intend to replace the tree.

## Consuming the registry

Add the raw URL as an extra plugin store source:

```yaml
plugins:
  enabled: true
  store-sources:
    - https://raw.githubusercontent.com/stark-shadow/ecpa-plugins-store/main/registry.json
```

The built-in official source is always included; this one is appended to it.
The client derives the source ID from the URL, so entries can be installed with
one click from the Plugin Store tab and from the ECPA Featured tab.

## Registry Format

```json
{
  "schema_version": 2,
  "plugins": [
    {
      "id": "sample-provider",
      "name": "Sample Provider",
      "description": "Adds sample provider support.",
      "author": "author-name",
      "repository": "https://github.com/author-name/cliproxy-sample-provider",
      "logo": "https://raw.githubusercontent.com/author-name/cliproxy-sample-provider/main/logo.png",
      "homepage": "https://github.com/author-name/cliproxy-sample-provider",
      "license": "MIT",
      "tags": ["Provider"]
    }
  ]
}
```

Required fields:

- `id`
- `name`
- `description`
- `author`
- `repository`

Optional fields:

- `version` (legacy; only a display fallback when the latest release cannot be queried)
- `logo`
- `homepage`
- `license`
- `tags`
- `locales` (`{ "zh-CN": { "description": "..." } }`)

Do not add `install`, `artifacts`, `source_id` or download URLs. Installation
metadata is derived from the plugin's own GitHub releases; an entry carries no
download information.

## Validation Rules

- `schema_version` is `2`. A `github-release` entry may state `"install": {"type": "github-release"}` (schema v1 rejects that shape, so v2 is required once any entry carries it).
- `id` must start with an ASCII letter or digit, then use only ASCII letters,
  digits, `.`, `_` or `-`, up to 128 characters total.
- `id` values must be unique.
- `version`, when present, must not start with `v`.
- `repository` must be exactly `https://github.com/{owner}/{repo}` (no `.git`
  suffix, query or fragment).

## Release Requirements

CLIProxyAPI reads the repository's **latest release**. The tag defines the
plugin version and must be `v<version>` with a dotted numeric version.

Each release must include one `checksums.txt` and one zip per supported platform,
named after the version without the leading `v`:

```text
<id>_<version>_<goos>_<goarch>.zip
checksums.txt
```

Example:

```text
aistudio2api_1.1.5_linux_arm64.zip
checksums.txt
```

`checksums.txt` uses sha256sum format:

```text
<sha256>  aistudio2api_1.1.5_linux_arm64.zip
```

## Zip Layout

Each platform zip must contain the target dynamic library at the zip root:

- Darwin: `<id>.dylib`
- Linux: `<id>.so`
- Windows: `<id>.dll`

Nested libraries, absolute paths, zip-slip paths, mismatched filenames and
multiple dynamic libraries are rejected by the installer.

A missing `<id>_<version>_<goos>_<goarch>.zip` or `checksums.txt` for the user's
platform makes one-click install fail with `release asset ... not found`. Check
both before listing a plugin here:

```bash
gh release view --repo <owner>/<repo> --json tagName -q .tagName
gh release view --repo <owner>/<repo> --json assets -q '.assets[].name'
python3 -c "import zipfile,sys;print(zipfile.ZipFile(sys.argv[1]).namelist())" <archive>.zip
```

## Adding A Plugin

Edit only `registry.json`, then add the entry in this order and with these facts:

- `version` is the newest released tag without the leading `v`.
- `logo` uses the author's real avatar id, never a guessed number:
  `gh api repos/<owner>/<repo> --jq .owner.id`.
- `locales.zh-CN.description` carries the Chinese one-liner.

Preserve existing entries and edit the file as JSON so quoting stays valid:

```bash
python3 - <<'PY'
import json, collections
p='registry.json'
d=json.load(open(p), object_pairs_hook=collections.OrderedDict)
d['plugins']=[e for e in d['plugins'] if e.get('id')!='<id>']+[entry]
open(p,'w').write(json.dumps(d,ensure_ascii=False,indent=2)+'\n')
PY
```

## Publishing

This checkout lives only on the Raspberry Pi push host:

```bash
cd ~/ecpa-plugins-store
eps-push "feat: add <plugin>"
```

Never pass `--from` with a directory that holds only part of the registry.

Then prove the client URL serves the entry rather than trusting the push:

```bash
sleep 4; curl -s https://raw.githubusercontent.com/stark-shadow/ecpa-plugins-store/main/registry.json \
  | python3 -c "import json,sys;[print(p['id'],p['version']) for p in json.load(sys.stdin)['plugins']]"
```

A private repository returns **404 for a file that exists** on unauthenticated
raw requests, so keep this repository public.
