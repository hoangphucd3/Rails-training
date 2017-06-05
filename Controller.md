Controller
========

## Naming Convention

The naming convention of controllers in Rails favors pluralization of the last word in the controller's name, although it is not strictly required (*e.g. ApplicationController*). For example, *ClientsController* is preferable to *ClientController*, *SiteAdminsController* is preferable to *SiteAdminController* or *SitesAdminsController*, and so on.

## Methods and Actions

 When your application receives a request, the routing will determine which controller and action to run, then Rails creates an instance of that controller and runs the method with the same name as the action.
 
 Only public methods are callable as actions. It is a best practice to lower the visibility of methods (with *private* or *protected*) which are not intended to be actions, like auxiliary methods or filters.

## Parameters

### Strong parameter

```ruby
class PeopleController < ActionController::Base
  # This will raise an ActiveModel::ForbiddenAttributesError exception
  # because it's using mass assignment without an explicit permit
  # step.
  def create
    Person.create(params[:person])
  end
 
  # This will pass with flying colors as long as there's a person key
  # in the parameters, otherwise it'll raise a
  # ActionController::ParameterMissing exception, which will get
  # caught by ActionController::Base and turned into a 400 Bad
  # Request error.
  def update
    person = current_account.people.find(params[:id])
    person.update!(person_params)
    redirect_to person
  end
 
  private
    # Using a private method to encapsulate the permissible parameters
    # is just a good pattern since you'll be able to reuse the same
    # permit list between create and update. Also, you can specialize
    # this method with per-user checking of permissible attributes.
    def person_params
      params.require(:person).permit(:name, :age)
    end
end
```

## Session

All session stores use a cookie to store a unique ID for each session

### Accessing the Session

In your controller you can access the session through the session instance method.

```ruby
class ApplicationController < ActionController::Base
 
  private
 
  # Finds the User with the ID stored in the session with the key
  # :current_user_id This is a common way to handle user login in
  # a Rails application; logging in sets the session value and
  # logging out removes it.
  def current_user
    @_current_user ||= session[:current_user_id] &&
      User.find_by(id: session[:current_user_id])
  end
end
```

To store something in the session, assign it to the key like a hash:

```ruby
class LoginsController < ApplicationController
  # "Create" a login, aka "log the user in"
  def create
    if user = User.authenticate(params[:username], params[:password])
      # Save the user ID in the session so it can be used in
      # subsequent requests
      session[:current_user_id] = user.id
      redirect_to root_url
    end
  end
end
```

To remove something from session, assign that key to *nil*

```ruby
class LoginsController < ApplicationController
  # "Delete" a login, aka "log the user out"
  def destroy
    # Remove the user id from the session
    @_current_user = session[:current_user_id] = nil
    redirect_to root_url
  end
end
```

### The Flash

The flash is a special part of the session which is cleared with each request.

```ruby
class LoginsController < ApplicationController
  def destroy
    session[:current_user_id] = nil
    flash[:notice] = "You have successfully logged out."
    redirect_to root_url
  end
end
```

Display errors or notices from the flash in application's layout:

```html
<html>
  <!-- <head/> -->
  <body>
    <% flash.each do |name, msg| -%>
      <%= content_tag :div, msg, class: name %>
    <% end -%>
 
    <!-- more content -->
  </body>
</html>
```

#### flash.now

Access flash's value in same request

```ruby
class ClientsController < ApplicationController
  def create
    @client = Client.new(params[:client])
    if @client.save
      # ...
    else
      flash.now[:error] = "Could not save client"
      render action: "new"
    end
  end
end
```

## Cookies

```ruby
class CommentsController < ApplicationController
  def new
    # Auto-fill the commenter's name if it has been stored in a cookie
    @comment = Comment.new(author: cookies[:commenter_name])
  end
 
  def create
    @comment = Comment.new(params[:comment])
    if @comment.save
      flash[:notice] = "Thanks for your comment!"
      if params[:remember_name]
        # Remember the commenter's name.
        cookies[:commenter_name] = @comment.author
      else
        # Delete cookie for the commenter's name cookie, if any.
        cookies.delete(:commenter_name)
      end
      redirect_to @comment.article
    else
      render action: "new"
    end
  end
end
```

## Rendering XML and JSON data

```ruby
class UsersController < ApplicationController
    def index
       @users = User.all
       respond_to do |format|
           format.html # index.html.erb
           format.xml  { render xml: @users }
           format.json { render json: @users }
       end
    end
end
```

## Filters

* before_action
* after_action
* around_action

```ruby

# before_action
class ApplicationController < ActionController::Base
  before_action :require_login
 
  private
 
  def require_login
    unless logged_in?
      flash[:error] = "You must be logged in to access this section"
      redirect_to new_login_url # halts request cycle
    end
  end
end

# Inline filter
class ApplicationController < ActionController::Base
  before_action do |controller|
    unless controller.send(:logged_in?)
      flash[:error] = "You must be logged in to access this section"
      redirect_to new_login_url
    end
  end
end

# Filter exceptions
class LoginsController < ApplicationController
  skip_before_action :require_login, only: [:new, :create]
end
```
