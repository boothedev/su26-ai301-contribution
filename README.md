# Contribution 1: Add inputMode='numeric' for TOTP and OTP input fields

**Contribution Number:** 1  
**Student:** Quang Pham  
**Issue:** https://github.com/marketcalls/openalgo/issues/1009  
**Status:** Phase II Complete  

---

## Why I Chose This Issue

I chose this issue because the tech stack aligns closely with what I already know. OpenAlgo's frontend is built with React 19, TypeScript, shadcn/ui, and Tailwind CSS - tools I've used hands-on in my own projects. That familiarity means I can focus on actually solving the issue and learning the Claude Code workflow rather than spending time ramping up on an unfamiliar stack.

The fix itself is also well-scoped: three files, one logical change. That made it a good candidate for exploring how Claude Code navigates a real codebase and executes a targeted fix, which is the point of the exercise.

The repo itself also caught my attention. OpenAlgo is a self-hosted algorithmic trading platform with 30+ broker integrations, a visual no-code strategy builder, and an MCP server for AI-assisted trading. It's a genuinely ambitious open source project, and contributing to something with real users feels more meaningful than a toy repo.

---

## Understanding the Issue

### Problem Description

The TOTP/OTP input fields in the React frontend use `type="text"` without an `inputMode` hint. Because the browser has no signal that the field is numeric-only, mobile devices render the full alphanumeric keyboard instead of a numeric keypad, making it harder for users to enter their 6-digit authentication codes.

### Affected Components

React frontend (`frontend/src/`), specifically the authentication/TOTP input fields:

- `frontend/src/pages/ResetPassword.tsx` - TOTP input (`#totp`) - **missing** `inputMode`.
- `frontend/src/pages/Login.tsx` - TOTP input (`#totp_code`) - already had `inputMode="numeric"` and `pattern`.
- `frontend/src/components/auth/TwoFactorEnforcement.tsx` - the TOTP setup/verification input used from the Profile flow - already had `inputMode="numeric"` and `pattern`.

The shared `Input` component (`frontend/src/components/ui/input.tsx`) spreads `{...props}` onto the native `<input>`, so `inputMode` and `pattern` pass through without any component change.

### Acceptance Criteria

- [ ] Numeric keyboard appears on mobile for TOTP inputs
- [ ] No spinner arrows (use type="text", not type="number")
- [ ] Works on both iOS and Android

---

## Reproduction Process

### Environment Setup

- Backend runs via `uv run app.py` (Python 3.12+, `uv` package manager).
- Frontend dependencies installed with `npm install` in `frontend/`. The repo keeps `frontend/dist/` out of feature branches (gitignored locally; CI force-commits the build on `main`), so a local build with Node was needed to typecheck the change.

### Steps to Reproduce

1. Open the React frontend and navigate to the Reset Password flow's TOTP step.
2. Using browser DevTools mobile emulation (device toolbar), focus the "TOTP Code" input.
3. Observed result: the input was declared `type="text"` with no `inputMode`, so the device does not request a numeric keypad.

### Expected Behavior

When a user focuses a TOTP/OTP field on a mobile device, the device should present a numeric keypad. The field should still avoid the up/down spinner arrows that `type="number"` introduces.

### Current Behavior

Focusing the TOTP field on the Reset Password page brings up the standard full keyboard, requiring the user to switch to the numeric layout manually before entering the code.

### Reproduction Evidence

- **Commit showing regression:**: https://github.com/boothedev/openalgo/commit/7ce35812c2a43d35f85af78e650c895680e6431b
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
