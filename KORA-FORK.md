# Kora Mem0 package fork

This repository is a minimal package-level fork of `mem0ai@3.1.7`.

- Upstream repository: <https://github.com/mem0ai/mem0>
- Upstream package commit: `dc82354e143c2581d505d581a00286d6ef8c3605`
- Upstream release: `ts-v3.1.7`
- Upstream license: Apache-2.0
- Original package integrity: `sha512-dKiBFw6H8xaDfmkCrrsL56AkG+ynnXpHPE0QL3ePPvXi+djeWJU7HxU10RifLsRslHqJwasTCo/gqN4WcX0M1Q==`

## Deliberate delta

The fork adds three general public lifecycle controls that Kora requires:

1. `MemoryConfig.enableEntityMemory`, defaulting to `true` for upstream
   compatibility. When `false`, add, search, update, and delete do not create
   or access entity storage.
2. `Memory.close()`, which deterministically closes the initialized entity,
   vector, and history stores once each and reports the first cleanup error
   after attempting every owned resource.
3. `Memory.forget(memoryId)`, which idempotently erases the serving vector,
   entity links, and all history rows for one memory. Unlike `delete`, it
   intentionally retains no reconstructable deletion history.
4. `Memory.deleteAll(scope)` applies the same irreversible forgetting
   semantics to every Memory in the requested scope. Ordinary single-record
   `delete` retains upstream history behavior.
5. `LLMFactory` accepts a `custom` provider whose public `config.client`
   implements Mem0's exported `LLM` contract. This lets an application reuse
   its authenticated model transport without an OpenAI-compatible proxy or an
   additional provider credential.

The custom provider validates both required `LLM` methods before returning the
client. The in-memory SQLite vector store now exposes `close()` so the top-level
lifecycle contract can release that owned connection. History providers expose
the internal erase operation consumed by the public `forget` method. No
extraction, embedding, search, ordinary single-record delete/history, expiration, provider,
or scoring behavior is otherwise changed.

The package is generated upstream output. Source maps were removed because
their upstream mappings would be stale after this reviewed package-level
change. Kora pins an exact commit and qualifies both the public surface and
real SQLite lifecycle before use.
