Written here are collection of ideas and realizations that have emerged from my experiences as a software engineer.

## Over Abstraction
There are 3 main considerations we make when dealing with code:
- _Velocity_ is when we prioritize shipping the changes as quick as possible.
- _Adaptability_ is when we write extensible code to lessen the friction for code changes.
- _Performance_ is when we write optimized code.

**Abstraction** is a critical aspect of creating adaptable code. However, excessive use of abstraction can negatively impact velocity and performance.

### Premature Abstraction
Often times we make abstractions because we assume that the implementation will be reused in the future. In some cases, we also think that abstraction is necessary because we need to _abide_ to certain principles (e.g. SRP). For example:

Let's say we're creating a class called `Jewellery` with the properties `carat` and `grams`. `Jewellery` should have a method that calculates its price based on its properties and the price of gold per gram. First, it finds the grams of pure gold using the computation `(carat / 24) * grams`. Then, it multiplies the result with the price of gold per gram. Finally, it rounds-off the result to a whole number. Below is a straightforward implementation:
```js
class Jewellery {
  constructor(grams, carat) {
    this.grams = grams;
    this.carat = carat;
  }
  calculatePrice(priceOfGold) {
    const gramsOfPureGold = (this.carat / 24) * this.grams;
    const price = Math.round(gramsOfPureGold * priceOfGold);

    return price;
  }
}

const necklace = new Jewellery(1.5, 18);
necklace.calculatePrice(1200); // 1350
```
This looks simple. However, since the topic is premature abstraction, we want to complicate things immediately.

We may want to obtain the jewellery's grams of pure gold in the future. To cater this, we can extract the computation from `#calculatePrice` and move it to another method:

```js
class Jewellery {
  constructor(grams, carat) {
    this.grams = grams;
    this.carat = carat;
  }
  getGramsOfPureGold() {
    return (this.carat / 24) * this.grams;
  }
  calculatePrice(priceOfGold) {
    const gramsOfPureGold = this.getGramsOfPureGold();
    const price = Math.round(gramsOfPureGold * priceOfGold);

    return price;
  }
}

const necklace = new Jewellery(1.5, 18);
necklace.getGramsOfPureGold(); // 1.125
```
This is better. But wait!

Should the computation for finding the carat's gold content belong to `Jewellery`? Perhaps we can move this to a separate function:
```js
function getGoldContent(carat) {
  return carat / 24;
}

class Jewellery {
  constructor(grams, carat) {
    this.grams = grams;
    this.carat = carat;
  }
  getGramsOfPureGold() {
    return getGoldContent(this.carat) * this.grams;
  }
  calculatePrice(priceOfGold) {
    const gramsOfPureGold = this.getGramsOfPureGold();
    const price = Math.round(gramsOfPureGold * priceOfGold);

    return price;
  }
}
```
We also don't want `24` to be a _magic number_. Perhaps we can declare it as a static property named `PURE_GOLD_CARAT` and encapsulate it with `getGoldContent` to make them more reusable:
```js
class Carat {
  static PURE_GOLD_CARAT = 24;

  static getGoldContent(carat) {
    return carat / this.PURE_GOLD_CARAT;
  }
}

class Jewellery {
  constructor(grams, carat) {
    this.grams = grams;
    this.carat = carat;
  }
  getGramsOfPureGold() {
    return Carat.getGoldContent(this.carat) * this.grams;
  }
  calculatePrice(priceOfGold) {
    const gramsOfPureGold = this.getGramsOfPureGold();
    const price = Math.round(gramsOfPureGold * priceOfGold);

    return price;
  }
}
```
Perfect!

This is what **premature abstraction** looks like. We thought of a potential scenario in the future and made an abstraction on the basis that we will encounter it eventually. We also thought that the computation for finding the carat's gold content belongs to a different _domain_. Hence, we made another abstraction.

There are <ins>2 important takeaways</ins> here:
- Unless we know the full roadmap of features, _we cannot predict every scenario_.
- Principles and any system of ideas serve as guides to help us tackle problems in programming. However, they should not be treated as the rule.

## One Abstraction to Rule them all
Another form of over abstraction is when we have reusable logic that attempts to account for every usage of that unit of code. In most cases, we want reusable logic to have a centralized domain to make our code modular. However, there are instances when a _consumer_ requires a different behavior from that logic and we attempt to rework it to fit every requirement. Here's an example:

