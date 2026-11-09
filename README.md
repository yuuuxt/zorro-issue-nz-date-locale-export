# Regression: NZ_DATE_LOCALE is no longer exported from ng-zorro-antd/i18n in 22.2.0

This directory is a standalone reproduction. Copy or publish only this directory;
it does not require the parent application, Angular CLI, or a browser.
The error occurs during TypeScript module checking, so no Angular application
bootstrap is needed.

## Reproduce

Run these commands from this directory:

```sh
npm ci --ignore-scripts --no-audit --no-fund
npm run repro
```

The dependencies are pinned and the lockfile is included.
The reproduction intentionally exits with code 2.

## Minimal source

See [repro.ts](./repro.ts):

```ts
import { NZ_DATE_LOCALE, NzI18nModule, provideNzI18n } from 'ng-zorro-antd/i18n';

export { NZ_DATE_LOCALE, NzI18nModule, provideNzI18n };
```

## Expected behavior

The import compiles, as it does with ng-zorro-antd 22.1.2.

## Actual behavior

With ng-zorro-antd 22.2.0, TypeScript reports:

```text
repro.ts(1,10): error TS2724: '"ng-zorro-antd/i18n"' has no exported member named 'NZ_DATE_LOCALE'. Did you mean 'DateLocale'?
```

`NzI18nModule` and `provideNzI18n` remain available from the same entry point.

## Verified comparison

| ng-zorro-antd | Import | Result |
| --- | --- | --- |
| 22.1.2 | All three symbols from `ng-zorro-antd/i18n` | Pass, exit 0 |
| 22.2.0 | All three symbols from `ng-zorro-antd/i18n` | TS2724, exit 2 |
| 22.2.0 | `NZ_DATE_LOCALE` from `ng-zorro-antd/core/time`, other symbols from `ng-zorro-antd/i18n` | Pass, exit 0 |

Angular, TypeScript, the compiler configuration, and the original source were
unchanged for the first two checks. This demonstrates an import compatibility
regression between these releases; it does not establish whether the removal
was intentional.

To repeat the 22.1.2 comparison without changing the manifest or lockfile:

```sh
npm install --no-save --package-lock=false --ignore-scripts --no-audit --no-fund ng-zorro-antd@22.1.2
npm run repro
```

Restore the locked failing version afterward:

```sh
npm ci --ignore-scripts --no-audit --no-fund
npm run repro
```

## Workaround

Replace the import in [repro.ts](./repro.ts) with:

```ts
import { NZ_DATE_LOCALE } from 'ng-zorro-antd/core/time';
import { NzI18nModule, provideNzI18n } from 'ng-zorro-antd/i18n';
```

This compiles with 22.2.0. Only import compatibility was tested, not runtime
date formatting or date-adapter configuration.

In the published 22.2.0 package, the token is declared and exported by
`core/time`, but is not re-exported by `i18n` in either its type declarations
or its JavaScript bundle.

## Environment

- ng-zorro-antd: 22.2.0 (failing), 22.1.2 (passing)
- Angular and Angular CDK: 22.2.4
- TypeScript: 6.0.3
- Node.js: 24.15.0
- npm: 11.12.1
- OS: Windows
- TypeScript module resolution: `bundler`

`skipLibCheck` is enabled to isolate the consuming source's missing-export
diagnostic from dependency-internal declaration checks.

## Suggested issue request

Please restore the `NZ_DATE_LOCALE` re-export from `ng-zorro-antd/i18n` for
backward compatibility, or clarify whether this is an intentional breaking
change and document the migration.

This README can serve as the issue body. Publish this standalone directory
(excluding `node_modules`) and include its URL in the issue's reproduction field.
