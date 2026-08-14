# Extra 02 · Several Next apps

WHEN: two or more **deployable** UIs (marketing site vs signed-in app vs admin) with different hosts or scale.
LOAD: this file **and** 03, 06. Not instead of them.
SCOPE: split of `frontend/` trees. MUST NOT: split because a page is slow (16).

Default remains one `frontend/` with route groups `(marketing)` / `(app)` (06). Split only when deploy, host, or trust already differs.

Each app is a full playbook tree: `app/` · `features/` · `ui/` · `lib/`. MUST NOT: an app that is only `app/` with no `lib/api.ts`.

Shared primitives: Extra 04-style package, or copy `ui/` until the second app. MUST NOT: `packages/utils`. Cookie domain: 12 + host config (04).

---

## Done

- [ ] Each deployable has `app` + `features` + `lib`
- [ ] Split is host/deploy/trust, not taste
