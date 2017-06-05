Views
=====

## Creating Responses
* *render*: full response to send back to the browser
* *redirect_to*: send an HTTP redirect status code to the browser

### Convention

```ruby
# app/controllers/books_controller.rb
class BooksController < ApplicationController
  def index
    @books = Book.all
  end
end

# config/routes.rb
resources :books

# app/views/books/index.html.erb
<h1>Listing Books</h1>
 
<table>
  <tr>
    <th>Title</th>
    <th>Summary</th>
    <th></th>
    <th></th>
    <th></th>
  </tr>
 
<% @books.each do |book| %>
  <tr>
    <td><%= book.title %></td>
    <td><%= book.content %></td>
    <td><%= link_to "Show", book %></td>
    <td><%= link_to "Edit", edit_book_path(book) %></td>
    <td><%= link_to "Remove", book, method: :delete, data: { confirm: "Are you sure?" } %></td>
  </tr>
<% end %>
</table>
 
<br>
 
<%= link_to "New book", new_book_path %>
```

### Using render

#### Render an Action's View

```ruby
def update
  @book = Book.find(params[:id])
  if @book.update(book_params)
    redirect_to(@book)
  else
    render "edit" # edit.html.erb
  end
end

# Using symbol
def update
  @book = Book.find(params[:id])
  if @book.update(book_params)
    redirect_to(@book)
  else
    render :edit
  end
end
```

#### Rendering an Action's Template from Another Controller

```ruby
render "products/show" # app/views/products/show.html.erb
```

You can use *:template* option if you want to be explicit

```ruby
render template: "products/show"
```

#### Rendering an Arbitrary File

```ruby
render file: "/u/apps/warehouse_app/current/app/views/products/show"
```

#### Wrapping it up

```ruby
render :edit
render action: :edit
render "edit"
render "edit.html.erb"
render action: "edit"
render action: "edit.html.erb"
render "books/edit"
render "books/edit.html.erb"
render template: "books/edit"
render template: "books/edit.html.erb"
render "/path/to/rails/app/views/books/edit"
render "/path/to/rails/app/views/books/edit.html.erb"
render file: "/path/to/rails/app/views/books/edit"
render file: "/path/to/rails/app/views/books/edit.html.erb"
```

### Using redirect_to

```ruby
redirect_to photos_url

# Return the user to the page they just came from
redirect_back(fallback_location: root_path)
```

## Structuring Layouts

### Asset Tag Helpers

* auto_discovery_link_tag
* javascript_include_tag
* stylesheet_link_tag
* image_tag
* video_tag
* audio_tag

```ruby
<%= javascript_include_tag "main" %> # app/assets/javascripts/main.js
<%= stylesheet_link_tag "main" %> # app/assets/stylesheets/main.css
<%= image_tag "header.png" %>
```

### yield and content_for

```ruby

# app/views/layouts/applications.html.erb
<html>
  <head>
  <%= yield :head %>
  </head>
  <body>
  <%= yield %>
  </body>
</html>

# app/views/static_pages/sample_page.html.erb
<% content_for :head do %>
  <title>A simple page</title>
<% end %>
 
<p>Hello, Rails!</p>

# Rendered HTMl

<html>
  <head>
  <title>A simple page</title>
  </head>
  <body>
  <p>Hello, Rails!</p>
  </body>
</html>
```

### Using partials

```ruby
<%= render "menu" %> # _menu.html.erb
<%= render "shared/menu" %> # app/views/shared/_menu.html.erb
```

### Passing local variables

```ruby
# new.html.erb
<h1>New zone</h1>
<%= render partial: "form", locals: {zone: @zone} %>

# _form.html.erb
<%= form_for(zone) do |f| %>
  <p>
    <b>Zone name</b><br>
    <%= f.text_field :name %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>
```
