Routes
===

## Resources Routing

### CRUD

```ruby
resources :photos
```

| HTTP Verb | Path             | Controller#Action | Used for                                     |
| ----------|------------------|-------------------|----------------------------------------------|
| GET       | /photos          | photos#index      | display a list of all photos                 |
| GET       | /photos/new      | photos#new        | return an HTML form for creating a new photo |
| POST      | /photos          | photos#create     | create a new photo                           |
| GET       | /photos/:id      | photos#show       | display a specific photo                     |
| GET       | /photos/:id/edit | photos#edit       | return an HTML form for editing a photo      |
| PATCH/PUT | /photos/:id      | photos#update     | update a specific photo                      |
| DELETE    | /photos/:id      | photos#destroy    | delete a specific photo                      |

### Path and URL Helpers

In the case of *resources: photos*

* *photos_path* returns */photos*
* *new_photo_path* returns */photos/new*
* *edit_photo_path(:id)** returns */photos/:id/edit* (for instance, edit_photo_path(10) returns /photos/10/edit)
* *photo_path(:id)** returns */photos/:id* (for instance, photo_path(10) returns /photos/10)

### Defining Multiple Resources at the Same Time

```ruby
resources :photos, :books, :videos
```

### Controller Namespace and Routing

```ruby
namespace :admin do
  resources :articles, :comments
end
```

### Nested Resources

Suppose your application includes these models:

```ruby
class Magazine < ApplicationRecord
  has_many :ads
end
 
class Ad < ApplicationRecord
  belongs_to :magazine
end
```

Nested routes allow you to capture this relationship in your routing. In this case, you could include this route declaration:

```ruby
resources :magazines do
  resources :ads
end
```

### Member Routes

```ruby
resources :photos do
  member do
    get 'preview' # /photo/1/preview
  end
end
```

This will recognize */photos/1/preview* with *GET*, and route to the *preview* action of *PhotosController*, with the resource id value passed in params[:id]. It will also create the *preview_photo_url* and *preview_photo_path* helpers.

## Collection Routes

```ruby
resources :photos do
  collection do
    get 'search' # /photo/search
  end
end
```

## Non-Resourceful Routes

### Bound Parameters

```ruby
get 'photos(/:id)', to: :display # /photos(/1)
```

### Dynamic Segments

```ruby
get 'photos/:id/:user_id', to: 'photos#show'
```

An incoming path of */photos/1/2* will be dispatched to the *show* action of the *PhotosController*. *params[:id]* will be "1", and *params[:user_id]* will be "2".

### Naming Routes

```ruby
get 'exit', to: 'sessions#destroy', as: :logout

# Helpers
logout_path
logout_url
```

### HTTP Verb Constraints

```ruby
match 'photos', to: 'photos#show', via: [:get, :post]
match 'photos', to: 'photos#show', via: :all # match all verbs
```

### Segment Constraints

```ruby
get 'photos/:id', to: 'photos#show', constraints: { id: /[A-Z]\d{5}/ } # /photos/A12345
```

### Route Globbing and Wildcard Segments

```ruby
get 'photos/*other', to: 'photos#unknown' #  photos/12 or /photos/long/path/to/12, params[:other] = 12
```

### Using root

```ruby
root to: 'pages#main'
root 'pages#main' # shortcut for the above
```

You can use root inside namespaces ans scopes as well.

```ruby
namespace :admin do
  root to "admin#index"
end
```

## Customizing Resourceful Routes

### Specifying a Controller to Use

```ruby
resources :photos, controller: 'images'
```

### Specifying Constraints

```ruby
resources :photos, constraints: { id: /[A-Z][A-Z][0-9]+/ } # /photos/RR27

# Using block form to apply single constraint to a number of routes
constraints(id: /[A-Z][A-Z][0-9]+/) do
  resources :photos
  resources :accounts
end
```
