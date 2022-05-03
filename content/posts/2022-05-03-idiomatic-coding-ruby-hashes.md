+++ 
draft = false
date = 2022-05-03T08:48:10+02:00
title = "Idiomatic Coding: Ruby Hash (& Python Dict)"
description = ""
slug = "2022-05-03-idiomatic-coding-ruby-hashes"
authors = ["Marcus Ilgner <mail@marcusilgner.com>"]
tags = ["ruby"]
categories = []
externalLink = ""
series = []
+++
# Idiomatic Ruby: `Hash` vs `class`

It is no secret that I'm a big fan of typed languages. To me, an expressive type system really helps to convey the mental model of the application to the developers maintaining the code base.

As such, I am often dismayed when reading code from untyped interpreted languages. Sure, there are classes in Javascript, Ruby and Python. But very often I find that they're eschewed in favour of just bundling a list of attributes together in an untyped `Object`, `Hash` or `Dict` respectively.

Luckily, there's an easy way to determine whether you should create a `class` or use a `Hash`: if all the keys and values are of the same type (respectively) and can be looked at without requiring the presence of specifically-named keys, a `Hash` is the way to go.
If, however, you find that you'll want to access the special property `hash[:my_value]` and then API users are expected to know that it contains an `Integer`, be sure to create a class for it instead of using a `Hash`.

The same principle can be applied to Javascript `Object`s and Python `Dict`s. Following, let's focus a bit on Ruby, though.

### Vanilla Ruby: the `Struct`

Using [`Struct`](https://ruby-doc.org/core-3.0.0/Struct.html) allows you to quickly create a class with a bit of convenience functionality.
Its instances get a `[]` method for drop-in compatibility with Hashes, but you're nonetheless able to give it a well-defined structure.
It is also possible to pass a block to its constructor and define some behaviour.

Although I'd like to argue that if you had needed more than a container, a regular `class` would have been in the picture from the start.


### Ruby on Rails: `ActiveModel::Model`

If you're working on a Rails-based project, there's an even better approach.
[`ActiveModel::Model`](https://api.rubyonrails.org/classes/ActiveModel/Model.html) is a [`Concern`](https://api.rubyonrails.org/classes/ActiveSupport/Concern.html) that you can mix in to your `class` to not only get a convenient constructor but also support for validations.
For it to unfold its full power, though, it's best to combine it with [`ActiveModel::Attributes`](https://api.rubyonrails.org/classes/ActiveModel/Attributes/ClassMethods.html).

Let's enhance the example from the API documentation and use both `ActiveModel::Model` and `ActiveModel::Attributes`:

```ruby
class Person
  include ActiveModel::Model
  include ActiveModel::Attributes

  attribute :name, :string
  attribute :age, :integer
end
```

Now you'll have a constructor with automatic type cast to both `integer` and `string`:

```ruby
> person = Person.new({age: '23', name: 42 })
> person.name
=> "42"
> person.age
=> 23
```

This can be especially valuable if you want to use value objects with your model.
You could, for example come up with a `DateRange` value object to express a `Range<Date>` that represents this persons next vacation - but that's a topic for another day.

### Documenting your code

The biggest difference between using a dedicated model class VS a `Hash` comes when you add documentation to your methods and use an IDE that understands them.

```ruby
# @param person [Person] the person to greet
def birthday_greetings(person)
  puts "Happy #{person.age + 1}. birthday, #{person.name}"
end
```

looks a lot nicer than

```ruby
# @param person [Hash] the person to greet; should have keys `:name` (String) and `:age` (Integer)
def birthday_greetings(person)
  puts "Happy #{person[:age]+1}. birthday, #{person[:name]}}"
end
```
