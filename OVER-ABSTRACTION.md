## Over-abstraction
There are 3 main considerations we make when dealing with code:
- _Velocity_ is when we prioritize shipping the changes as quick as possible.
- _Adaptability_ is when we write extensible code to lessen the friction for code changes.
- _Performance_ is when we write optimized code.

**Abstraction** is a critical aspect of creating adaptable code. However, excessive use of abstraction can negatively impact velocity and performance.

### Hasty Abstraction
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
This looks simple. However, since the topic is hasty abstraction, we want to complicate things immediately.

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

This is what **hasty abstraction** looks like. We thought of a potential scenario in the future and made an abstraction on the basis that we will encounter it eventually. We also thought that the computation for finding the carat's gold content belongs to a different _domain_. Hence, we made another abstraction.

There are <ins>2 important takeaways</ins> here:
- Unless we know the full roadmap of features, _we cannot predict every scenario_.
- Principles and any system of ideas serve as guides to help us tackle problems in programming. However, they should not be treated as the rule.

### One Abstraction for Everything
Another form of over-abstraction is when reusable code tries to handle every possible usage. Ideally, when there are identical implementations, we make the code reusable. However, sometimes a consumer requires a different behavior from that code that may result to massive rework to maintain backwards compatibility. Rather than keeping it simple by localizing the implementation to the specific consumer, we try to modify the reusable code to fit every requirement. For example:

Let's say we have a generic function for computing rent and service cost in a beach resort. The computation is based on the number of service days and the type of availed service.
```js
function findDays(from, to) {
  const dateDifference = new Date(to).getTime() - new Date(from).getTime();
  const days = dateDifference / (1000 * 3600 * 24) + 1;

  return days;
};

function computeServiceCost(service, priceTable) {
  const days = findDays(service.dateFrom, service.dateTo);

  return days * priceTable[service.type];
};
```
Here `computeServiceCost` takes in a `service` object and a `priceTable` object. It calculates the total cost of the service requested by the client by first calling `findDays` to get the number of days of service. Next, it multiplies the days by the rate of the service type based on `priceTable`. Finally, the function returns the total service cost.

Below is how we use the function when computing costs for renting rooms and meal services:
```js
const ROOM_RATES = {
  SINGLE: 1000,
  DOUBLE: 1750,
  SUITE: 2500,
};

const roomCost = computeServiceCost(
  {
    type: 'SINGLE',
    dateFrom: '2023-01-09',
    dateTo: '2023-01-13',
  },
  ROOM_RATES
);

const MEAL_SERVICE_RATES = {
  1ML: 150,
  2ML: 350,
  3ML: 500,
};

const mealServiceCost = computeServiceCost(
  {
    type: '3ML',
    dateFrom: '2023-01-09',
    dateTo: '2023-01-13',
  },
  MEAL_SERVICE_RATES
);

console.log('Room cost:', roomCost); // 5000
console.log('Meal service cost', mealServiceCost); // 2500
```
As demonstrated, we can add more services easily since `computeServiceCost` is modular and reusable. However, let's say we're adding a new service for renting kayaks, but this time, the rate is computed by _hours_. Since we want to maintain _one code_ for computing service costs, we can rework the function to accept another argument that defines the _rate scheme_ (`daily` or `hourly`) so it can either compute by service days or by service hours. Observe:
```js
function findDays(from, to) {
  const dateDifference = new Date(to).getTime() - new Date(from).getTime();
  const days = dateDifference / (1000 * 3600 * 24) + 1;

  return days;
};

function findHours(from, to) {
  const dateDifference = new Date(to).getTime() - new Date(from).getTime();
  const hours = Math.abs(Math.round(dateDifference / 1000 / (60 * 60)));

  return hours;
};

function computeServiceCost(service, priceTable, rateScheme = 'daily') {
  let time;

  if (rateScheme === 'daily') {
    time = findDays(service.dateFrom, service.dateTo);
  } else if (rateScheme === 'hourly') {
    time = findHours(service.dateFrom, service.dateTo);
  } else {
    throw new Error(`Invalid rate scheme passed: ${rateScheme}`);
  }

  return time * priceTable[service.type];
};
```
Now, the function seems more complicated but I think the abstraction here is considerable. We have one code to maintain and it caters both service days and service hours. However, what happens if we introduce more complexity?

Let's say a new service is being added again for renting function rooms and is also rated hourly. This time the rate varies depending on whether the place is rented during day or night to account for electricity costs. How do we integrate this in `computeServiceCost`?

Well, maybe we can start by adding another rate scheme called `roundTheClock`? And maybe it calls the function `findHoursRoundTheClock`?
```js
function computeServiceCost(service, priceTable, rateScheme = 'daily') {
  let time;

  if (rateScheme === 'daily') {
    time = findDays(service.dateFrom, service.dateTo);
  } else if (rateScheme === 'hourly') {
    time = findHours(service.dateFrom, service.dateTo);
  } else if (rateScheme === 'roundTheClock') {
    time = findHoursRoundTheClock(service.dateFrom, service.dateTo);
  } else {
    throw new Error(`Invalid rate scheme passed: ${rateScheme}`);
  }

  return time * priceTable[service.type];
};
```
Since daily rates differ from nightly rates, we can't assign the result to `time` as `time * priceTable[service.type]` may not be the right equation to compute for the service cost. We might also need to return a different value from `findHoursRoundTheClock`. One option is to return an object that has the properties `day` and `night` which indicates the rent hours for both times:
```js
function computeServiceCost(service, priceTable, rateScheme = 'daily') {
  switch(rateScheme) {
    case 'daily':
      const time = findDays(service.dateFrom, service.dateTo);

      return time * priceTable[service.type];

    case 'hourly':
      const time = findHours(service.dateFrom, service.dateTo);

      return time * priceTable[service.type];

    case 'roundTheClock':
      const result = findHoursRoundTheClock(service.dateFrom, service.dateTo);
      const day = result.day * priceTable[service.type].data;
      const night = result.night * priceTable[service.type].night;

      return day + night;

    default:
      throw new Error(`Invalid rate scheme passed: ${rateScheme}`);
  }
};
```
This way we can structure `priceTable` in this manner:
```js
const FUNCTION_ROOM_RATE = {
  'FR-A': {
    day: 6500,
    night: 8000,
  },
};
```
Now, let's take a moment to ask:

_"Was all of this necessary?"_

Could we not have simplified the code by creating separate functions that would cater specifically to each rate scheme? If the goal here is to keep our code modular, then perhaps a better approach would be moving the functions in a separate file that is in module structure:
```js
// serviceCostComputer.js

export function computeByDays() { ... }
export function computeByHours() { ... }
export function computeRoundTheClock() { ... }
```
By doing so, we can maintain a separation of concerns and avoid tightly coupling the functions together, which can make the code more difficult to understand and maintain in the long run. Before we integrate new logic to reusable code, we should always consider if we're actually improving the reusability of the code or if we're simply _encapsulating logic_.

