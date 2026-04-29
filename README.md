# Factory Hub — connectaiml.com

Static landing page for the seven Factory Hub instances. Hosted on GitHub Pages, served at [connectaiml.com](https://connectaiml.com).

## Files

```
.
├── index.html    Single-file React app (CDN-loaded React 18 + Babel)
├── CNAME         Custom domain marker for GitHub Pages
└── .nojekyll     Prevents Jekyll processing
```

`index.html` is self-contained: no build step, no Node dependencies. Open it locally in any browser to preview, or push to GitHub Pages to deploy.

## Deploy

### One-time setup

1. **Create a GitHub repo.** Public, any name (e.g. `factory-hub` or `connectaiml`). Push these three files to `main`.

2. **Enable Pages.** Repo → Settings → Pages → Source: *Deploy from a branch* → Branch: `main`, folder: `/ (root)` → Save.

3. **Set custom domain.** Same Pages settings page → Custom domain: `connectaiml.com` → Save. GitHub re-reads the `CNAME` file and runs a DNS check; the check will fail until step 4 completes.

4. **Configure DNS at GoDaddy** (see DNS section below).

5. **Enforce HTTPS.** Once the DNS check passes (can take 10–60 minutes), tick *Enforce HTTPS* in Pages settings.

### Iterating

Edit `index.html` locally, commit, push to `main`. Pages rebuilds within ~30 seconds. No CI/CD needed for the simple path.

## DNS at GoDaddy

Sign in → My Products → Domain → DNS. Two record sets:

### Apex (`connectaiml.com`)

Four `A` records, all with name `@`, all pointing to GitHub's Pages IPs:

| Type | Name | Value             | TTL    |
|------|------|-------------------|--------|
| A    | @    | 185.199.108.153   | 1 hour |
| A    | @    | 185.199.109.153   | 1 hour |
| A    | @    | 185.199.110.153   | 1 hour |
| A    | @    | 185.199.111.153   | 1 hour |

Delete any existing `A` records on `@` first (GoDaddy parking page placeholders).

### `www` subdomain (optional but recommended)

One `CNAME` record pointing to your GitHub user/org site:

| Type  | Name | Value                       | TTL    |
|-------|------|-----------------------------|--------|
| CNAME | www  | `<your-github-username>.github.io` | 1 hour |

Replace `<your-github-username>` with the account that owns the repo. The trailing `.github.io` is required.

GitHub will then serve `connectaiml.com` and redirect `www.connectaiml.com` → `connectaiml.com` automatically.

### Verifying

Once propagated:

```bash
dig connectaiml.com +short          # should return the four GitHub IPs
dig www.connectaiml.com +short      # should return your-username.github.io then the IPs
```

The Pages settings page shows a green check next to the custom domain when DNS resolves correctly.

## Costs

- GitHub Pages: **free** (public repo)
- HTTPS certificate: **free** (Let's Encrypt, auto-renewed by GitHub)
- Custom domain: whatever GoDaddy charges you, no extra fees from GitHub

## When to upgrade to a real build

The current `index.html` loads React and Babel from a CDN and compiles JSX in the browser on every page load. That's ~3 MB of dependencies before the page renders. Acceptable for a portfolio hub; not ideal for SEO or first-paint metrics.

When the content stabilises, graduate to a Vite build:

```
npm create vite@latest factory-hub -- --template react
# Move the JSX into src/, set up a GitHub Actions workflow that
# runs `npm run build` and deploys `dist/` to the gh-pages branch.
```

The deployed bundle drops to ~150 KB minified, first paint becomes near-instant, and the dev loop gets HMR. Worth doing once the page stops changing weekly.
