+++ 
draft = false
date = 2022-05-04T01:06:00+02:00
title = "Rails Value Objects"
description = ""
slug = "2022-05-04-rails-value-objects"
authors = ["Marcus Ilgner <mail@marcusilgner.com>"]
tags = ["ruby"]
categories = []
externalLink = ""
series = []
+++
# Value Objects in Rails

While writing yesterdays post about the usage of `Hash` vs `class`, I couldn't
help but think of another powerful tool that helps us to build a well-defined
model of our problem domain in the programming language of our choice: the
*value object*.

To understand the value of value objects (*scnr*), let's take an example:
`42`. "What is `42`?", you might be tempted to ask - and rightfully so.
In Ruby it might be an instance of `Integer`, other languages may have cause
to use a more restrictive definition like `u8` or `i16`. But still - that
doesn't help us to understand what `42` might mean.

It could be the result of having calculated `21*2`, but it might also be a
distance (in whatever unit), a price (in whatever currency), a weight and so
on.

This is where value objects come in: they help us to resolve these ambiguities.

## Hands-on activity

{{< notice tip >}}
Full source code for the examples below can be [found on Github](https://github.com/milgner/blog-examples-activities).
{{< / notice >}}

To give a concrete example, we're going to implement a model for a basic fitness tracking app. There will be activities like walking, cycling, swimming and so on - we don't care about that right now - and users can track how far they have gone during any given activity.

```ruby
class Activity < ApplicationRecord
  belongs_to :user
  attribute :distance, :integer
end
```

### First approach:

Given the model of the initial iteration above, the problem becomes clear: we can't be sure what unit the `distance` attribute is measured in. It could be meters, kilometers, yards or multiples of the Planck length 🤪.

As a first version, you could rename the attribute to `distance_meters`. Now if you're asked to show the total number of miles traveled, you'd have to do something like `current_user.activities.sum(:distance_meters) / 1609.344`. Sure, it works... but having to append `_meters` all of the time and having to know and apply the conversion factor manually is a bit ugly.

### Going the `Distance`

Instead, we're going to implement a *value object* to represent distances.

```ruby
class Distance
  attr_reader :meters

  class << self
    def from_meters(meters)
      new.tap { |d| d.instance_eval { @meters = meters } }
    end
  end

  # see the link above for the full code on Github

  def miles
    meters * 6.213712e-4
  end

  def parsecs
    meters * 3.240779e-17
  end
end
```

Given that this is Ruby, you could even monkey-patch `Numeric` :

```ruby
class Numeric
  def meters
    Distance.from_meters(self)
  end
end
```

This will allow you to write:

```ruby
flown = 400000000000000000.meters
puts "You flew the Kessel Run in #{flown.parsecs} parsecs. Impressive, but not a record!"
```

which will yield:
```
You flew the Kessel Run in 12.963116000000001 parsecs. Impressive, but not a record!
```

Constructors from other units are left as an exercise to the reader - for now, though, we'll skip implementing them to more easily see type casting at work.

### Integrating into the Attributes API

To properly benefit from this construct, we're going to have to register it with the ActiveRecord Attributes API.

```ruby
class DistanceInMetersType < ActiveRecord::Type::Value
  def cast(value)
    return value if value.is_a?(Distance)

    raise ArgumentError, "Can't cast from #{value.class}" unless value.respond_to?(:to_f)

    Distance.from_meters(value.to_f)
  end

  def serialize(value)
    raise ArgumentError unless value.is_a?(Distance)

    value.meters
  end
end

# Note that ActiveModel types have to be registered separately and have a slightly different signature here
ActiveRecord::Type.register(:distance_in_meters, DistanceInMetersType)
```

and now you can use it to output the total traveled distance in miles very comfortably:

```ruby
class Activity < ApplicationRecord
  # *snip*
  attribute :distance, :distance_in_meters
end

# And use it:
> puts "You traveled #{current_user.activities.sum(:distance).miles} miles in total! 🥳"
```

Now the beauty here is that even though `Activity.sum(:distance)` only creates a stand-alone value without a surrounding instance of `Activity`, the model still defines it as `distance_in_meters` - and ActiveRecord is smart enough to instantiate an object of the `Distance` class for us upon which we can invoke `.miles`.


## Non-Ruby excourse: typed languages

In typed languages, there is a precursor to full-blown value objects: *type aliases*. Using a type alias can help you clarify the meaning of your function arguments.

For example, given the example below

```go
package main

import "fmt"

func circumnavigations(distance_walked float32) (float32) {
        return distance_walked / 24859.734
}

func main() {
    walked := circumnavigations(1234)
    fmt.Println("I walked the earth", walked, "times")
}
```

you can facilitate understanding a lot by replacing the *magic number* with a properly-named constant and introducing a type alias.

```go
package main

import "fmt"

type miles = float32
const EARTH_CIRCUMFERENCE_IN_MILES = 24859.734

func circumnavigations(distance_walked miles) (float32) {
    return distance_walked / EARTH_CIRCUMFERENCE_IN_MILES
}

func main() {
    walked := circumnavigations(1234)
    fmt.Println("I walked the earth", walked, "times")
}
```

## A word on proportionality

Of course, these constructs would be *somewhat* overblown if you just want to log your own fitness activities in an app that might reach a couple of hundreds lines of code in total at best. 😅

On the other hand, if you're planning on writing a scalable codebase which is going to manage many different activities for thousands of users across the world, you're crossing from *programming* territory into *software engineering* - and will be glad for having built a proper model very soon.
