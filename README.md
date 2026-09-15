# Persona — Sileo Repository

Sileo/Cydia repository for [Persona](https://staging.0x17.dev/) — device identity spoofing for jailbroken iOS (rootless).

**Source URL:** `https://staging.0x17.dev/`

Add it in Sileo: *Sources → Add*, paste the URL, reload, install **Persona**.

## Layout

- `Packages`, `Packages.gz` — repository index (regenerated on every publish)
- `*.deb` — package builds (latest per package name)
- `index.html` — landing page
- `depictions/` — package detail pages + terms (app design system, EN/VI)

Package builds are published from the private Persona project via `Scripts/publish.sh`; depiction pages ship alongside releases.
