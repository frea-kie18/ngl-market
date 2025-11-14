---

## **2️⃣ `Scripts/deploy.js.md`**
```markdown
```js
async function main() {
  const MarketFactory = await ethers.getContractFactory("MarketFactory");
  const market = await MarketFactory.deploy();
  await market.deployed();
  console.log("MarketFactory deployed to:", market.address);
}

main().catch((error) => {
  console.error(error);
  process.exit(1);
});