# Publishing and maintaining releases

Repository: https://github.com/thiagolcordeiro/home-assistant-wiz-segments

Publish only this project, with README, LICENSE, hacs.json and
custom_components/wiz_segments at the repository root. Exclude firmware, backups,
local test records and the enclosing WLED workspace.

1. Update the manifest version and `CHANGELOG.md`.
2. Run the tests and `python tools/validate_release.py`.
3. Push a commit to `main` and inspect GitHub Actions results.
4. Open **Releases → Draft a new release**, creating a matching tag such as
   `v0.2.1` against the verified commit.
5. Describe changes, evidence and limitations. Mark experimental versions as
   prereleases. Publish the release; a tag alone is not a GitHub release.

GitHub supplies source archives. HACS uses the normal repository layout; do not
enable `zip_release`. Prereleases may require enabling beta versions in HACS.
Without a release, custom repository installation uses the default branch content.
Test installation in a real Home Assistant instance before declaring stability.

Set a repository description and topics, and keep Issues enabled. HACS validation
also checks remote metadata. Neither HACS nor hassfest replaces runtime testing.

Users can add this as a custom repository using the [README](../README.md).
Default catalog inclusion is a separate [HACS publication process](https://www.hacs.xyz/docs/publish/).

For subsequent local updates, run inside this project:

```text
git add custom_components tests tools docs .github README.md README.pt-BR.md HARDWARE.md CONTRIBUTING.md CHANGELOG.md LICENSE hacs.json .gitignore .gitattributes
git diff --cached --stat
git commit -m "Describe the change"
git push origin main
```

Review staged files before committing. Never include credentials.
