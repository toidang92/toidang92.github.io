---
layout: post
title: Ruby Best Practices
---

## Avoid allocates a lot of objects

Example:

**DON'T**

```ruby
1_000_000.times { "some string or object" }
```

**SHOULD BE**

```
my_invariant = "some string or object"
1_000_000.times { my_invariant }
```

OR

```
1_000_000.times { "some string or object".freeze }
```

## Do not use exceptions for a control flow

**DON'T**

```ruby
def with_condition
  respond_to?(:mythical_method) ? self.mythical_method : nil
end
```

**SHOULD BE**

```ruby
def with_condition
  respond_to?(:mythical_method) ? self.mythical_method : nil
end
```

## Be careful with calculation within iterators

**DON'T**

```ruby
def func(array)
  array.inject({}) { |h, e| h.merge(e => e) }
end
```

**SHOULD BE**

```ruby
def func(array)
  array.inject({}) { |h, e| h[e] = e; h }
end
```

## Avoid object creation if possible

### String concatenation

Avoid using `+=` to concatenate strings in favor of `<<` method

```ruby
str1 = "first"
str2 = "second"
str1.object_id       # => 16241320
```

**DON'T**

```ruby
str1 += str2    # str1 = str1 + str2
str1.object_id  # => 16241240, id is changed
```

**SHOULD BE**

```ruby
str1 << str2
str1.object_id  # => 16241240, id is the same
```

When you use `+=` ruby creates a temporal object which is result of `str1 + str2`. Then it overrides `str1` variable with reference to the new built object. On the other hand `<<` modifies existing one.

As a result of using += you have the next disadvantages:

* More calculation to join strings.
* Redundant string object in memory (previous value of str1), which approximates time when GC will trigger.

### Use bang! methods

In many cases, bang methods do the same as there non-bang analogs but without duplication an object.

```
array = ['a', 'b', 'c']
array.map(&:upcase)
array.map!(&:upcase)
```

***Some methods are essentially an alias for bang methods***

1. select vs. keep_if

```ruby
a = %w{ a b c d e f }
a.keep_if { |v| v =~ /[aeiou]/ }
puts a
=> ["a", "e"]
```

```ruby
a = %w{ a b c d e f }
a.select { |v| v =~ /[aeiou]/ }
puts a
=> ["a", "b", "c", "d", "e", "f"]
```

2. reject vs. delete_if

```ruby
a = [1,2,3,4,5]
a.delete_if(&:even?)
puts a
=> [1,3,5]
```

### Parallel assignment is slower

**DON'T**

```ruby
a, b = 10, 20
```

**SHOULD BE**

```ruby
a = 10
b = 20
```

## Use the methods are optimized by ruby

DON'T| SHOULD BE
---|-----
Enumerable#reverse.each | Enumerable#reverse_each
Enumerable#select.first | Enumerable#detect

[READ MORE](https://github.com/JuanitoFatas/fast-ruby)

## Be careful when using EVAL, SEND

When you use them, please make sure you can handle what will be entered.

**DON'T**

```ruby
type = params[:type] # type = "system('rm -rf ./')"
eval(type)
```

```ruby
type = params[:type] # type = delete_all
User.send(type)
```

**SHOULD BE**

```ruby
type = params[:type]
User.send(type) if ['abc', 'zyz'].include?(type)
```

## Ruby Garbage Collection

I have researched about it, so I will update later. :d

