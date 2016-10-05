---
layout: post
title: Ruby on Rails Best Practices
---

## I. Unused code should be removed

## III. Limit amount of data in a controller method

Thin controllers are easy to test and have a good performance profile because there’s some overhead involved in passing the controller instance variable around

## III. DRY (Don't Repeat Yourself)

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

## IV. Eager loading

Eager loading is a way to solve the classic N + 1 query performance problem caused by inefficient use of child objects.

Let’s look at the following code. It will fetch zip of 10 users.

**DON'T**

```ruby
users = User.all(limit: 10)
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

## V. Use Index

Database indexing is one of the simplest ways to improve database performance. The INSERT or UPDATE operation will become slower but will boost up fetching data which is more frequently used in web application.

## VI. Avoid dynamism

Although find\_by and find\_by\_xyz dynamic methods are cool, the are also kind of slow because each one needs to run through method_missing and parse the filename against the list of columns in database table.

## VII. Caching

This is the purest way to speed up a rails application.

## VIII. Image spriting

In websites, a significant times are consumed for loading large number of images. One way of minimizing is to sprite your images. This will reduce the number of images to be served significantly.

## IX. Use CDN

CDN (*content delivery network*) is an interconnected system of computers on the Internet that provides Web content rapidly to numerous users by duplicating the content on multiple servers and directing the content to users based on proximity.

When concurrent users will come to your site, using CDN rather than serving asset (like image, javascript, stylesheets) from your server will boost up performance.

## X. Minify and gzip stylesheets and javascript

This is the last point, but an important one. You can reduce size of the stylesheets and javascript significantly by Minifying it and serve as GZip format. It will improve the performance significantly by reducing request/response time.

## XI. Specify which columns you need if possible

If you need to get the only email of user

```ruby
User.limit(100).map(&:email)
```

Can Be...

```ruby
User.select(:email).limit(100).map(&:email)
```

Better

```ruby
User.limit(100).pluck(:email)
```

Helpfully, if you just need a few values, you can use ActiveRecord#pluck to avoid instantiating ActiveRecord objects.

## XII. Use Background Jobs

We should use the background jobs for:

* The task need to spend a longer time to complete than the normal request.
* The task need not return an immediate response to the user.

## XIII. SQL Injection

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

## XIV. `serve_static_files` (or `serve_static_assets`) should be disabled (false) on production

We really shouldn't rely on serving assets from public/ via your Rails app, it is better to let the web server (e.g. Apache or Nginx) handle serving assets for performance.

