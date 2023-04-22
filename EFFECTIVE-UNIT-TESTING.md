## [WIP] Effective Unit Testing

**Unit Testing** let's us run automated tests against our code at a granular level. We know that automated tests are important because they provide us _quick feedback_ regarding the correctness of our code as opposed to manual testing. In this topic, I will be sharing some of the techniques that I've learned to create effective unit tests.

### Use business language
When describing tests, use business language. It makes the tests _"human"_ readable thus it's easier to understand and maintain:

```rb
describe Storage do
  describe '#dispose' do

    # begin with `when` to describe scenarios
    context 'when storage has perishables' do

      # explain result in present tense
      it 'disposes the perishables' do
        storage = described_class.new([Item.new('Cabbage', :vegetable)])
        storage.dispose

        expect(storage.total).to eq(0)
      end
    end
  end
end
```
_Note: You may also check the [Gherkin](https://cucumber.io/docs/gherkin/reference/) syntax for reference._

### Setup and Teardown
When your test requires setting up data multiple times, use the setup and teardown method. This will reduce unnecessary duplication of code and make the test more readable.

❌ Bad

```rb
describe Wallet do
  describe '#withdraw' do
    context 'when user withdraws from wallet' do
      # prepares the data and also asserts the result
      it 'deducts the wallet amount' do
        wallet = described_class.new(1500)
        wallet.withdraw(500, '2023-04-23')

        expect(wallet.amount).to eq(1000)
      end

      # repeats the same task with a different assertion
      it 'logs the withdrawal' do
        wallet = described_class.new(1500)
        wallet.withdraw(200, '2022-05-24')

        expect(@wallet.transaction_history.logs[0]).to eq(
          {
            amount: 200,
            date: '2022-05-24'
          }
        )
      end
    end
  end
end
```

✔ Good

```rb
describe Wallet do
  describe '#withdraw' do
    # sets up the data
    before do
      @wallet = described_class.new(1500)
      @wallet.withdraw(500, '2023-04-23')
    end

    context 'when user withdraws from wallet' do
      # asserts using the setup data
      it 'deducts the wallet amount' do
        expect(@wallet.amount).to eq(1000)
      end

      # also uses the same setup data
      it 'logs the withdrawal' do
        expect(@wallet.transaction_history.logs[0]).to eq(
          {
            amount: 500,
            date: '2023-04-23'
          }
        )
      end
    end
  end
end
```

### Test behavior over implementation
Tests become _fragile_ when the subject being tested is the implementation rather than the behavior. This is because <ins>implementations will often change</ins> as business requirements change. On the other hand, the underlying behaviors will usually remain consistent.
