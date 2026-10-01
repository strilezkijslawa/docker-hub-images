# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Dockerfiles for PHP-FPM Alpine images published to Docker Hub under the `strilezkijslawa/` namespace. There is no application code, build system, or test suite. Each top-level directory is one image's build context.

## Image layering

Every PHP version (8.2, 8.3, 8.4, 8.5) has three images that build on top of each other via Docker Hub tags, not local paths:

```
php:8.X-fpm-alpine
  └─ strilezkijslawa/php8.X-alpine        (dir: strilezkijslawa-php8X-alpine-{latest|stable})
       └─ strilezkijslawa/php8.X-ldap-alpine   (dir: ...-alpine-ldap-...)   adds ldap ext
            └─ strilezkijslawa/php8.X-oci-alpine    (dir: ...-alpine-oci-...)    adds Oracle Instant Client + oci8
```

- The ldap and oci Dockerfiles `FROM` the **published** `:latest` tag of the parent image. Changes to a base image only reach derived images after the base is built, tagged, and pushed (or tagged locally with the exact same name) — build in order base → ldap → oci.
- Only the base image directories contain `php.ini` and `imagemagick/policy.xml`; derived images inherit them.
- The directory suffix (`-stable` for 8.4, `-latest` for 8.2/8.3/8.5) is part of the directory name only; the `FROM` lines always reference `:latest`.

## Per-version differences

Directories for different PHP versions are near-identical copies. The intended differences are only:
- the `FROM php:8.X-fpm-alpine` line in the base image,
- the parent tag in ldap/oci images,
- `ARG OCI8_VERSION` in the oci image (8.2 → 3.2.1, 8.3 → 3.3.0, 8.4 and 8.5 → 3.4.1; oci8 versions are tied to PHP versions).
- `date.timezone` in `php.ini`: 8.5 uses `Europe/Kyiv`; 8.2–8.4 intentionally keep the deprecated `Europe/Kiev`.

When changing extensions, packages, `php.ini`, or `policy.xml`, apply the change to every version's directory unless it is intentionally version-specific, and keep them otherwise identical (`diff` between versions should show only the lines above).

## Conventions in the Dockerfiles

- Runtime libs go in a plain `apk add --no-cache`; compile-time `-dev` packages go in a `--virtual` build-deps group that is deleted in the same `RUN` layer.
- Each Dockerfile ends with `RUN ! php -m 2>&1 | grep -iE 'warning|error'` — the build fails if any PHP extension emits a startup warning/error. Keep this as the last build check when adding extensions.
- Oracle Instant Client is downloaded from oracle.com at build time (controlled by `ORACLE_IC_VERSION` / `ORACLE_IC_DIR` build args), not committed to git. It needs `gcompat` because it is glibc-built.
- Base images ship Composer (copied from `composer:latest`), use `WORKDIR /www`, and expose php-fpm on 9000.

## Building

Full rebuild/push instructions (single version and all versions) are in `README.md`. Key points:

- Build in order base → ldap → oci; all images are tagged `:latest`.
- Use `--pull` only for the base image. With `--pull` on ldap/oci, Docker fetches the old parent from Docker Hub instead of the freshly built local one.

```bash
docker build --pull -t strilezkijslawa/php8.4-alpine:latest      strilezkijslawa-php84-alpine-stable
docker build        -t strilezkijslawa/php8.4-ldap-alpine:latest strilezkijslawa-php84-alpine-ldap-stable
docker build        -t strilezkijslawa/php8.4-oci-alpine:latest  strilezkijslawa-php84-alpine-oci-stable
```

Quick verification of a built image: `docker run --rm <tag> php -m`.
