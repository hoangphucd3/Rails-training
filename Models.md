Models
======

## Convention

### Naming conventions

The model class name should follow the Ruby convention, using the CamelCase form, while the table name must contain the words separated by underscores.

 - **Database table**: Plural with underscores separating words (e.g., *book_clubs*)
 - **Model Class**: Singular with the first letter of each word capitalized (e.g., *BookClub*)

### Schema conventions

 - **Foreign keys**: should be named following the pattern *singularized_table_name_id (e.g., item_id, order_id)*
 - **Primary keys** - By default, Active Record will use an integer column named id as the table's primary key. When using Active Record Migrations to create tables, this column will be automatically created.

## Migrations

### Creating a standalone migration

```ruby
rails generate migration AddPartNumberToProducts
```

### Passing modifiers

Some commonly modifiers can be passed directly on the command line. They are enclosed by curly braces and follow the field type:

```ruby
rails generate migration AddDetailsToProducts 'price:decimal{5,2}'
```

### Writing a migration

#### Creating a table

A typical would be

```ruby
create_table :products do |t|
    t.string :name
end
```

which creates a *products* table with a column called *name*

#### Using the change method

The *change* method is the primary way of writing migrations. It works for the majority of cases, where Active Record knows how to reverse the migration automatically.

### Running Migrations

```ruby
rails db:migrate
```

### Rolling Back

```ruby
rails db:rollback
```

### Setup the Database

```ruby
rails db:setup
```

This command will create the database, load the schema and initialize it with the seed data.

### Resetting the Database

```ruby
rails db:reset
```

This command will drop the database and set it up again.

## Validation

### Validate checkbox

```ruby
class Person < ApplicationRecord
    validates :terms_of_service, acceptance: true
end
```
### Validate associated records

```ruby
class Library < ActiveRecord::Base
    has_many :books
    validates_associated :books
end
```

### Confirmation

```ruby
class Person < ActiveRecord::Base
    validates :email, :confirmation => true
end
```

### Validate format

```ruby
class Product < ActiveRecord::Base
    validates :legacy_code, :format => { :with => /\A[a-zA-Z]+\z/, :message => "Only letters allowed" }
  end
```
### Validate length

```ruby
class Person < ActiveRecord::Base
    validates :name, :length => { :minimum => 2 }
    validates :bio, :length => { :maximum => 500 }
    validates :password, :length => { :in => 6..20 }
    validates :registration_number, :length => { :is => 6 }
end
```
### Validate numeric

```ruby
class Player < ActiveRecord::Base
    validates :points, :numericality => true
    validates :games_played, :numericality => { :only_integer => true }
end
```

### Validate non-empty

```ruby
class Person < ActiveRecord::Base
    validates :name, :login, :email, :presence => true
  end
```

### Validate uniqueness

```ruby
class Account < ApplicationRecord
    validates :email, uniqueness: true
  end
```

### Custom validation

```ruby
class Person < ActiveRecord::Base
    validate :foo_cant_be_nil

    def foo_cant_be_nil
      errors.add(:foo, 'cant be nil')  if foo.nil?
    end
end
```

## Associations

### One to One

```ruby
  # app/models/user.rb
  class User < ApplicationRecord
    has_one :account
  end

  # app/models/account.rb
  class Account < ApplicationRecord
    belongs_to :user
  end
```

### One To Many

```ruby
  # app/models/user.rb
  class User < ApplicationRecord
    has_many :orders
  end

  # app/models/order.rb
  class Order < ApplicationRecord
    belongs_to :user
  end
```

### Many to Many

```ruby
# app/models/category.rb
  class Category < ApplicationRecord
   has_many :relations
   has_many :products, through: :relations
  end

  # app/models/product.rb
  class Product < ApplicationRecord
    has_many :relations
    has_many :categories, through: :relations
  end

  # app/models/relation.rb
  class Relation < ApplicationRecord
   belongs_to :category
   belongs_to :product
  end
```

## Call backs

* after_create
* after_initialize
* after_validation
* after_save
* after_commit
