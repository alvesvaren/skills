---
name: backend-best-practices
description: Server-side conventions for Node/Hono APIs — few layers, Zod-validated route input, typed AppError mapped to HTTP in one onError, Drizzle for data, tRPC/Hono RPC for internal contracts and OpenAPI for public ones. Use when writing or refactoring API routes, server logic, database access, or endpoint design.
---

# Backend best practices

Conventions for server-side TypeScript. [typescript-best-practices](../typescript-best-practices/SKILL.md) applies in full; this covers what is specific to APIs.

**Stack defaults:** Hono, Drizzle, Zod.

## Layers

Fewer layers is the design goal: route → domain function → db is usually the whole stack. Add a layer (service class, repository interface, DTO mapper) only when a second consumer genuinely needs it.

## Contract

- Internal or single-consumer APIs: end-to-end typed — tRPC or Hono RPC client — so types flow and iteration stays fast.
- Public or multi-consumer APIs: OpenAPI (`@hono/zod-openapi`) so consumers get a spec; accept the slower iteration as the price of a published contract.

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
- Table shapes stay out of API responses: select or map to the shape the endpoint promises — the response is a contract, the table is not.
- Wrap multi-write invariants in a transaction.
