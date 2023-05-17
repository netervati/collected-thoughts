## Ruby Magic
After working with **Ruby** for more than a year now, I can definitely say that it is deceptively easy to use. Perhaps it is due to its semantics written in descriptive English which may seem more readable to non-programmers as opposed to C-like languages. However, once we dive deeper, there are certain patterns and implementations that seem too much _"magic"_ compared to how most code are written. Whether this magic leads to better written code or not, I will be documenting them here for future reference.

### Using `.tap`

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

### Enumerators in `.each`

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
- Useful for loops that require a _"counter"_ mechanism

```rb
['Apple', 'Orange', 'Grapes'].each.with_index(1) do |fruit, idx|
  puts "#{idx}: #{fruit}"
end

# => 1: Apple
# => 2: Orange
# => 3: Grapes
```
