# fdox-squirrel-n4o-collection

The **collection repository** that publishes the [FDOx
Registry](https://github.com/FDOx-squirrel/fdo-squirrel-registry)'s bundle to
the **NFDI4Objects Knowledge Graph**, via
[n4o-rse/n4o-kg-profile](https://github.com/n4o-rse/n4o-kg-profile).

```
fdo-squirrel-registry          produces the bundle, knows nothing about the KG
    dist/fdo-registry-n4o.ttl  DCAT + FDOx + CIDOC CRM, SHACL-gated (S4/S5)
        │  raw.githubusercontent.com, fetched on every build
        ▼
fdox-squirrel-n4o-collection   this repository
    metadata.yaml              the only file maintained by hand
    .github/workflows/build.yml
        │  uses: n4o-rse/n4o-kg-profile/.github/workflows/collection.yml@v1
        ▼
dist/n4o-collection.ttl        the registration record N4O reads
dist/metadata.ttl              DCAT + VoID statistics + CRM alignment + queries
docs/                          landing page, browsable SPARQL page
```

Nothing here is bespoke to this repository beyond `metadata.yaml`: the
profile, the SHACL gate and both page generators are versioned in
`n4o-kg-profile` and copied in on every run, so this repository cannot
quietly drift from the shapes every other collection is checked against.

## What happens manually, outside this repository

- **Publishing to Zenodo.** This repository, and `n4o-kg-profile`, only build
  and validate; nobody here uploads anything. Once the graph is right, the
  result is published by hand.
- **Registering with NFDI4Objects.** VZG, who operate the KG, enter the
  collection into `n4o-collections.json` by hand once it exists and
  validates. The `id` field in `metadata.yaml` is a placeholder until they
  assign the real `https://graph.nfdi4objects.net/collection/<n>` URI.

## Current status (2026-09-04)

The workflow is **not yet enabled on push**. `n4o-rse/n4o-kg-profile` is
patched directly (see `n4o-kg-profile-patch.zip` and its `PATCH-README.md`)
rather than reported as an issue, since that org belongs to the same person
maintaining this one. Once that patch is committed and tagged `v1`:

1. `git ls-remote --tags https://github.com/n4o-rse/n4o-kg-profile.git`
   should show `v1`.
2. Add `push: {branches: [main]}` back to `.github/workflows/build.yml`
   here — the only change needed, the `uses:` line already matches.
3. Create this repository on GitHub (see **Setting up the repository**
   below) and push this tree to it.
4. *Settings → Pages → Source: GitHub Actions*.
5. Push (or run *Actions → Build → Run workflow* by hand first).

`metadata.yaml`'s `sameAs` now carries the FDOx Registry's real Wikidata item
(Q141277996). `id` and `homepage` still point at the source repository as a
placeholder until the registry has its own DOI (`fdo-squirrel-registry`
PRIMER.md, S8/S9) and VZG assigns a collection number.

## Setting up the repository

Not created yet. When you do:

| Setting | Value |
|---|---|
| Owner | `FDOx-squirrel` (same org as `fdo-squirrel-registry`, `fdo-squirrel`) |
| Name | `fdox-squirrel-n4o-collection` |
| Description | "NFDI4Objects Knowledge Graph collection for the FDOx Registry" |
| Visibility | Public — GitHub Pages and the `raw.githubusercontent.com` fetch both need it |
| Default branch | `main` |
| Topics | `nfdi4objects`, `fair-digital-object`, `linked-open-data`, `dcat`, `cidoc-crm`, `knowledge-graph` |
| Initialise with | nothing (README/LICENSE/gitignore) — this tree already has them; an auto-created README would just collide on the first push |
| Pages | *Settings → Pages → Source: GitHub Actions*, only after the first successful workflow run — the option has nothing to point at before then |

Push this tree as the initial commit (`git init`, `git remote add origin
git@github.com:FDOx-squirrel/fdox-squirrel-n4o-collection.git`, `git add -A`,
one commit, `git push -u origin main`) — the usual `primer-repo` skeleton
(`main.py`, `PRIMER.md`) is deliberately **not** part of it; `n4o-kg-profile`
copies `profile/` and `build/` in on every run and expects exactly one
hand-maintained file (`fdo-squirrel-registry`'s own `PRIMER.md`, A4,
"Ort und Form des Collection-Repos").

## Running it by hand

Not possible yet either, for the same reason `build.yml` cannot run: `python
build/make_metadata.py` needs `profile/` and `build/` copied in from
`n4o-kg-profile`, and there is no tagged release to copy from. Once one
exists:

```
pip install -r requirements.txt   # from n4o-kg-profile
robocopy \path\to\n4o-kg-profile\profile profile /E
robocopy \path\to\n4o-kg-profile\build build /E
python build\make_metadata.py
python build\build_sparql.py
rmdir /S /Q profile
rmdir /S /Q build
```

## Licence

Code (this repository's own files): MIT. The published bundle inherits its
licence from `fdo-squirrel-registry` (CC BY 4.0).
