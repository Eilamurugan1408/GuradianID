## Plan: OpenAI Chatbot Integration Rollout

Integrate OpenAI as the primary LLM provider in the existing chatbot backend while preserving Groq as automatic fallback, starting with the current non-streaming tourist chat path for a safe rollout. Reuse the existing provider call boundaries in backend/chatbot_api.py, add provider routing + observability, and keep frontend API contracts unchanged in phase 1.

**Steps**
1. Phase 1 - Provider configuration and routing foundation
2. Add environment configuration in backend loading logic: OPENAI_API_KEY, OPENAI_MODEL (default gpt-4.1-mini), LLM_PROVIDER (default openai), LLM_FALLBACK_PROVIDER (default groq), and optional OPENAI_TIMEOUT_SECONDS. *blocks steps 3-6*
3. Add provider-agnostic routing helpers in backend/chatbot_api.py (single entry points for sync chat, stream chat, and rewrite) that dispatch to OpenAI first and fallback to Groq on retryable errors (429/5xx/network timeout). *depends on 2*
4. Keep existing response schema unchanged for /chatbot/chat so frontend continues working without changes. *depends on 3*

5. Phase 2 - OpenAI provider implementation
6. Implement OpenAI non-streaming chat function mirroring existing Groq signature and return semantics. Use strict error normalization so fallback logic can detect retryable vs non-retryable failures. *depends on 3*
7. Implement OpenAI rewrite function used by itinerary polishing path, with same input/output contract as current rewrite_with_groq. *depends on 6*
8. Wire primary paths to provider router:
9. - generate_enhanced_response
10. - enhanced_travel_planning
11. - generate_itinerary_plan_response
12. Keep existing Groq functions intact as fallback backend. *depends on 6-7*

13. Phase 3 - Streaming compatibility (backend-ready, frontend deferred)
14. Implement OpenAI streaming function in backend using current SSE behavior so /chatbot/chat/stream remains provider-compatible even if UI is still non-streaming. *parallel with phase 2 after step 3*
15. Route /chatbot/chat/stream through provider router with OpenAI primary and Groq fallback. *depends on 14*

16. Phase 4 - Reliability and observability
17. Add structured provider telemetry logs (provider used, fallback invoked, latency, status category) without logging API keys or full prompts. *parallel with steps 8-15*
18. Add concise user-safe fallback messaging when both providers fail, preserving current API response shape. *depends on 8,15*
19. Optional: add lightweight /chatbot/provider-health endpoint for ops visibility (active provider + key presence checks only). *optional*

20. Phase 5 - Frontend and rollout
21. Keep frontend/components/tourist/chatbot.tsx unchanged initially (contract-compatible).
22. Optional UX improvement: append a subtle metadata note in context_data indicating fallback occurred; frontend can surface small banner later.
23. Add documentation updates for env setup and rollout steps in docs/INTEGRATION_GUIDE.md (or docs/README_IMPLEMENTATION.md if preferred).

**Relevant files**
- d:/TOURIST-PORTAL-feature-two-way-chat-call/TOURIST-PORTAL-feature-two-way-chat-call/backend/chatbot_api.py — add OpenAI provider functions, routing helpers, fallback logic, and endpoint wiring reuse.
- d:/TOURIST-PORTAL-feature-two-way-chat-call/TOURIST-PORTAL-feature-two-way-chat-call/backend/main.py — no contract change expected; verify router include remains stable.
- d:/TOURIST-PORTAL-feature-two-way-chat-call/TOURIST-PORTAL-feature-two-way-chat-call/backend/.env.example (or backend/.env docs equivalent) — document new provider env vars and defaults.
- d:/TOURIST-PORTAL-feature-two-way-chat-call/TOURIST-PORTAL-feature-two-way-chat-call/frontend/components/tourist/chatbot.tsx — no required phase-1 code change; optional fallback indicator handling later.
- d:/TOURIST-PORTAL-feature-two-way-chat-call/TOURIST-PORTAL-feature-two-way-chat-call/docs/INTEGRATION_GUIDE.md — rollout/config documentation updates.

**Verification**
1. Configuration checks
2. - Start backend with OPENAI_API_KEY present and GROQ_API_KEY present; ensure boot succeeds and /chatbot/health is healthy.
3. Functional checks
4. - POST /chatbot/chat with normal prompt returns ChatResponse and session_id.
5. - Trigger simulated OpenAI failure (invalid key or temporary provider override) and verify automatic Groq fallback still returns response.
6. - POST /chatbot/chat/travel-plan returns itinerary with rewritten output.
7. Streaming readiness checks
8. - POST /chatbot/chat/stream returns incremental SSE chunks when OpenAI succeeds.
9. - Force OpenAI streaming failure and verify Groq streaming fallback path.
10. Safety checks
11. - Confirm logs include provider + fallback events without sensitive data.
12. - Confirm frontend tourist chatbot UI continues working without payload changes.

**Decisions**
- OpenAI is primary provider.
- Groq remains fallback provider.
- Rollout starts with non-streaming UX first; streaming backend support still implemented for compatibility.
- Initial OpenAI model: gpt-4.1-mini.

**Further Considerations**
1. Rate limit strategy recommendation: add exponential backoff with max 1 retry before fallback to keep latency bounded.
2. Cost guardrail recommendation: include max_tokens ceiling per endpoint profile (chat vs itinerary) to prevent surprise spend.
3. Security recommendation: keep keys server-side only; do not expose provider selection in public frontend until auth controls exist.