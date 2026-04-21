# Feature Flag System

A feature flag system answers one question: "For this user/request/context, is feature X enabled, and which variant should they see?"

What this is NOT
❌ Not a remote config service
❌ Not persistent (in-memory only)
❌ Not distributed (no sync between instances)
❌ Not a full LaunchDarkly/Unleash replacement
This is the evaluation engine only - the core that makes feature flags work. -> You’d wrap this in your own storage/APIs for production use.

### Architecture

```
┌─────────────────────────────────────────┐
│  pkg/featureflag  (Public Service API)  │  ← What consumers import
│  Coordinates store + engine             │
├─────────────────────────────────────────┤
│  internal/store   (In-Memory Store)     │  ← Holds flag definitions
├─────────────────────────────────────────┤
│  internal/engine  (Rule Engine)         │  ← Evaluates flags against context
├─────────────────────────────────────────┤
│  internal/domain  (Pure Domain Types)   │  ← Flag, Rule, Context, Result
│  internal/rules   (Rule Implementations)│  ← Concrete rule strategies
└─────────────────────────────────────────┘
```

Key architectural principle: dependencies flow downward only.
- The domain layer depends on nothing, it's pure types and interfaces.
- Rules depend only on domain types.
- The engine depends on domain types (it evaluates rules, but doesn't know which concrete rules exist).
- The store depends on domain types (it stores flags).
- The service (public API) depends on store and engine, coordinating them.

### What is Feature Flag System?

A feature flag is a software development technique that let us turn features on/off without deploying new code.

#### Why use this?
1. Safe releases
-> Deploy code to production but keep new features hidden. Turn them on when ready.
2. A/B testing
-> Roll out features to a percentage of users to test performance before full launch.
3. Kill switches
-> Instantly disable a broken feature without rolling back or redeploying.
4. User targeting
-> Enable features for specific users (e.g., premium users, internal team).

**How does it work?**

```sh
User Request
    │
    ▼
┌─────────────┐
│  Service    │  ◄── Orchestrates everything
└─────────────┘
    │
    ├──► Store (lookup flag definition)
    │
    ▼
┌─────────────┐
│   Engine    │  ◄── Iterates rules, returns first match
└─────────────┘
    │
    ▼
┌─────────────┐
│    Rules    │  ◄── Actual logic (AlwaysOn, Percentage, etc.)
└─────────────┘
    │
    ▼
EvaluationResult{Enabled: bool, Variant: string}
```
