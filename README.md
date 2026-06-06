# Contribution 1: Add inputMode='numeric' for TOTP and OTP input fields

**Contribution Number:** 1  
**Student:** Quang Pham  
**Issue:** https://github.com/marketcalls/openalgo/issues/1009  
**Status:** Phase IV Complete  

---

## Why I Chose This Issue

I chose this issue because the tech stack aligns closely with what I already know. OpenAlgo's frontend is built with React 19, TypeScript, shadcn/ui, and Tailwind CSS — tools I've used hands-on in my own projects. That familiarity means I can focus on actually solving the issue and learning the Claude Code workflow rather than spending time ramping up on an unfamiliar stack.

The fix itself is also well-scoped: three files, one logical change. That made it a good candidate for exploring how Claude Code navigates a real codebase and executes a targeted fix, which is the point of the exercise.

The repo itself also caught my attention. OpenAlgo is a self-hosted algorithmic trading platform with 30+ broker integrations, a visual no-code strategy builder, and an MCP server for AI-assisted trading. It's a genuinely ambitious open source project, and contributing to something with real users feels more meaningful than a toy repo.

---

## Understanding the Issue

### Problem Description

The TOTP/OTP input fields in the React frontend use `type="text"` without an `inputMode` hint. Because the browser has no signal that the field is numeric-only, mobile devices render the full alphanumeric keyboard instead of a numeric keypad, making it harder for users to enter their 6-digit authentication codes.

### Expected Behavior

When a user focuses a TOTP/OTP field on a mobile device, the device should present a numeric keypad. The field should still avoid the up/down spinner arrows that `type="number"` introduces.

### Current Behavior

Focusing the TOTP field on the Reset Password page brings up the standard full keyboard, requiring the user to switch to the numeric layout manually before entering the code.

### Affected Components

React frontend (`frontend/src/`), specifically the authentication/TOTP input fields:

- `frontend/src/pages/ResetPassword.tsx` — TOTP input (`#totp`) — **missing** `inputMode`.
- `frontend/src/pages/Login.tsx` — TOTP input (`#totp_code`) — already had `inputMode="numeric"` and `pattern`.
- `frontend/src/components/auth/TwoFactorEnforcement.tsx` — the TOTP setup/verification input used from the Profile flow — already had `inputMode="numeric"` and `pattern`.

The shared `Input` component (`frontend/src/components/ui/input.tsx`) spreads `{...props}` onto the native `<input>`, so `inputMode` and `pattern` pass through without any component change.

---

## Reproduction Process

### Environment Setup

- Backend runs via `uv run app.py` (Python 3.12+, `uv` package manager).
- Frontend dependencies installed with `npm install` in `frontend/`. The repo keeps `frontend/dist/` out of feature branches (gitignored locally; CI force-commits the build on `main`), so a local build with Node was needed to typecheck the change.

### Steps to Reproduce

1. Open the React frontend and navigate to the Reset Password flow's TOTP step.
2. Using browser DevTools mobile emulation (device toolbar), focus the "TOTP Code" input.
3. Observed result: the input was declared `type="text"` with no `inputMode`, so the device does not request a numeric keypad.

### Reproduction Evidence

- **Commit showing reproduction:** No separate reproduction commit was required — the bug is reproducible by inspection of `frontend/src/pages/ResetPassword.tsx` in its pre-fix state (the `#totp` input declared `type="text"` with no `inputMode`). That state corresponds to the parent of fix commit `6ff9bac3`.
- **Screenshots/logs:** Not captured as device screenshots; verified instead via DevTools element inspection of the rendered `<input>` attributes.
- **My findings:** Of the three files named in the issue, two (`Login.tsx` and `TwoFactorEnforcement.tsx`) already carried `inputMode="numeric"` and a numeric `pattern`. Only `ResetPassword.tsx` was missing the `inputMode` hint, so it was the single file requiring a change.

---

## Solution Approach

### Analysis

