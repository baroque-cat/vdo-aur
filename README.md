# vdo AUR package

[![Check for updates](https://github.com/$GITHUB_REPOSITORY/actions/workflows/check-update.yml/badge.svg)](https://github.com/$GITHUB_REPOSITORY/actions/workflows/check-update.yml)
[![Build](https://github.com/$GITHUB_REPOSITORY/actions/workflows/build-test.yml/badge.svg)](https://github.com/$GITHUB_REPOSITORY/actions/workflows/build-test.yml)

Maintenance repository for the [vdo](https://github.com/dm-vdo/vdo) AUR package — userspace tools for managing Virtual Data Optimizer (VDO) volumes.

## Workflow

### Automated update detection

Every Monday, a scheduled GitHub Action runs [`nvchecker`](https://github.com/lilydjwg/nvchecker) against the upstream `dm-vdo/vdo` repository. If a new release is found, it:

1. Bumps `pkgver` and resets `pkgrel` to `1`
2. Recalculates `sha256sums` (downloads the new tarball)
3. Regenerates `.SRCINFO`
4. Opens a **Pull Request** with the changes

No automatic merging — every update requires human review.

### CI checks on PR

When a PR touches `PKGBUILD` or `.SRCINFO`, the build pipeline:

1. Builds the package in a clean Arch Linux container
2. Runs [`namcap`](https://wiki.archlinux.org/title/Namcap) for packaging correctness
3. Uploads the built `.pkg.tar.zst` and namcap report as artifacts

### Release checklist (before merging a PR)

- [ ] CI build is green
- [ ] `namcap` output reviewed — no real errors
- [ ] Upstream [release notes](https://github.com/dm-vdo/vdo/releases) checked for breaking changes
- [ ] `depends` and `optdepends` still match the new version's requirements
- [ ] Package installs and works correctly (test if possible)

### Pushing to AUR

After merging the PR, push the updated files to the AUR remote:

```bash
# One-time setup: add the AUR remote
git remote add aur ssh://aur@aur.archlinux.org/vdo.git

# Push after merge
git fetch aur
git rebase aur/master
git push aur main:master
```

> **Note:** The AUR only accepts the `master` branch. Always keep your local branch name (`main`) distinct from the AUR remote tracking.

## Local development

### Check for new versions manually

```bash
nvchecker --file .nvchecker.toml
```

### Update checksums after changing source

```bash
updpkgsums
```

### Build and install locally

```bash
makepkg -si
```

### Regenerate .SRCINFO

```bash
makepkg --printsrcinfo > .SRCINFO
```

## Files

| File | Purpose |
|------|---------|
| `PKGBUILD` | Package build recipe |
| `.SRCINFO` | Machine-readable metadata (required by AUR) |
| `.nvchecker.toml` | nvchecker config tracking `dm-vdo/vdo` releases |
| `.github/workflows/check-update.yml` | Automated update detection + PR creation |
| `.github/workflows/build-test.yml` | CI build + namcap validation |

## License

GPL-2.0-or-later (matching upstream)
