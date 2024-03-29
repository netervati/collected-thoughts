## Effective Unit Testing

**Unit Testing** lets us write and run automated tests against our code at a granular level. We know that automated tests are important because they provide us _quick feedback_ regarding the correctness of our code as opposed to manual testing. In this topic, I will be sharing some of the techniques that I've learned in creating effective unit tests.

### Use business language
When writing tests, use business language. It makes the tests _"human"_ readable and motivates us to describe our test closer to business requirements:

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
When testing state with multiple assertions, use the setup and teardown technique. This will reduce unnecessary duplication of code and make the test leaner.

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

        expect(wallet.logs[0]).to eq(
          amount: 200,
          date: '2022-05-24'
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
          amount: 500,
          date: '2023-04-23'
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
  it 'calls the repository with the right parameters and returns `true`' do
    activate_account_service

    # Regardless of how the service verifies the existence of the account, the product details
    #
    # should clearly define the expected behavior of the service when the account exists or does
    #
    # not exists, which is what we're supposed to test. Yet, here we insist on testing the
    #
    # implementation for this behavior.
    expect(AccountRepository).to have_received(:exists?)
      .with(account_id)
      .and_return(true)
  end
end
```

✔ Good

```rb
context 'when processing the service' do
  context 'and account does not exists' do
    it 'notifies that the account does not exists' do
      # Since we're only testing the behavior, this allows our service to be more flexible
      #
      # on how we implement the verification for the account's existence.
      expect { activate_account_service }.to raise_error(described_class::NotFoundError)
    end
  end
end
```

### Lesser mocks and stubs
Many preach that **Unit Testing** means writing tests for the smallest unit of code, and that everything else that's not within the bounds of the code should be mocked or stubbed. However, I believe that this _absolute thinking_ hinders us from discovering tests that actually provide value.

One of the main drawbacks of faking everything is that we end up dedicating too much time trying to replicate the implementation of the dependencies. This often results in tests that are harder to read and can lead to false results.

❌ Bad

```rb
describe GetLogsService do
  let(:formatted_date_from) { '2023-02-01' }
  let(:formatted_date_to) { '2023-02-02' }
  let(:logs) { [] }

  subject do
    described_class.run(
      date_from: Time.now,
      date_to: 10.days.from_now
    )
  end

  context 'when retrieving recorded logs between filtered dates' do
    before do
      # Since we are stubbing the classes here, we are now forced to create isolated tests
      #
      # for each class. Also, should we change their return values, this test will not be able
      #
      # to detect it since we are stubbing them!
      allow(DateFormatter).to receive(:format).and_return(formatted_date_from, formatted_date_to)
      allow(LogRepository).to receive(:list).and_return(logs)
    end

    it 'returns the logs' do
      is_expected.to eq(logs)
    end
  end
end
```

✔ Good

```rb
describe GetLogsService do
  subject do
    described_class.run(
      date_from: Time.now,
      date_to: 10.days.from_now
    )
  end

  context 'when retrieving recorded logs between filtered dates' do
    it 'returns the logs' do
      # Here the test is easier to read, and at the same time, we are also
      #
      # testing `DateFormatter.format` and `LogRepository.list` in the background.
      logs = [create(:logs)]

      is_expected.to eq(logs)
    end
  end
end
```

### Sufficient Tests
One of the common metrics to measure the quality of code in a team or in a company is code coverage. The philosophy is that higher percentage means the codebase is robust. Hence, there is a standard in place to maintain or increase this metric. However, this is unreliable and oftentimes, leads to developers creating meaningless tests.

❌ Bad

```rb
describe Fees do
  describe '.matrix' do
    # There is nothing to test here because the return value is hard-coded
    #
    # and will break expectedly if there was an update on the matrix.
    it 'returns the fee matrix' do
      expect(described_class.matrix).to eq(
        local: 0.05,
        foreign: 0.08
      )
    end
  end

  # Test for `.calculate`
  # Test for 0 amount
  # etc...
end
```

✔ Good

```rb
describe Fees do
  describe '.calculate' do
    subject do
      described_class.calculate(
        amount: 10_000,
        type: :foreign
      )
    end

    context 'when transaction is international' do
      # This test provides a demonstration and evidence that the computation
      #
      # logic is correct making this valuable.
      it 'returns 8% of the amount' do
        is_expected.to eq(800)
      end
    end
  end


  # One more test for local transaction
end
```

## Conclusion
Finding the right techniques to write tests requires time and experience. With so many patterns and philosophies, it's easy to be misled into thinking that every test should be written in only one certain way. However, I believe that developing a pragmatic mindset will enable us to discover better ways to create meaningful tests without being confined to certain ideologies.


