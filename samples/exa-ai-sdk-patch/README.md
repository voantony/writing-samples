# Patch for `@exalabs/ai-sdk`

This is a patch against [exa-labs/ai-sdk](https://github.com/exa-labs/ai-sdk) (`@exalabs/ai-sdk` 2.1.0). It is not an official Exa package. It has not been submitted as a pull request, and it is not published to npm.

The original package is MIT-licensed and authored by [Exa](https://exa.ai). This repository keeps that license.

## What's broken in the published 2.1.0

- **AI SDK 7 does not install.** `@exalabs/ai-sdk@2.1.0` declares `peerDependencies.ai` as `^6.0.0`. `ai@7` has been `latest` since 25 June 2026, so `npm install ai@7 @exalabs/ai-sdk` fails with ERESOLVE. Users have asked for a v7 update on [issue #4](https://github.com/exa-labs/ai-sdk/issues/4), and one comment there describes vendoring the package to get around the peer conflict.
- **`abortSignal` is ignored.** The tool does not forward the AI SDK's `abortSignal` to `fetch`, so cancelling a run does not cancel the Exa request. See [issue #22](https://github.com/exa-labs/ai-sdk/issues/22).
- **`contents.maxAgeHours` is documented but not accepted.** The example on [Exa's AI SDK docs](https://exa.ai/docs/integrations/vercel/ai-sdk) uses `maxAgeHours`. In 2.1.0 it fails type-checking and is dropped at runtime.

The install and type-check failures are recorded in `test-output/`.

## What the patch changes

- `peerDependencies`: `ai ^6.0.0 || ^7.0.0`, and `zod ^3.25.76 || ^4.1.8` (the same zod range `ai` itself declares). zod moves from `dependencies` to `peerDependencies`.
- `execute()` forwards `abortSignal` to `fetch`, rejects immediately if the signal is already aborted, and rethrows `AbortError` instead of wrapping it. ([#22](https://github.com/exa-labs/ai-sdk/issues/22))
- Non-2xx responses throw `ExaApiError` (`status`, `tag`, `requestId`, `retryAfterSeconds`, `body`). The message text is unchanged from 2.x.
- Optional `config.fetch` for proxies and tests.
- Tool output is typed as `ExaApiResponse` instead of `unknown`.
- Stop sending the deprecated `contents.livecrawl: "fallback"` on every call (omitting it uses the API default). Add `contents.maxAgeHours`. Passing `livecrawl` together with `maxAgeHours` throws up front, because the API rejects that combination. Also adds the `lite` / `deep-lite` types, `people` / `publication` categories, and `highlights.maxCharacters`, and marks deprecated options, following Exa's OpenAPI spec.

`ai-sdk/` in this repository is the three-commit series applied on `main` at [a7c4761](https://github.com/exa-labs/ai-sdk/commit/a7c4761fe6bcc96682f5c46bb0a47d204b0d2628) (the 2.1.0 release). A one-commit variant in `patches-on-pr21/` stacks on the open [PR #21](https://github.com/exa-labs/ai-sdk/pull/21) instead. Use that if #21 is about to merge; it leaves #21's work alone and only adds what #21 does not already cover.

## How it was tested

The published 2.1.0 package had no tests. This patch adds:

- `npm test`: 10 `node:test` cases against the built `dist/`. They call the real `https://api.exa.ai/search` with a deliberately invalid key, so the real 401 `INVALID_API_KEY` comes back. No Exa responses are faked. Request bodies are validated with ajv against the live `https://exa.ai/docs/exa-spec.json` `SearchRequest` schema. Cancellation is covered for a pre-aborted signal and for a mid-flight abort.
- `npm run test:compat`: packs the package and installs it with plain `npm install` (no `--legacy-peer-deps`) into fresh projects for ai@6/7 × zod@3/4. Each project type-checks the example from Exa's docs, runs `generateText` with the AI SDK test model, and checks that `abortSignal` reaches the Exa request.

The same matrix was run against the published 2.1.0 for comparison. Raw output is in `test-output/`.

| combination | published 2.1.0 | this patch |
| --- | --- | --- |
| ai@6 zod@3 | install ok; types fail (`maxAgeHours`); runtime 1 pass / 1 fail (`abortSignal` not forwarded) | install ok; types ok; runtime 2 pass / 0 fail |
| ai@6 zod@4 | same as above | install ok; types ok; runtime 2 pass / 0 fail |
| ai@7 zod@3 | install fails (ERESOLVE, peer `ai@"^6.0.0"`) | install ok; types ok; runtime 2 pass / 0 fail |
| ai@7 zod@4 | install fails (ERESOLVE) | install ok; types ok; runtime 2 pass / 0 fail |

Unit tests were 10/10 on Node 20 and Node 24. The tests need outbound HTTPS to api.exa.ai and exa.ai, and no secrets.

## What's untested without an API key

There is no Exa API key in this work, so these were not tested:

- A successful search response through the tool
- Whether the API returns warnings for the old `livecrawl` default
- Real 429 / `Retry-After` values

`ExaApiError` parsing was only exercised on a real 401 body.

## How to apply it

The three-commit series in `patches/` applies on `main` of [exa-labs/ai-sdk](https://github.com/exa-labs/ai-sdk) at `a7c4761` (the 2.1.0 release):

```bash
git clone https://github.com/exa-labs/ai-sdk.git
cd ai-sdk
git checkout a7c4761fe6bcc96682f5c46bb0a47d204b0d2628
git am /path/to/exa-ai-sdk-patch/patches/*.patch
npm ci
npm test
npm run test:compat
```

To stack on open PR #21 instead:

```bash
git clone https://github.com/exa-labs/ai-sdk.git
cd ai-sdk
git fetch origin pull/21/head:pr21
git checkout pr21
git am /path/to/exa-ai-sdk-patch/patches-on-pr21/*.patch
```

The patched source (three commits on `main`) is also in `ai-sdk/` in this repository, so you can inspect or run it without applying the patches:

```bash
cd ai-sdk
npm ci
npm test
npm run test:compat
```

A draft pull request description is in `PR_DESCRIPTION.md`. No pull request has been opened on GitHub, and this patch is not published to npm.

## License

[MIT](LICENSE), same as the upstream package. Copyright for the original `@exalabs/ai-sdk` source belongs to Exa. The patch, tests, and documentation in this repository are also MIT-licensed.
