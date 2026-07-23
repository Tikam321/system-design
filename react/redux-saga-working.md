# Redux vs Redux Saga vs Redux Toolkit — Interview Notes

## 1. Core Concepts

### Redux (the base library)
Redux is a predictable **global state container**. Any component can read from or dispatch to it without prop drilling.

Three pieces only:
1. **Action** — a plain object describing what happened: `{ type: 'FETCH_USER_REQUEST', payload: { userId: 123 } }`
2. **Reducer** — a **pure, synchronous** function: `(state, action) => newState`. It cannot make API calls or have side effects.
3. **Store** — holds the state tree and lets components subscribe/dispatch.

> **Key limitation:** Redux by itself has no way to handle asynchronous work (API calls, timers, race conditions) because reducers must stay pure. This is the gap Redux Saga fills.

### Redux Saga (the async middleware)
Saga is **not** part of the reducer flow. It's a separate middleware that listens to the *same* dispatched actions as the reducer, in parallel — it doesn't fetch from or call the reducer.

```
Component dispatches action
        ↓
   ┌────┴────┐
   ↓         ↓
Reducer    Saga (middleware)
(sync,     (async: API calls,
pure)      delays, race conditions)
```

- **Watcher saga** — listens for a specific action type (e.g. `takeLatest('FETCH_USER_REQUEST', worker)`)
- **Worker saga** — does the actual async work (API call), then **dispatches new actions** with the result
- The reducer picks up those *new* actions independently — Saga never touches state directly

**Correct flow for one API call:**
```
1. Component dispatches FETCH_USER_REQUEST
2. Reducer sees it → sets loading: true
3. Saga ALSO sees it (separately) → runs the worker generator
4. Worker calls the API (async, using yield call(...))
5. Worker dispatches FETCH_USER_SUCCESS with the result
6. Reducer sees FETCH_USER_SUCCESS → updates data, sets loading: false
```

### Redux Toolkit (RTK)
RTK is the **official, opinionated way to write Redux** — it doesn't replace Saga's job (async orchestration), it removes the boilerplate around actions/reducers.

Key APIs:
- `configureStore` — store setup with good defaults (dev tools, middleware) built in
- `createSlice` — combines action types + action creators + reducer into a single object
- `createAsyncThunk` — an alternative to Saga for simpler async flows (no generators, just async/await)
- **RTK Query** — a higher-level tool that replaces Axios + thunks + manual caching entirely, with built-in caching and cache invalidation

RTK uses **Immer** internally, so you can write "mutating" code like `state.loading = true` safely — no need to manually spread `{ ...state }`.

---

## 2. Side-by-Side: Without RTK vs With RTK (Saga kept in both)

### WITHOUT Redux Toolkit (plain Redux + Saga)

```js
// actionTypes.js
export const FETCH_USER_REQUEST = 'FETCH_USER_REQUEST';
export const FETCH_USER_SUCCESS = 'FETCH_USER_SUCCESS';
export const FETCH_USER_FAILURE = 'FETCH_USER_FAILURE';

// actions.js
export const fetchUserRequest = (userId) => ({ type: FETCH_USER_REQUEST, payload: userId });
export const fetchUserSuccess = (data) => ({ type: FETCH_USER_SUCCESS, payload: data });
export const fetchUserFailure = (error) => ({ type: FETCH_USER_FAILURE, payload: error });

// reducer.js
const initialState = { data: null, loading: false, error: null };
export default function userReducer(state = initialState, action) {
  switch (action.type) {
    case FETCH_USER_REQUEST: return { ...state, loading: true, error: null };
    case FETCH_USER_SUCCESS: return { ...state, loading: false, data: action.payload };
    case FETCH_USER_FAILURE: return { ...state, loading: false, error: action.payload };
    default: return state;
  }
}

// sagas.js
function* fetchUserSaga(action) {
  try {
    const data = yield call(axios.get, `/api/users/${action.payload}`);
    yield put(fetchUserSuccess(data));
  } catch (e) {
    yield put(fetchUserFailure(e.message));
  }
}
function* watchUser() {
  yield takeLatest(FETCH_USER_REQUEST, fetchUserSaga);
}

// store.js
const sagaMiddleware = createSagaMiddleware();
const store = createStore(rootReducer, applyMiddleware(sagaMiddleware));
sagaMiddleware.run(rootSaga);
```
**4+ separate files, ~50 lines, for a single API call.**

### WITH Redux Toolkit (Saga still integrated)

