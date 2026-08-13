# SDI Product Development Playbook — STAGING

This is the **testing environment**, not the real app. It's a byte-for-byte
copy of the production Playbook (`App - GitHub Hosted/`) with exactly one
line changed — see below — so new features can be built and broken here
without any way of reaching real content.

**If you're looking for the live app your team actually uses, this isn't
it.** That's in `App - GitHub Hosted/`, hosted at
`https://kburkhart2026.github.io/sdi-product-playbook/`.

## Why this exists, and the one line that makes it safe

Production and staging are connected to **completely separate GitHub
repos** — different app repo, different private content repo. The one
thing that actually enforces that separation is this line near the top of
`index.html`'s `<script>` block:

```js
const GITHUB_CONTENT_REPO = 'sdi-product-playbook-staging-content';
```

Production's copy of this same file has `'sdi-product-playbook-content'`
instead — a different repo entirely. Because your GitHub tokens are each
scoped to *one specific repo* (not just "whatever this app happens to be
pointed at"), a staging token literally cannot read or write the production
content repo, and vice versa, even by accident. That's the real safety
boundary here — not carefulness, an actual GitHub-enforced limit.

## One-time GitHub setup (same steps as production, a second time)

1. Create two more repos on the same GitHub account:
   - `sdi-product-playbook-staging` (public) — this folder becomes that repo.
   - `sdi-product-playbook-staging-content` (private) — staging's content.
2. On `sdi-product-playbook-staging`: Settings → Pages → Source: `main`
   branch, root folder. You'll get a URL like
   `https://kburkhart2026.github.io/sdi-product-playbook-staging/`.
3. Push this folder to that repo:
   ```bash
   cd "App - GitHub Hosted (Staging)"
   git remote add origin https://github.com/kburkhart2026/sdi-product-playbook-staging.git
   git branch -M main
   git push -u origin main
   ```
4. Generate two **new** fine-grained PATs, scoped to *only*
   `sdi-product-playbook-staging-content` — same process as production, but
   don't reuse a production token here, and don't reuse a staging token on
   production later. Editor: Contents Read and write. Viewer: Contents
   Read-only.

## One-time content migration

Same flow as production, pointed at staging's own repo:
1. Open the staging Pages URL.
2. **Export ▾ → Restore from backup** → load a copy of your real backup
   file. (Confirmed: staging should start from a real copy of your data,
   not the generic demo seed — this matters most for testing anything
   performance-related, since a tiny fake dataset won't tell you much.)
3. **⚙ Data & Backup → Connect to GitHub…** → paste the staging editor PAT.
4. **⚙ Data & Backup → Push local content to GitHub** → seeds the staging
   content repo.

## The actual workflow: sync → build → test → promote

This is the part that matters every time you want to try something new,
not just once:

1. **Sync**: before starting a new round of feature work, copy production's
   *current* `index.html` over this folder's copy, then re-apply just the
   one `GITHUB_CONTENT_REPO` line change above. This keeps staging honest —
   it always starts from what's actually live, not some older snapshot that's
   quietly drifted from production.
2. **Build**: make the change here, in this copy, the same way every feature
   in this project has been built — edit, syntax-check, test.
3. **Test**: push to *this* repo, try it against the staging Pages URL with
   staging's own tokens. Break things freely. Nothing here can reach real
   content.
4. **Promote**: once you're happy with it, apply the same change to
   production's real `index.html` (in `App - GitHub Hosted/`, not here),
   then commit and push *that* repo as usual. Production is never touched
   as a side effect of anything that happens in this folder — promoting is
   always its own deliberate, separate step.

This is a good place to try the bigger ideas that are too risky to test
directly against production — e.g. making the content repo public, or
splitting one big content file into several smaller ones, both from the
load-speed discussion. Try them here first.

## Never put tokens in this file, either

Same rule as production's README: don't paste PATs into this file or any
file in this folder — it's public, same as production's app repo. Paste
tokens directly into the app's own prompts, never into a document.
