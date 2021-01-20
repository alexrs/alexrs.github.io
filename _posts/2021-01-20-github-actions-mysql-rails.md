---
title:  "Using MySQL and Rails with Github Actions"
description: How to configure Github Actions to work with MySQL and Rails. 
date: 2021-01-20 14:00:00
layout: post
---

I recently had to configure Github Actions to execute tests in a Rails project that uses MySQL.  At first glance, it
seemed like a reasonably easy task, but I ran across some issues when connecting to the database with errors that 
looked like:

```ruby
Mysql2::Error::ConnectionError: Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)
```

or:

```ruby
Mysql2::Error::ConnectionError: Access denied for user 'root'@'localhost' (using password: NO)
```

Here's how I solved it. Github provides a `mysql` service. The way to configure it is the following:

```yaml
   services:
      mysql:
        image: mysql:5.7
        env:
          MYSQL_ALLOW_EMPTY_PASSWORD: yes
          MYSQL_DATABASE: your-db_dev
        ports:
          - 3306
        options: >-
          --health-cmd="mysqladmin ping"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=3
```

We will use MySQL 5.7. We can see some interesting bits. `MYSQL_ALLOW_EMPTY_PASSWORD: yes` tells MySQL that we don't
need to specify a password. You can use a password here, although I don't know what the default is in this case
(maybe `password`?). For testing, I think it's usually fine to use this option. `MYSQL_DATABASE` specifies the name of
your testing database. This needs to match the one declared in your `database.yml` file.

After that, we will specify the steps we want to run. In my case, they look like this:

```yaml
    steps:
    - uses: actions/checkout@v2
    - name: Set up Ruby 2.6
      uses: actions/setup-ruby@v1
      with:
        ruby-version: 2.6
    - name: Install dependencies
      run: |
        gem install bundler
        bundle install --jobs 4 --retry 3
    - name: Run Tests
      run: |
        sudo service mysql start
        bundle exec rails db:prepare
        bundle exec rails test
      env:
        DB_PORT: 3306 
        RAILS_ENV: test
```

Nothing surprising here. We first fetch the code, then we set up Ruby, and we install the dependencies. Then, we 
start the MySQL service with `sudo service mysql start`. This command is very important. Otherwise, you will get one
of the errors mentioned above.

This is how my file looks like:

```yaml
name: CI

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      mysql:
        image: mysql:5.7
        env:
          MYSQL_ALLOW_EMPTY_PASSWORD: yes
          MYSQL_DATABASE: your-db_dev
        ports:
          - 3306
        options: >-
          --health-cmd="mysqladmin ping"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=3
    steps:
    - uses: actions/checkout@v2
    - name: Set up Ruby 2.6
      uses: actions/setup-ruby@v1
      with:
        ruby-version: 2.6
    - name: Install dependencies
      run: |
        gem install bundler
        bundle install --jobs 4 --retry 3
    - name: Run Tests
      run: |
        sudo service mysql start
        bundle exec rails db:prepare
        bundle exec rails test
      env:
        DB_PORT: 3306
        RAILS_ENV: test
```

Last but not least, this is how my `database.yml` file looks like:

```yaml
default: &default
  adapter: mysql2
  collation: utf8mb4_unicode_ci
  encoding: utf8mb4

development: &development
  <<: *default
  username: root
  database: your-db_dev

test:
  <<: *development
  database: your-db_test
  <% if ENV['CI'] %>
  host: 127.0.0.1
  port: <%= ENV['DB_PORT'] %>
  <% end %>

production: &production
  <<: *default
  url: <%= ENV['DATABASE_URL'] %>
```

The interesing part here is found in the `test` configuration. If the environment is CI, we connect to the host
`127.0.0.1` and the port available in the variable `DB_PORT`.

Hope this was useful!

Alejandro 👾.

