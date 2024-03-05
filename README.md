# privatenumber/pkgroll#54, Swatinem/rollup-plugin-dts#127 reproduction

This repository demonstrates the issues in Swatinem/rollup-plugin-dts
(Swatinem/rollup-plugin-dts#127) with `composite` Typescript monorepos. When
using pkgroll, this will be fixed by privatenumber/pkgroll#54.

To see the issue, simply run:

```console
$ pnpm install
$ pnpm build
> pkgroll-monorepo-types-issue@0.0.0 build /Users/halostatue/code/pkgroll-monorepo-types-issue
> pnpm recursive run build

Scope: 2 of 3 workspace projects
packages/one build$ tsc --noEmit && pkgroll
│ src/index.ts(1,27): error TS6307: File '/Users/halostatue/code/pkgroll-monorepo-types-issue/packages/one/src…
│   File is ECMAScript module because '/Users/halostatue/code/pkgroll-monorepo-types-issue/packages/one/packag…
│ Error [RollupError]: Failed to compile. Check the logs above.
│     at error (/Users/halostatue/code/pkgroll-monorepo-types-issue/node_modules/.pnpm/rollup@3.29.4/node_modu…
│     at Object.error (/Users/halostatue/code/pkgroll-monorepo-types-issue/node_modules/.pnpm/rollup@3.29.4/no…
│     at Object.error (/Users/halostatue/code/pkgroll-monorepo-types-issue/node_modules/.pnpm/rollup@3.29.4/no…
│     at c (/Users/halostatue/code/pkgroll-monorepo-types-issue/node_modules/.pnpm/pkgroll@2.0.1_typescript@5.…
│     at Object.transform (/Users/halostatue/code/pkgroll-monorepo-types-issue/node_modules/.pnpm/pkgroll@2.0.…
│     at /Users/halostatue/code/pkgroll-monorepo-types-issue/node_modules/.pnpm/rollup@3.29.4/node_modules/rol…
│   id: '/Users/halostatue/code/pkgroll-monorepo-types-issue/packages/one/src/index.ts',
│   hook: 'transform',
│   code: 'PLUGIN_ERROR',
│   plugin: 'dts',
│   watchFiles: [
│     '/Users/halostatue/code/pkgroll-monorepo-types-issue/packages/one/src/index.ts',
│     '/Users/halostatue/code/pkgroll-monorepo-types-issue/packages/one/src/other.ts'
│   ]
│ }
└─ Failed in 844ms at /Users/halostatue/code/pkgroll-monorepo-types-issue/packages/one
/Users/halostatue/code/pkgroll-monorepo-types-issue/packages/one:
 ERR_PNPM_RECURSIVE_RUN_FIRST_FAIL  @org/one@0.0.0 build: `tsc --noEmit && pkgroll`
Exit status 1
 WARN   Local package.json exists, but node_modules missing, did you mean to install?
 ELIFECYCLE  Command failed with exit code 1.
```

The "fix" in privatenumber/pkgroll#54 can be demonstrated with:

```console
$ git checkout with-pkgroll-54
$ pnpm build
❯ pnpm build

> pkgroll-monorepo-types-issue@0.0.0 build /Users/halostatue/code/pkgroll-monorepo-types-issue
> pnpm recursive run build

Scope: 2 of 3 workspace projects
packages/one build$ tsc --noEmit && pkgroll
└─ Done in 1.3s
packages/two build$ tsc --noEmit && pkgroll
└─ Done in 492ms
```

The change there simply sets `composite: false` while building types with
Swatinem/rollup-plugin-dts. That is, it is not a fix of the underlying bug or
changing the processor to something which understands composite Typescript
projects, but throwing open an escape hatch which should not work (if
`references` are present in `tsconfig.json`, the referenced project _must_ be
marked `composite: true`, so a middle layer package that also exports types
_may_ still demonstrate this issue).
