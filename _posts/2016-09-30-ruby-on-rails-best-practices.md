---
layout: post
title: Ruby and Ruby on Rails Best Practices
---

Here is my collection for Ruby performance tricks

## Do You Need to Optimize?

The first rule of optimization is, “Don’t do it.” Of course, experienced developers still optimize from time to time because “don’t do it” is really shorthand for several warnings:

1. Optimization may not be necessary.
2. Optimization may not be feasible and will waste your time.
3. You are going to be tempted to optimize the wrong thing.
4. Even successful optimization can make your code harder to understand.

## Set Up Metrics

Pink and plaid do not go together, so if you get dressed in the dark you may surprise your coworkers when you walk into the office. Similarly, you can’t optimize code in the dark: you need numbers to show that you are making progress. It’s time to set up metrics.

## Ruby Tricks

### Avoid allocates a lot of objects

Example:

```ruby
1_000_000.times { "some string or object" }
```

This is better.

```
my_invariant = "some string or object"
1_000_000.times { my_invariant }
```

OR

```
1_000_000.times { "some string or object".freeze }
```

### Do not use exceptions for a control flow

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

### Be careful with calculation within iterators

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

### Avoid object creation if possible

#### String concatenation

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

#### Use bang! methods

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

#### Parallel assignment is slower

**DON'T**

```ruby
a, b = 10, 20
```

**SHOULD BE**

```ruby
a = 10
b = 20
```

### Use the methods are optimized by ruby

DON'T| SHOULD BE
---|-----
Enumerable#reverse.each | Enumerable#reverse_each
Enumerable#select.first | Enumerable#detect

[READ MORE](https://github.com/JuanitoFatas/fast-ruby)

### Eval is evil

**Sources**:

1. http://greyblake.com/blog/2012/09/02/ruby-perfomance-tricks/

## Ruby on Rails Tricks

### Unused code should be removed

### Limit amount of data in a controller method

Thin controllers are easy to test and have a good performance profile because there’s some overhead involved in passing the controller instance variable around

### Dry (don’t repeat yourself)

**DON'T**

```ruby
def published?
  state == 'published'
end

def draft?
  state == 'draft'
end

def spam?
  state == 'spam'
end
```

**SHOULD BE**

```
STATES = ['draft', 'published', 'spam']

STATES.each do |state_name|
  define_method "#{state_name}?" do
    state == state_name
  end
end
```

In above case, we can use the enum instead of this.

### Eager loading

Eager loading is a way to solve the classic N + 1 query performance problem caused by inefficient use of child objects.

Let’s look at the following code. It will fetch zip of 10 users.

**DON'T**

```ruby
users = User.all(:limit => 10)
  users.each do |user|
  puts user.address.zip
end
```

**SHOULD BE**

```ruby
users = User.includes(:address).limit(10)

users.each do |user|
	puts user.address.zip
end
```

### Use index

Database indexing is one of the simplest ways to improve database performance. The insert operation will become slower but will boost up fetching data which is more frequently used in web application.

### Avoid dynamism

Although find\_by and find\_by\_xyz dynamic methods are cool, the are also kind of slow because each one needs to run through method_missing and parse the filename against the list of columns in database table.

### Caching

This is the purest way to speed up a rails application.

### Image spriting

In websites, a significant times are consumed for loading large number of images. One way of minimizing is to sprite your images. This will reduce the number of images to be served significantly.

### Use CDN

CDN (*content delivery network*) is an interconnected system of computers on the Internet that provides Web content rapidly to numerous users by duplicating the content on multiple servers and directing the content to users based on proximity.

When concurrent users will come to your site, using CDN rather than serving asset (like image, javascript, stylesheets) from your server will boost up performance.

### Minify and gzip stylesheets and javascript

This is the last point, but an important one. You can reduce size of the stylesheets and javascript significantly by Minifying it and serve as GZip format. It will improve the performance significantly by reducing request/response time.

### Specify which columns you need if possible

If you need to get the only email of user

```ruby
User.limit(100).map(&:email)
```

Normal

```ruby
User.select(:email).limit(100).map(&:email)
```

Better

```ruby
User.limit(100).pluck(:email)
```

Helpfully, if you just need a few values, you can use ActiveRecord#pluck to avoid instantiating ActiveRecord objects.

### Use background jobs

We should use the background jobs for:

* The task need to spend a longer time to complete than the normal request.
* The task need not return an immediate response to the user.

### Careful with SQL Injection

Example:

```ruby
params[:id] = "1 = 1"
User.find_by params[:id]
```

```ruby
Query
SELECT "users".* FROM "users" WHERE (admin = 't') LIMIT 1
Result
#<User id: 30, name: "Admin", password: "supersecretpass", age: 25, admin: true, created_at: "2016-01-19 16:12:11", updated_at: "2016-01-19 16:12:11">
```

[READ MORE](http://rails-sqli.org/)

## Ruby Garbage Collection

I have researched about it, and I will update later. :d
