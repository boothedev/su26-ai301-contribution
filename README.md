# Contribution 1: Add inputMode='numeric' for TOTP and OTP input fields

**Contribution Number:** 1  
**Student:** Quang Pham  
**Issue:** https://github.com/marketcalls/openalgo/issues/1009  
**Status:** Phase I Complete  

---

## Why I Chose This Issue

I chose this issue because the tech stack aligns closely with what I already know. OpenAlgo's frontend is built with React 19, TypeScript, shadcn/ui, and Tailwind CSS - tools I've used hands-on in my own projects. That familiarity means I can focus on actually solving the issue and learning the Claude Code workflow rather than spending time ramping up on an unfamiliar stack.

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

- `frontend/src/pages/ResetPassword.tsx` - TOTP input (`#totp`) - **missing** `inputMode`.
- `frontend/src/pages/Login.tsx` - TOTP input (`#totp_code`) - already had `inputMode="numeric"` and `pattern`.
- `frontend/src/components/auth/TwoFactorEnforcement.tsx` - the TOTP setup/verification input used from the Profile flow - already had `inputMode="numeric"` and `pattern`.

The shared `Input` component (`frontend/src/components/ui/input.tsx`) spreads `{...props}` onto the native `<input>`, so `inputMode` and `pattern` pass through without any component change.