The root cause is a missing `inputMode="numeric"` attribute on the TOTP input. Without it, browsers fall back to the default text keyboard on mobile. The issue is purely a client-side input-attribute fix; no logic change is required because the `onChange` handler already filters to digits.

### Proposed Solution

Add the standard mobile numeric-keyboard hints to the `#totp` input on `ResetPassword.tsx`:

- Add `inputMode="numeric"` (numeric keypad on Android and modern browsers).
- Change `pattern="[0-9]{6}"` to `pattern="[0-9]*"` (the iOS numeric-keyboard trigger, as specified in the issue).
- Keep `type="text"` to avoid `type="number"` spinner arrows.
- Leave the existing `onChange` digit filter (`.replace(/\D/g, '').slice(0, 6)`) and `maxLength={6}` unchanged.

### Implementation Plan

**Understand:** TOTP inputs need an `inputMode`/`pattern` hint so mobile devices show a numeric keypad, without introducing number spinners.

**Match:** `Login.tsx` and `TwoFactorEnforcement.tsx` already implement exactly this pattern (`inputMode="numeric"` + numeric `pattern` + `type="text"`), providing an in-codebase reference to match.

**Plan:**
1. Edit the `#totp` `<Input>` in `frontend/src/pages/ResetPassword.tsx`.
2. Add `inputMode="numeric"` and change `pattern="[0-9]{6}"` → `pattern="[0-9]*"`.
3. Verify via a frontend build, lint, and DevTools mobile emulation.

**Implement:**
- Branch: `fix/totp-numeric-keyboard`
- Commit: `6ff9bac3` — `fix(frontend): show numeric keypad for TOTP input on ResetPassword`

**Review:** Single-file, three-line change; follows Conventional Commits (`fix:` scope); matches existing TOTP-input pattern in the codebase; no behavior change beyond the keyboard hint.

**Evaluate:** Frontend build + lint pass and the rendered input carries the expected attributes (see Testing).

---

## Testing Strategy

### Unit Tests

No unit tests were added — descoped intentionally. The change is a presentational input-attribute fix (`inputMode`/`pattern`) that introduces no new logic; the existing `onChange` digit filter is unchanged, so there is no new unit-testable behavior. The project has no existing frontend test suite covering these auth inputs to extend.

### Integration Tests

Not applicable. The attribute change does not affect form submission, validation, or API behavior — the TOTP verification flow itself is unchanged.

### Manual Testing

- `npm run build` (tsc + vite) — passes, no TypeScript errors.
- `biome lint src/pages/ResetPassword.tsx` — clean.
- DevTools mobile emulation: the rendered `<input>` carries `inputmode="numeric"` and `pattern="[0-9]*"` with `type="text"` (no spinner arrows); non-digit characters are still ignored by the existing `onChange` filter.

### Acceptance Criteria

- [x] Numeric keyboard appears on mobile for TOTP inputs.
- [x] No spinner arrows (uses `type="text"`, not `type="number"`).
- [x] Works on both iOS (`pattern="[0-9]*"`) and Android (`inputMode="numeric"`).

---

## Implementation Notes

### Implementation Progress

- Explored the three files named in the issue plus the shared `Input` component, and searched the frontend for any other OTP/TOTP inputs. Found that `Login.tsx` and `TwoFactorEnforcement.tsx` were already fixed, which narrowed the work to `ResetPassword.tsx`.
- Applied the attribute change, installed frontend deps (`npm install`), then ran `npm run build` (tsc + vite) and `biome lint` to verify, and confirmed the rendered attributes via DevTools.
- Committed as `6ff9bac3` on branch `fix/totp-numeric-keyboard` and pushed.

### Code Changes

- **Files modified:** `frontend/src/pages/ResetPassword.tsx`
- **Key commit:** `6ff9bac3` (branch `fix/totp-numeric-keyboard`)
- **Diff:**

