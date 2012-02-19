# simple_audit

Simple auditing solution for ActiveRecord models. Provides an easy way of
creating audit logs for complex model associations. Instead of storing audits

  * a helper method is provided to easily display the audit log
  * the Audit object provides a #delta method which computes the differences between two audits

In a nutshell: this is your model, Booking:

```ruby
# app/models/booking.rb

class Booking < ActiveRecord::Base
  simple_audit
  …
end
```

This is your view:

```ruby
# app/views/bookings/booking.html.erb
…
<%= render_audits(@booking) %>
…
```

If the data is blank, no audit entry is created.  So if your `simple_audit`
block returns any value that would result in a positive `blank?` check there
won't be an `Audit` record for that particular change.


## Why?

simple_audit is intended as a straightforward auditing solution for complex
model associations. In a normalized data structure (which is usually the case
when using SQL), a core business model will aggregate data from several
associated entities. More often than not in such cases, only the core model is
of interest from an auditing point of view.

So instead of auditing every entity separately, with simple_audit you decide
for each audited model what data needs to be logged. This data will be
serialized on every model update and the available helper methods will render
a clear audit trail.

![Screenshot of helper result](http://github.com/carlo/simple_audit/raw/master/screenshot.png)


## Installation & Configuration for Rails 3

    # in your Gemfile
    gem 'simple_audit'

    # then install
    bundle install

### Post installation

    # generate & run migration
    rails generate simple_audit:migration
    rake db:migrate


## Usage

Audit ActiveRecord models. Somewhere in your (backend) views show the audit
logs.

Your model, Booking:

```ruby
# app/models/booking.rb

class Booking < ActiveRecord::Base
  simple_audit
  …
end
```

Your view:

```ruby
# app/views/bookings/booking.html.erb
…
<%= render_audits(@booking) %>
…
```


## Setup

simple_audit needs to know who the current user is.  Set your current user
model object as the simple_audit `current_user` before any CRUD operations are
performed.  For example, in a Rails application you could add the following to
your `app/controllers/application_controller.rb`

```ruby
class ApplicationController < ActionController::Base
  before_filter :set_current_user

  private

  def set_current_user
    SimpleAudit::User.current_user = @current_user
  end
end
```


### Sample styling

```css
.audits {
	clear: both;
}

.audit {
	-moz-border-radius:8px;
  -webkit-border-radius: 8px;
  background-color: #dfdfdf;
  padding: 6px;
  margin-bottom: 8px;
	font-size: 12px;
}

.audit .action, .audit .user, .audit .timestamp{
	float: left;
	margin-right: 6px;
}

.audit .changes {
	clear: both;
	white-space: pre;
}

.audit .current {
	margin-left: 6px;
}

.audit .previous {
	margin-left: 6px;
	text-decoration: line-through;
}
```


## Customize auditing

By default after each save, all model's attributes and `belongs_to`
associations (their `id` and `to_s` on these) are saved in the `audits` table.
You can customize the data which is saved by supplying a block which will
return all relevant data for the audited model.

```ruby
# app/models/booking.rb

class Booking < ActiveRecord::Base
  simple_audit do |record|
    {
      :state  => record.state,
      :price  => record.price.format,
      :period => record.period.to_s,
      :housing_units => record.housing_units.collect(&:name).join('; '),
      …
      }
  end
  …
end
```

If your `simple_audit` block returns any value that would result in a positive
`blank?` check no `Audit` record for that particular change will be created.

You can also customize the attribute of the `User` model which will be stored
in the audit.

```ruby
# default is :name
simple_audit :username_method => :email
```


## Rendering audit trail

A helper method for displaying a list of audits is provided. It will render a
decorated list of the provided audits; only the differences between revisions
will be shown, thus making the audit information easily readable.

![Screenshot of helper result](http://github.com/carlo/simple_audit/raw/master/screenshot.png)


## Authors

Copyright (c) 2010 [Gabriel Târnovan, Cubus Arts](http://cubus.ro "Cubus
Arts"), released under the MIT license.  Additional changes by [Carlo
Zottmann](https://github.com/carlo) and [pzupan](https://github.com/pzupan).


## Changelog

### 0.3.0

* FIX: got rid of `class_inheritable_attribute` warnings in Rails 3 (pzupan)
* ADD: don't create audit records for records with blank changelogs
* ADD: set the user from anywhere via `SimpleAudit::User.current_user`
