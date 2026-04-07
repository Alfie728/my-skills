# Interface Design For Testability

Good interfaces make tests natural:

1. Accept dependencies instead of constructing them internally.
2. Return results instead of relying on hidden side effects.
3. Keep the public surface area small.

A smaller, clearer interface usually yields fewer, better tests.
