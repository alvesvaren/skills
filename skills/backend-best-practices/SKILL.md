---
name: backend-best-practices
description: Server-side conventions for Node/Hono APIs — few layers, Zod-validated route input, typed AppError mapped to HTTP in one onError, Drizzle for data, tRPC/Hono RPC for internal contracts and OpenAPI for public ones. Use when writing or refactoring API routes, server logic, database access, or endpoint design.
---

# Backend best practices

Conventions for server-side TypeScript. [typescript-best-practices](../typescript-best-practices/SKILL.md) applies in full; this covers what is specific to APIs.

**Stack defaults:** Hono, Drizzle, Zod.

## Layers

Default to few layers: route → domain function → db is usually the whole stack. Abstractions are welcome when they earn their place — a real second consumer, a deep module hiding genuine complexity, a boundary expected to churn. What doesn't earn a place is ceremony added for symmetry: a service class per table, a repository interface with one implementation.

## Contract

- Internal or single-consumer APIs: end-to-end typed — tRPC or Hono RPC client — so types flow and iteration stays fast.
- Public or multi-consumer APIs: OpenAPI (`@hono/zod-openapi`) so consumers get a spec; accept the slower iteration as the price of a published contract.
- Either way the contract derives from the route schemas — a hand-maintained spec is a second source of truth.

## Validation

Every route input — body, params, query, any header you read — is parsed with Zod at the route boundary (`zValidator` in Hono, `input()` schemas in tRPC). Past that line, data is typed and trusted. Env vars are parsed once at startup with a Zod schema so bad config crashes at boot, not mid-request.

## Errors

Domain code throws a typed `AppError` with a coarse closed set of codes; one `onError` maps codes to HTTP. Routes and domain functions stay free of status codes.

```ts
// errors.ts
export type ErrorCode = "not_found" | "forbidden" | "conflict" | "invalid";

export class AppError extends Error {
  constructor(
    readonly code: ErrorCode,
    message: string,
  ) {
    super(message);
  }
}

// app.ts — the one place errors become HTTP
const STATUS: Record<ErrorCode, ContentfulStatusCode> = {
  not_found: 404,
  forbidden: 403,
  conflict: 409,
  invalid: 400,
};

app.onError((err, c) => {
  if (err instanceof AppError) {
    return c.json({ error: err.code, message: err.message }, STATUS[err.code]);
  }
  console.error(err);
  return c.json({ error: "internal" }, 500);
});

// domain code — no HTTP knowledge
if (!order) throw new AppError("not_found", `Order ${id} not found`);
```

- Keep the code set coarse — around five codes. A new code must map to a distinct HTTP status or distinct client handling; otherwise reuse an existing one.
- Unknown errors log server-side and return a generic 500 body — internals stay out of responses.
- When a caller must recover from a specific failure mid-flow, return a discriminated union from that function instead of throwing; throwing is for failures that end the request.
- With tRPC, apply the same principle: map `AppError` to `TRPCError` in one place (error formatter or a shared wrapper).

## Database

- Drizzle with schema in code; migrations generated and checked in.
- Shape responses with destructuring, not mapper layers: pick the fields the endpoint returns (`const { id, name, email } = user`, or `columns:` in the query) — renames stay compiler-checked one-line edits. On tables holding sensitive columns, pick rather than rest-omit: `...rest` is a deny-list, so a later-added column silently flows to clients, while a pick is an allow-list. Public contracts still get a real mapping to the promised shape.
- Wrap multi-write invariants in a transaction.
- Derived values are computed (generated columns, views, defaults, or at read time), not written as second copies. When denormalizing for performance, one code path owns the write.

## Before finishing

Re-read the full diff against the rules above and fix any drift — adherence decays over a long session, and the diff is where it shows.
