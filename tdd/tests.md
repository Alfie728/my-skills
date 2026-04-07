# Good And Bad Tests

## Good Tests

Integration-style tests verify observable behavior through real interfaces.

```typescript
test("user can checkout with valid cart", async () => {
  const cart = createCart();
  cart.add(product);
  const result = await checkout(cart, paymentMethod);
  expect(result.status).toBe("confirmed");
});
```

Good tests:

- Assert behavior users or callers care about
- Use public interfaces only
- Survive internal refactors
- Describe what happens, not how

## Bad Tests

Implementation-detail tests are coupled to structure instead of behavior.

```typescript
test("checkout calls paymentService.process", async () => {
  const mockPayment = jest.mock(paymentService);
  await checkout(cart, payment);
  expect(mockPayment.process).toHaveBeenCalledWith(cart.total);
});
```

Red flags:

- Mocking internal collaborators
- Testing private helpers
- Asserting call order or call counts
- Failing on refactor without behavior change
