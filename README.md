<h2 align="center" class='center' style="text-align:center">
  TheRole::Api. Role model and general API methods
</h2>

<p align="center" class='center' style="text-align:center">
  <b>Authorization gem for Ruby on Rails</b><br>
  <i>with <a href="https://github.com/TheRole/TheRoleManagementPanelBootstrap3">Management Panel</a></i>
</p>

<p align="center" class='center' style="text-align:center">
  <img src="https://raw.githubusercontent.com/TheRole/TheRoleApi/master/docs/the_role.png" alt="TheRole. Authorization gem for Ruby on Rails with Administrative interface">
</p>

<p align="center" class='center' style="text-align:center">
  <b>Semantic. Flexible. Lightweigh</b>
</p>

<div align="center" class='center' style="text-align:center">

<a href="http://badge.fury.io/rb/the_role"><img src="https://badge.fury.io/rb/the_role.svg" alt="Gem Version" height="18"></a>
&nbsp;
<a href="https://travis-ci.org/TheRole/DummyApp"><img src="https://travis-ci.org/TheRole/DummyApp.svg?branch=master" alt="Build Status" height="18"></a>
&nbsp;
<a href="https://codeclimate.com/github/TheRole/TheRoleApi"><img src="https://codeclimate.com/github/TheRole/TheRoleApi/badges/gpa.svg" /></a>
&nbsp;
<a href="https://www.ruby-toolbox.com/categories/rails_authorization">ruby-toolbox</a>
</div>

### INTRO

TheRole is an authorization library for Ruby on Rails which restricts what resources a given user is allowed to access. All permissions are defined in with **2-level-hash**, and **stored in the database as a JSON string**.

<p align="center" class='center' style="text-align:center">
  <img src="./docs/hash2string.png" alt="TheRole. Authorization gem for Ruby on Rails with Administrative interface">
</p>

Using hashes, makes role system extremely easy to configure and use

* Any Role is a two-level hash, consisting of the <b>sections</b> and nested <b>rules</b>
* A <b>Section</b> may be associated with a <b>controller</b> name
* A <b>Rule</b> may be associated with an <b>action</b> name
* A Section can have many rules
* A Rule can be <b>true</b> or <b>false</b>
* <b>Sections</b> and nested <b>Rules</b> provide an <b>ACL</b> (<b>Access Control List</b>)

#### Import/Export

If you have 2 Rails apps, based on TheRole - you can move roles between them via export/import abilities of TheRole Management Panel.
It can be usefull for Rails apps based on one engine.

<div align="center" class='center' style="text-align:center">
  <img src="./docs/import_export.png" alt="TheRole. Authorization gem for Ruby on Rails with Administrative interface">
</div>

### Installation

**Gemfile**

```ruby
gem 'the_role_api', '~> 3.0.0'
```

or

```ruby
gem 'the_role_api',
  github: 'TheRole/the_role_api',
  branch: 'master'
```

and after that

```sh
bundle
```

#### Change User migration file

Add a **role_id:integer** field to your User Model

```ruby
def self.up
  create_table :users do |t|
    t.string :login
    t.string :email
    t.string :crypted_password
    t.string :salt

    # !!! TheRole field !!!
    t.integer :role_id

    t.timestamps
  end
end
```

#### Install TheRole migration file

```sh
bundle exec rake the_role_engine:install:migrations
```

you will see following line

```sh
  Copied migration XXXXXXXXXXXXX_create_roles.the_role_engine.rb from the_role_engine
```

#### Invoke migrations

```sh
rake db:migrate
```

#### Change User model

```ruby
class User < ActiveRecord::Base
  # include TheRole::Api::User
  has_the_role

  # ... code ...
end
```

#### Create Role model

```sh
bundle exec rails g the_role install
```

you will see following lines

```show
  create  app/models/role.rb
  create  config/initializers/the_role.rb
```

#### Create admin role

```sh
rake db:the_role:admin
```

you will see following line

```sh
  TheRole >>> Admin role created
```

Now you can make any user an Admin via `rails console`, for instance:

```ruby
User.first.update( role: Role.with_name(:admin) )
User.first.admin? # => true
```

#### Integration with Rails controllers

**application_controller.rb**

```
class ApplicationController < ActionController::Base

  include TheRole::Controller

  protect_from_forgery with: :exception
  protect_from_forgery

  # ... code ...
end
```

Any Rails controller, for instance, **pages_controller.rb**

```ruby
class PagesController < ApplicationController
  before_action :login_required, except: [ :index, :show ]
  before_action :role_required,  except: [ :index, :show ]

  # !!! WARNING !!!
  #
  # `@owner_check_object` variable have to be exists
  # before check ownership via `owner_required` method
  #
  # You have to define `@owner_check_object` in `set_page` method

  before_action :set_page,       only: [ :edit, :update, :destroy ]
  before_action :owner_required, only: [ :edit, :update, :destroy ]

  private

  def set_page
    @page = Page.find params[:id]

    # TheRole: object ownership checking
    @owner_check_object = @page
  end
end
```

# TheRole API

## User

```ruby
# User's role
@user.role # => Role obj
```

Is a user Administrator?

```ruby
@user.admin?                       => true | false
```

Is a user Moderator?

```ruby
@user.moderator?(:pages)           => true | false
@user.moderator?(:blogs)           => true | false
@user.moderator?(:articles)        => true | false
```

Has user got access to **rule** of **section** (action of controller)?

```ruby
@user.has_role?(:pages,    :show)  => true | false
@user.has_role?(:blogs,    :new)   => true | false
@user.has_role?(:articles, :edit)  => true | false

# return true if one of roles is true
@user.any_role?(pages: :show, posts: :show) => true | false
```

Is user **Owner** of object?

```ruby
@user.owner?(@page)                => true | false
@user.owner?(@blog)                => true | false
@user.owner?(@article)             => true | false
```

## Role

```ruby
# Find a Role by name
@role = Role.with_name(:user)
```

```ruby
@role.has?(:pages, :show)       => true | false
@role.moderator?(:pages)        => true | false
@role.admin?                    => true | false

# return true if one of roles is true
@role.any?(pages: :show, posts: :show) => true | false
```

#### CREATE

```ruby
# Create a section of rules
@role.create_section(:pages)
```

```ruby
# Create rule in section (false value by default)
@role.create_rule(:pages, :index)
```

#### READ

```ruby
@role.to_hash => Hash

# JSON string
@role.to_json => String

# check method
@role.has_section?(:pages) => true | false
```

#### UPDATE

```ruby
# set this rule on
@role.rule_on(:pages, :index)
```

```ruby
# set this rule off
@role.rule_off(:pages, :index)
```

```ruby
# Incoming hash is true-mask-hash
# All the rules of the Role will be reset to false
# Only rules from true-mask-hash will be set true
new_role_hash = {
  :pages => {
    :index => true,
    :show => true
  }
}

@role.update_role(new_role_hash)
```

#### DELETE

```ruby
# delete a section
@role.delete_section(:pages)

# delete a rule in section
@role.delete_rule(:pages, :show)
```

### Limitations by Design

0. Only `User` Model supported  (why?)
0. `User` **has_one** `Role`    (why?)
0. Role stored in database as a JSON String (why?)

### MIT License

Copyright (c) 2012-2015 [Ilya N.Zykin]