```js
// userSlice.js — actions + reducer combined, auto-generated
import { createSlice } from '@reduxjs/toolkit';

const userSlice = createSlice({
  name: 'user',
  initialState: { data: null, loading: false, error: null },
  reducers: {
    fetchUserRequest: (state) => { state.loading = true; state.error = null; },
    fetchUserSuccess: (state, action) => { state.loading = false; state.data = action.payload; },
    fetchUserFailure: (state, action) => { state.loading = false; state.error = action.payload; },
  }
});

export const { fetchUserRequest, fetchUserSuccess, fetchUserFailure } = userSlice.actions;
export default userSlice.reducer;

// sagas.js — logic stays almost identical, just imports actions from the slice
import { fetchUserRequest, fetchUserSuccess, fetchUserFailure } from './userSlice';

function* fetchUserSaga(action) {
  try {
    const data = yield call(axios.get, `/api/users/${action.payload}`);
    yield put(fetchUserSuccess(data));
  } catch (e) {
    yield put(fetchUserFailure(e.message));
  }
}
function* watchUser() {
  yield takeLatest(fetchUserRequest.type, fetchUserSaga);
}

// store.js
const sagaMiddleware = createSagaMiddleware();
const store = configureStore({
  reducer: { user: userSlice.reducer },
  middleware: (getDefault) => getDefault().concat(sagaMiddleware),
});
sagaMiddleware.run(rootSaga);
```
**2 files instead of 4+.** No separate action-type constants file — `createSlice` auto-generates action types (`user/fetchUserRequest`) and action creators.

---

## 3. Comparison Table

| Concern | Plain Redux + Saga | Redux Toolkit + Saga |
|---|---|---|
| Files needed for one API flow | actionTypes, actions, reducer, saga (4+) | slice (actions+reducer combined), saga (2) |
| Action types | Manually defined strings | Auto-generated from slice name |
| Reducer immutability | Manual spreading `{...state}` | Immer — "mutating" syntax is safe |
| Async orchestration | Saga (generators, `yield`) | Same — Saga untouched, or replace with `createAsyncThunk` |
| Caching | Manual, not built in | Manual, unless you adopt **RTK Query** |
| Boilerplate | High | Low |

> **Key interview point:** RTK and Saga are **not mutually exclusive**. RTK modernizes the action/reducer layer; Saga still owns async orchestration if you keep it. Teams only fully drop Saga when they migrate to `createAsyncThunk` or RTK Query.

---

## 4. `type` vs `interface` (TypeScript) — quick contract reference

| | `interface` | `type` |
|---|---|---|
| Object shapes | Yes | Yes |
| Extending | `extends` | `&` (intersection) |
| Unions (`"a" \| "b"`) | No | Yes |
| Declaration merging (reopening same name) | Yes | No |
| Primitives / tuples / function types | No | Yes |

**Convention:** `interface` for component props / class shapes / anything extendable. `type` for unions, utility compositions, function signatures. They overlap only for plain object shapes — everywhere else, only one of them works, so the choice is use-case driven, not a matter of preference.

**Why this matters (contracts):**
```ts
interface UserCardProps {
  user: User;
  onDelete: (id: number) => void;
}
function UserCard({ user, onDelete }: UserCardProps) { /* ... */ }
```
This gives compile-time safety (catches wrong-shape data before runtime), self-documenting code, better tooling (autocomplete/refactor), and — for REST integrations with a backend (e.g. Kotlin/Spring Boot) — immediate compile errors if the backend response shape changes and breaks the contract.

---

## 5. Modern Alternatives (2026 landscape)

### TanStack Query (formerly React Query)
Purpose-built for **server state** (data from APIs/DBs), as opposed to Redux which is built for **client state** (UI state, form inputs, modals).

```ts
const { data, isLoading, error } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
});
```
Handles loading, error, caching, background refetching, and stale-while-revalidate — all built in, with no actions/reducers/thunks required.

**2026 best-practice pattern:** split state by purpose —
- TanStack Query → server/cache state
- Context → app-wide config (theme, locale, feature flags)
- Redux Toolkit → only for genuinely complex client-side workflows

**RTK Query vs TanStack Query:** the choice is mostly about what's already in your stack. If your team is already deep in Redux/DevTools, RTK Query integrates more naturally. If you want offline/persistence support out of the box, TanStack Query has more mature plugins for that (`persistQueryClient`).

**Caution before pitching a migration in an interview:** if the team already knows Redux well, factor in the learning-curve cost and hiring pool (more engineers know Redux than TanStack) — a senior answer weighs this tradeoff rather than just chasing the newer tool.

### React 19.2 — relevant new features
- **`useEffectEvent`** — read latest props/state inside an Effect without adding them to the dependency array (removes the old `useRef` workaround for stale closures).
- **`Activity` component** — `visible`/`hidden` modes; hidden mode unmounts effects and defers updates, useful for pre-rendering inactive tabs/sections without hurting current performance.
- **Partial Pre-rendering** — pre-render static parts of the app ahead of time, resume the rest later — relevant for dashboards where some widgets load fast and others depend on slow queries.
- **Server Components / Server Functions** — render parts of the UI server-side, shipping less JS to the browser. Adoption is still moderate since it requires a bigger architectural shift than something like Suspense.

---

## 6. Sample Answer to Use in an Interview

> "We currently use Redux with Redux Saga — Saga listens to the same dispatched actions as our reducers, but handles the async side effects like API calls, then dispatches follow-up actions that the reducers pick up. It's great for complex async orchestration like race conditions or cancellation. That said, for BI/data-heavy features where caching and invalidation matter a lot, I'd evaluate introducing RTK Query or TanStack Query — they give built-in cache management, automatic refetching, and tag-based invalidation out of the box, which today we have to wire manually across actions → saga → reducer → selector. I'd weigh that against the team's familiarity with Redux and the migration cost before recommending a full switch."
