# AGENTS Smoke Test (Questions)

Use these questions as a quick check of the agent’s labeling discipline and
code-vs-docs reasoning:

1. Which module guarantees transaction recovery after a crash, and what
   invariants apply?
2. Where is the architectural boundary between `transactions-jta` and
   `transactions-jdbc`?
3. What is the formal timeout semantics for 2PC, and in which module is it
   defined?
4. Which component “owns” connection pooling and deadlock prevention?
5. Which public modules are intended for Spring Boot 3 integration, and what
   do they guarantee?
