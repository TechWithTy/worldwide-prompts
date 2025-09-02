---
description: Toggle between real APIs and faker mocks with NEXT_PUBLIC_APP_TESTING_MODE
---

# Goal
Ensure the app cleanly switches between real API data and faker-based mock data using the `NEXT_PUBLIC_APP_TESTING_MODE` flag, without runtime surprises. This workflow guides you through verifying the env flag, hardening exports, refactoring consumers, and validating with lint/tests.

# Preconditions
- You have `pnpm` installed.
- You can edit `.env.local`.
- The faker modules live under `constants/_faker/` and most exports are gated by `NEXT_PUBLIC_APP_TESTING_MODE && ...`.

# Steps

1. Verify env flag in `.env.local`
   - Open `.env.local` and ensure it contains:
     - `NEXT_PUBLIC_APP_TESTING_MODE=true` to use faker data
     - `NEXT_PUBLIC_APP_TESTING_MODE=false` to use real APIs
   - If using Next/Vite, restart dev server after changing envs.

2. Audit current faker exports
   - Search for all usages of the flag and gated exports:
     - Command: `rg -n "NEXT_PUBLIC_APP_TESTING_MODE"` (or use your IDE search)
     - Command: `rg -n "constants/_faker"`
   - Note that many files export like: `export const mockX = NEXT_PUBLIC_APP_TESTING_MODE && generateX()`.
   - Outcome: list modules that return `false` when the flag is off.

3. Decide a consistent consumption pattern
   Choose ONE of the patterns below project-wide and stick to it:
   - Pattern A (preferred for runtime safety): Export generators, and consumers call them only when `NEXT_PUBLIC_APP_TESTING_MODE` is true.
     - Example consumer:
       ```ts
       import { generateSampleTextMessage } from "@/constants/_faker/texts/texts";
       const messages = NEXT_PUBLIC_APP_TESTING_MODE ? generateSampleTextMessage() : [];
       ```
   - Pattern B (keep constants but guard): Keep `mockX = NEXT_PUBLIC_APP_TESTING_MODE && ...` in faker files; consumers must coalesce:
       ```ts
       import { mockTexts } from "@/constants/_faker/texts/texts";
       const texts = mockTexts || [];
       ```

4. Harden faker modules (if adopting Pattern A)
   - Ensure each faker file exports the generator function(s) (e.g., `generateSampleEmails`, `generateCallData`, etc.).
   - Keep (or remove) the `mock*` constants as convenience, but mark them as testing-only in a JSDoc comment.
   - Avoid exporting `false`-typed values by setting clear types when needed:
     ```ts
     export const mockTexts: TextMessage[] | false = NEXT_PUBLIC_APP_TESTING_MODE && generateSampleTextMessages();
     ```
   - Prefer type-only imports per Biome rules: `import type { X } from '...'`.

5. Refactor consumers to respect the flag
   - For components/hooks/services that currently import `mock*` constants directly:
     - Replace with guarded usage or swap to generator calls.
     - Example:
       ```ts
       // Old (unsafe when flag is false)
       const data = mockCallCampaignData;
       // New (safe)
       const data = (NEXT_PUBLIC_APP_TESTING_MODE ? mockCallCampaignData : []) || [];
       // or
       const data = NEXT_PUBLIC_APP_TESTING_MODE ? generateCallData() : [];
       ```

6. Centralize a small helper (optional but recommended)
   - Create a helper `getMock.ts` in `constants/_faker/`:
     ```ts
     import { NEXT_PUBLIC_APP_TESTING_MODE } from "@/constants/data";
     export function getMock<T>(factory: () => T, fallback: T): T {
       return NEXT_PUBLIC_APP_TESTING_MODE ? factory() : fallback;
     }
     ```
   - Consumers:
     ```ts
     const items = getMock(() => generateSampleTextMessages(20), []);
     ```

7. Validate no runtime falsy leaks remain
   - Search for direct uses without coalescing:
     - `rg -n "mock[A-Z][A-Za-z0-9]*\b"` and check each usage is guarded or replaced.
   - Confirm all `constants/_faker/**` exports are either generators or have clear `T | false` typing.

8. Switch to real APIs when ready
   - Set `NEXT_PUBLIC_APP_TESTING_MODE=false` in `.env.local`.
   - Ensure API services/queries are enabled and hydrated in place of mocks.
   - Keep the faker paths intact for local dev/testing toggles.

9. Run lint and format (Biome)
   - `pnpm biome check .`
   - `pnpm biome format .`
   - Fix any type-only import or style nits Biome reports.

10. Smoke test
   - Start the app and exercise UI flows that previously read faker data.
   - Verify:
     - With `NEXT_PUBLIC_APP_TESTING_MODE=true`: UI shows mock data (lists populated).
     - With `NEXT_PUBLIC_APP_TESTING_MODE=false`: UI falls back to API or empty states gracefully (no crashes).

# Notes
- Many modules under `constants/_faker/` already use the `NEXT_PUBLIC_APP_TESTING_MODE && ...` pattern. The critical change is to ensure consumers never assume a non-falsy value.
- For reproducible tests, you may seed faker at module tops: `faker.seed(123);`.
- Respect Biome: prefer type-only imports, avoid unnecessary template literals, and provide explicit button `type` in React when adding buttons.
