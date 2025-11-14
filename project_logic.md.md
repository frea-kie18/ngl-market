---

## **3️⃣ `Notes/project_logic.md`**
```markdown
# Polygon Market MVP Logic

- Market creation fee: 5 USDT
- Trading fee: 5% per bet
- Market creator earns 50% of trading fees
- Referral system: 20% of creation fee goes to referrer
- Market resolution:
  - Users submit proof (off-chain)
  - Anyone can call `resolveMarket` with outcome
  - Winners automatically receive payouts