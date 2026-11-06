# Stage: integrate

Goal: give the frontend team, or any team integrating with this work, exactly what they need to start. Two files with the same facts: `INTEGRATION.md` (for their AI agents: terse, exact, consistent headings) and `INTEGRATION.html` (for humans: same content, easier to scan). Use a fresh context.

## Read

The final code (diff and current state of the public surface), PLAN.md, TEST-RESULTS.md, DEVIATIONS.md. Not the chat. Document what was actually built and tested, not what the plan once said.

## Content (one page, keep it short)

Use these headings in this order, and skip a section only if it truly doesn't apply:

1. **What changed**: one or two sentences, what it does, not how.
2. **How to use**: endpoints with request/response examples; or for on-chain work: program ID, network, instruction names, accounts (with signer/writable), and arguments.
3. **Types and data formats**: fields the integrator needs, units and decimals, address encoding, enums, nullable fields.
4. **Errors**: each error code/message and what the integrator should do about it.
5. **Flow**: the order of calls in 3-6 steps (e.g. approve, submit, wait for confirmation, poll status).
6. **Gotchas**: only things that cause a bug if missed: auth, rate limits, confirmation/finality waits, environment differences, breaking changes.
7. **Quick test**: one working example (curl or minimal call) to verify the integration works.

## Rules

- **Delete and rewrite, don't patch.** If these files already exist, regenerate both from the current code. Patched docs drift and keep stale text.
- **Hard length limit: about one page.** If it's longer, cut explanation and keep examples.
- **No internal details**: no refactors, deviation history, or reasoning behind decisions. That is in NOTES.md.
- **Only verified examples.** Take them from the code, tests, or a real run. If you can't verify one, leave it out or label it clearly as unverified.
- Define any term an outsider wouldn't know. Write for someone who has never seen the codebase.
- State what is not included only when an integrator might reasonably assume it is.
- Generate the HTML from the MD content so they can't disagree: a single self-contained file, simple readable styling, code blocks for examples, no external dependencies.

Finish by telling the user where the files are.
