## [WIP] Effective Unit Testing

**Unit Testing** let's us write and run automated tests against our code at a granular level. We know that automated tests are important because they provide us _quick feedback_ regarding the correctness of our code as opposed to manual testing. In this topic, I will be sharing some of the techniques that I've learned in creating effective unit tests.

### Use business language
When writing tests, use business language. It makes the tests _"human"_ readable and motivates us to describe our code closer to business requirements:

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
When you are testing state with multiple assertions, use the setup and teardown method. This will reduce unnecessary duplication of code and make the test more readable.

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

        expect(@wallet.logs[0]).to eq(
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
        expect(@wallet.logs[0]).to eq(
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

### Favor testing behavior over implementation
When emphasis is placed on the implementation rather than the behavior, tests tend to become fragile. This is because <ins>implementations will often change</ins> as business requirements change. On the other hand, the underlying <ins>behaviors will usually remain consistent</ins>. If you find yourself in a situation where you have to change your tests for every iteration of your code, then the tests might not be providing value.

To simplify, here's how we differentiate the two:

- **Behavior testing** means we're testing the outcome.
- **Implementation testing** means we're testing how the outcome was produced.

❌ Bad

```rb
context 'when processing the service' do
  context 'and account does not exists' do
    before do
      allow(AccountRepository).to receive(:exists?).and_return(false)
      activate_account_service
      rescue described_class::NotFoundError # Forced error-catching to allow the expectation below
    end

    it 'calls the repository with the right parameters' do
      # Regardless of how the service verifies the existence of the account, the behavior should
      #
      # be consistent when the account is not found, which is to throw a "not found" error.
      #
      # Yet, here we insist on testing the implementation for this behavior.
      expect(AccountRepository).to have_received(:exists?).with(account_id)
    end
  end
end
```

✔ Good

```rb
context 'when processing the service' do
  context 'and account does not exists' do
    before { allow(AccountRepository).to receive(:exists?).and_return(false) }

    it 'notifies that the account does not exists' do
      expect { activate_account_service }.to raise_error(described_class::NotFoundError)
    end
  end
end
```

