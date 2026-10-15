# Changelog

## [0.3.0](https://github.com/Krustytoe/platform-workflows/compare/v0.2.0...v0.3.0) (2026-10-15)


### ⚠ BREAKING CHANGES

* terraform-apply requires an `environment` input. Move `environment:` from the calling job into `with:`.

### Features

* apply the reviewed plan behind an environment gate ([#6](https://github.com/Krustytoe/platform-workflows/issues/6)) ([c821236](https://github.com/Krustytoe/platform-workflows/commit/c82123606a7fc043babc6c2245beddfd3006deae))


### Bug Fixes

* inline composite action steps so releases don't edit workflow files ([#7](https://github.com/Krustytoe/platform-workflows/issues/7)) ([10f7ce8](https://github.com/Krustytoe/platform-workflows/commit/10f7ce854f3fe5b57fafb3e7af250453b739cb35))

## [0.2.0](https://github.com/Krustytoe/platform-workflows/compare/v0.1.0...v0.2.0) (2026-10-06)


### Continuous Integration

* Bump the github-actions group across 4 directories with 7 updates ([#2](https://github.com/Krustytoe/platform-workflows/issues/2)) ([ea07724](https://github.com/Krustytoe/platform-workflows/commit/ea07724045f08968760ca484c42d59511a14d985))
