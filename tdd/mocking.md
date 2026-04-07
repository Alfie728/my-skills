# When To Mock

Mock at system boundaries only:

- External APIs
- Time and randomness
- Filesystem access when isolation matters
- Databases only when a realistic local substitute is not practical

Do not mock:

- Your own modules
- Internal collaborators
- Code you control

Prefer dependency injection so boundary dependencies can be replaced cleanly in tests.
