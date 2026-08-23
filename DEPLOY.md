# Deploying to Streamlit Community Cloud

Order matters: the organisation must exist **before** you connect Streamlit, or
the OAuth prompt will not offer organisation access and you will have to
disconnect and start again.

## 1. Create the repository

In the `celnicconsulting` organisation, create a **public** repository named
`kainga_ora_housing_nz`. Do not initialise it with a README — this build already
has one.

Then, from this folder:

```bash
git remote add origin https://github.com/celnicconsulting/kainga_ora_housing_nz.git
git push -u origin main
```

The repository is already initialised and committed, so `git init` is not needed.

The data file is 18 MB, comfortably under GitHub's 50 MB warning threshold, so
Git LFS is not required.

## 2. Deploy

1. Go to https://share.streamlit.io and sign in with GitHub
2. At the OAuth prompt, grant **organisation access** to `celnicconsulting`
3. Click **Create app**, then choose the existing repository
4. Set:
   - Repository: `celnicconsulting/kainga_ora_housing_nz`
   - Branch: `main`
   - Main file path: `app/kainga_ora_housing_nz.py`
   - **Custom subdomain**: `celnic-housing-nz`

   Streamlit subdomains allow lowercase letters, digits and hyphens only — no
   underscores — so the URL uses hyphens even though the repository name uses
   underscores. That gives `celnic-housing-nz.streamlit.app`.
5. Deploy

The first build takes a few minutes while dependencies install.

## 3. After deploying

Put the live URL in `README.md` where it says _add your Streamlit URL here_.

## Resource envelope

Community Cloud allows up to 2.7 GB memory, 2 CPU cores and 50 GB storage. This
app opens an 18 MB DuckDB file read-only and caches query results, so it sits
well inside those limits. The heaviest query is the H3 map at resolution 12,
which aggregates 72,951 rows into 72,144 hexagons — under a second, and cached
after the first call.

## Updating the data later

The pipeline lives in the parent project, not in this repository. To refresh:

```bash
python scripts/run_all.py
```

Then copy the rebuilt extract and app into this folder and push:

```bash
cp public/kainga_ora_housing_public.duckdb public_repo/data/
cp app/kainga_ora_housing_nz.py public_repo/app/
git commit -am "Refresh data extract" && git push
```

Community Cloud redeploys automatically on push to the tracked branch.

**Before refreshing, check the Kāinga Ora cookie.** Their site is behind Imperva
bot protection; `scripts/.ko_cookie` needs a current session cookie or step 01
falls back to cached copies of the pages. The refresh procedure is printed by
`nz_http.refresh_ko_cookie_instructions()`.
