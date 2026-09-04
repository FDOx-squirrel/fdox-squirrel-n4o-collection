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

**Live and working end-to-end.** Build #4 succeeded (`collection` and `pages`
both green, 36s) after two upstream fixes to `n4o-rse/n4o-kg-profile` and one
correction here:

- the checkout-org bug (`n4o-kg-profile-patch.zip`),
- a self-checkout that resolved to the wrong ref inside a composite action
  (`n4o-kg-profile-patch-2.zip`, `github.action_path` instead of a second
  `actions/checkout` keyed on `github.action_ref`),
- **GitHub Pages must be set to *Settings → Pages → Source: GitHub Actions*
  *before* the first run that tries to deploy it**, not after — the first
  attempt (Build #3) failed its `pages` job for exactly that reason. The
  order given in an earlier revision of this README had it backwards.

Live:

- Collection page: <https://fdox-squirrel.github.io/fdox-squirrel-n4o-collection/>
- Query page (SPARQL in the browser, via Pyodide): <https://fdox-squirrel.github.io/fdox-squirrel-n4o-collection/sparql.html>
- `push` is now enabled (`.github/workflows/build.yml`) — every push to
  `main` rebuilds and redeploys.

`metadata.yaml`'s `sameAs` carries the FDOx Registry's real Wikidata item
(Q141277996). `id` and `homepage` still point at the source repository as a
placeholder until the registry has its own DOI (`fdo-squirrel-registry`
PRIMER.md, S8/S9) and VZG assigns a collection number — updating either is a
one-line edit to `metadata.yaml`, nothing structural.

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
