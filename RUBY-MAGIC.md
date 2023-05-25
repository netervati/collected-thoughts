## Ruby Magic
After working with **Ruby** for more than a year now, I can definitely say that it is deceptively easy to use. Perhaps it is due to its semantics written in descriptive English which may seem more readable to non-programmers as opposed to C-like languages. However, once we dive deeper, there are certain patterns and implementations that seem too much _"magic"_ compared to how most code are written. Whether this magic leads to better written code or not, I will be documenting them here for future reference.

### Using `.tap`

Provides a more terse way to handle performing operations on an object and
returning the object from a method or assigning it to another variable.

❌ Traditional way

```rb
def create_new_officer(first_name, last_name)
  officer = Officer.new
  officer.first_name = first_name
  officer.last_name = last_name
  officer.save!

  officer
end

create_new_officer('John', 'Doe') # returns newly created officer
```

✔ With `.tap`

```rb
def create_new_officer(first_name, last_name)
  Officer.new.tap do |officer|
    officer.first_name = first_name
    officer.last_name = last_name
    officer.save!
  end
end

create_new_officer('John', 'Doe') # returns newly created officer
```

### Method syntax sugars

❓ Predicate methods
- Returns a boolean value
- Useful when evaluating a value or an object's property

```rb
class Person
  def initialize(full_name, age, citizenship)
    @age = age
    @citizenship = citizenship
    @full_name = full_name
  end

  def legal_age?
    @age >= 18
  end
end

person = Person.new('John Doe', 20, 'PHL')
person.legal_age? # true
```

❗ Dangerous methods
- Methods that modify the object's properties
- Methods that can throw an exception

```rb
class Fee
  DEFAULT_FEE = 2.5

  def initialize
    @amount = DEFAULT_FEE
  end

  def amount=(value)
    @amount = value
  end

  def reset!
    @amount = DEFAULT_FEE
  end

  def validate!
    raise ArgumentError, 'invalid amount set' unless @amount.positive?
  end
end

fee = Fee.new
fee.amount = 0
fee.validate! # Raises ArgumentError

fee = Fee.new
fee.amount = 6
fee.reset! # fee amount is now back to 2.5
```

### Index in `.each`

🗂Using `.each_with_index`
- Does not accept parameter
- Useful for matching two or more arrays

```rb
['Apple', 'Orange', 'Grapes'].each_with_index do |fruit, idx|
  puts "#{idx}: #{fruit}"
end

# => 0: Apple
# => 1: Orange
# => 2: Grapes
```

📇 Using `.each.with_index`
- Accepts an offset parameter (default is 0)
- Useful when a _"counter"_ is needed during iteration

```rb
['Apple', 'Orange', 'Grapes'].each.with_index(1) do |fruit, idx|
  puts "#{idx}: #{fruit}"
end

# => 1: Apple
# => 2: Orange
# => 3: Grapes
```