```diff
   <Input
     id="totp"
     type="text"
+    inputMode="numeric"
     value={totpCode}
     onChange={(e) => setTotpCode(e.target.value.replace(/\D/g, '').slice(0, 6))}
     placeholder="000000"
-    pattern="[0-9]{6}"
+    pattern="[0-9]*"
     maxLength={6}
     className="text-center text-2xl tracking-widest"
     required
     autoFocus
   />
```

- **Approach decisions:** Scoped the change to `ResetPassword.tsx` only, since `Login.tsx` and `TwoFactorEnforcement.tsx` already had the fix. Other OTP inputs (`BrokerTOTP.tsx` broker configs, `SamcoAuth.tsx`) also lack one or both attributes but are outside this issue's scope and left as follow-ups. Used the issue-specified `pattern="[0-9]*"`.

---

## Pull Request

**PR Link:** https://github.com/marketcalls/openalgo/pull/1478

**PR Description:**

> Closes #1009
>
> TOTP/OTP input fields used `type="text"` without `inputMode="numeric"`, so mobile devices showed a full alphanumeric keyboard instead of a numeric keypad. This adds `inputMode="numeric"` and `pattern="[0-9]*"` to the TOTP input on the Reset Password page, keeping `type="text"` to avoid spinner arrows. `Login.tsx` and the 2FA-setup input already had this fix, so only `ResetPassword.tsx` needed the change.

**Maintainer Feedback:**

- 2026-06-04: PR #1478 merged by the maintainer with no change requests.

> Successfully added inputMode="numeric" to the Reset Password TOTP field, ensuring a mobile numeric keypad appears for users. Scope investigation was validated as accurate.
>
> The owner added a commit to revert the pattern attribute change from `[0-9]*` back to `[0-9]{6}`.
>
> **Reasoning:** `inputMode="numeric"` handles the modern mobile keypad natively, making the `[0-9]*` legacy trick redundant. Keeping `[0-9]{6}` preserves exact 6-digit HTML5 validation and maintains consistency with the project's existing TOTP inputs.
>
> **To do:** No further action needed.

**Status:** Merged (2026-06-04)

---

## Learnings & Reflections

### Technical Skills Gained

- Mobile-first form design and HTML input attributes for mobile UX (`inputMode`, `pattern`, `type` trade-offs).
- Cross-platform input handling (iOS `pattern="[0-9]*"` vs. Android `inputMode="numeric"`).
- Navigating an unfamiliar large codebase to verify an issue's stated scope before changing code, and identifying the shared component (`ui/input.tsx`) that makes the one-line fix work without component changes.

### Challenges Overcome

- **The issue's stated scope was partly stale.** It named three files, but two (`Login.tsx`, `TwoFactorEnforcement.tsx`) already had `inputMode`/`pattern`. Rather than trust the issue text, I inspected each file and confirmed only `ResetPassword.tsx` needed the change — avoiding redundant edits and keeping the PR tightly scoped.
- **`frontend/dist/` is gitignored locally but tracked on `main`.** A local `npm run build` regenerated hashed bundle files and produced a noisy working tree. I learned these are CI-managed artifacts (force-committed on `main` by the `commit-dist` job), so I kept them out of the commit and limited the PR to the single source change.

### What I'd Do Differently Next Time

- Verify the issue's claimed file list against the current codebase before planning — issues can go stale as other PRs land, so I'd front-load that check.
- Several other OTP/TOTP inputs (`BrokerTOTP.tsx` broker configs, `SamcoAuth.tsx`) are missing the same attributes. Next time I'd raise this with the maintainers up front to decide whether to bundle them or open a follow-up issue, instead of only noting it as out-of-scope.

---

## Resources Used

- [MDN — `inputmode` global attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/inputmode)
- [MDN — `<input>` `pattern` attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/pattern)
- GitHub issue [#1009](https://github.com/marketcalls/openalgo/issues/1009) — problem statement and acceptance criteria.
- In-repo reference pattern: `frontend/src/pages/Login.tsx` and `frontend/src/components/auth/TwoFactorEnforcement.tsx` (existing `inputMode`/`pattern` usage to match).
