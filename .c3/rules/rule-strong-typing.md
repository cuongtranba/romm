---
id: rule-strong-typing
c3-seal: 1e5f0f028e0da7aef696bb8a7f1ff65d30edd10e071344aaf3feb21fba6be393
title: strong-typing
type: rule
goal: Keep both backend and frontend strongly typed end-to-end so refactors are mechanically checkable. No escape types (`Any`/`interface{}`/untyped dicts in Python; `any`/un-narrowed `unknown` in TypeScript) outside narrowly justified system boundaries.
---

## Goal

Keep both backend and frontend strongly typed end-to-end so refactors are mechanically checkable. No escape types (`Any`/`interface{}`/untyped dicts in Python; `any`/un-narrowed `unknown` in TypeScript) outside narrowly justified system boundaries.

## Rule

All exported symbols, function signatures, and stored fields must declare a concrete type; `typing.Any` in Python and `any` in TypeScript are forbidden in committed code unless paired with an inline justification comment.

## Golden Example

Python (`backend/endpoints/heartbeat.py` style):

```python
# REQUIRED: explicit return type
async def heartbeat() -> dict[str, str]:
    # REQUIRED: typed local
    payload: dict[str, str] = {"status": "ok"}
    return payload
```

TypeScript (`frontend/src/stores/auth.ts` style):

```typescript
// REQUIRED: typed state
interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
}

// REQUIRED: typed return
export function isAdmin(state: AuthState): boolean {
  return state.user?.role === "admin";
}
```

## Not This

| Anti-Pattern | Correct | Why Wrong Here |
| --- | --- | --- |
| def handle(payload: dict) -> Any: | def handle(payload: PayloadModel) -> ResponseModel: | Untyped dicts and Any defeat mypy and the OpenAPI generator |
| const data: any = await response.json(); | const data: RomSchema = await response.json(); | any propagates and breaks vue-tsc; use the generated schema |
| interface Foo { meta: object } | interface Foo { meta: RomMetadata } | object is opaque; downstream code cannot navigate it |

## Scope

Applies to all committed `.py` and `.ts`/`.vue` files in `backend/` and `frontend/`. Test fixtures and generated code (`frontend/src/__generated__/`) are exempt.

## Override

When integrating an external library that genuinely returns untyped JSON, narrow it at the boundary (e.g. `cast(SgdbResponse, payload)` after schema validation) and reference the upstream limitation in a comment.
