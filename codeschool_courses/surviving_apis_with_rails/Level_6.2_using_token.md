#### Task 1: INTEGRATION TESTING TOKEN AUTH
We now need to protect our zombies resources against unauthorized access by using a Token Based Auth. Let’s begin by writing some integration tests that simulate authenticated API requests.

1. Issue a GET request to the zombies resource URI.
2. By using a Token Based strategy, authenticate the request using the proper request header, and the token value set to @user.auth_token. Make sure to check the test_helper.rb file on the secondary tab for a test helper method that you may need.
3. Verify that the response returns a 200 - Successful status code.
4. Now verify that the response content type is set to JSON.
```ruby
# task code
# test_helper.rb
ENV['RAILS_ENV'] = 'test'
require File.expand_path('../../config/environment', __FILE__)
require 'rails/test_help'

class ActiveSupport::TestCase
  ActiveRecord::Migration.check_pending!
  fixtures :all

  def token_header(token)
    ActionController::HttpAuthentication::Token.encode_credentials(token)
  end
end

# test/integration/listing_zombies_test.rb
class ListingZombiesTest < ActionDispatch::IntegrationTest
  setup { @user = User.create! }

  test 'valid token lists zombies' do

  end
end


# solution code
class ListingZombiesTest < ActionDispatch::IntegrationTest
  setup { @user = User.create! }

  test 'valid token lists zombies' do
    get '/zombies', {}, { 'Authorization' => token_header(@user.auth_token) }
    assert_equal response.status, 200
    assert_equal Mime::JSON, response.content_type
  end
end
```

#### Task 2: GENERATING AUTH TOKEN
The first step to implementing Token Based Auth is to generate a unique authentication token for each user. To do that, we’ll add an `ActiveRecord` callback to our User model, which runs everytime a new user record is created.

1. From the `set_auth_token` method, add a guard clause that immediatelly returns from the method if the `auth_token` property is already set.
2. Now use `TokenGenerator.create` to set the `auth_token` property on the `User` model.

```ruby
# task code
# app/models/token_generator.rb
class TokenGenerator
  def self.create
    loop do
      token = SecureRandom.hex
      break token unless User.exists?(auth_token: token)
    end
  end
end

# app/models/user.rb
class User < ActiveRecord::Base
  before_create :set_auth_token

  private

    def set_auth_token
      # code here
    end
end


# solution code
# app/models/user.rb
require 'token_generator'

class User < ActiveRecord::Base
  before_create :set_auth_token

  private

    def set_auth_token
      return if auth_token.present?
      self.auth_token = TokenGenerator.create
    end
end
```

#### Task 3: IMPLEMENTING TOKEN BASED AUTH
Finally, let’s add the Token Based Authentication strategy to our API. We’ll go back to `ApplicationController` and add a callback, which will be run for each request that comes in.

1. Implement the `authenticate_token` method. This method should call `authenticate_with_http_token`, which takes a block as an argument. The block should take token as its first argument.
2. Inside the block, authenticate the request using `User.find_by` and the token from the block argument. The field on the users table that holds the token is called `auth_token`.

```ruby
# task code
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  before_action :authenticate

  protected
    def authenticate
      authenticate_token
    end

    def authenticate_token
      # code here
    end
end


# solution code
class ApplicationController < ActionController::Base
  before_action :authenticate

  protected
    def authenticate
      authenticate_token
    end

    def authenticate_token
      authenticate_with_http_token do |token|
        User.find_by(auth_token: token)
      end
    end
end
```

#### Task 4: CUSTOM TOKEN RESPONSE
Now that you’re getting the hang of this, we need to implement the response for invalid Token Based authentication attempts. It is up to us to set the correct response status code and the correct response header.

1. On the render_unauthorized method, set the `WWW-Authenticate` header to `'Token realm="Zombies"'`.
2. Now add support for both `JSON` and `XML` responses. Both should return the message Bad credentials with a 401 - Unauthorized status code.

```ruby
# task code
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  before_action :authenticate

  protected
    def authenticate
      authenticate_token || render_unauthorized
    end

    def authenticate_token
      authenticate_with_http_token do |token|
        User.find_by(auth_token: token)
      end
    end

    def render_unauthorized
      # code here
    end
end

# solution code
class ApplicationController < ActionController::Base
  before_action :authenticate

  protected
    def authenticate
      authenticate_token || render_unauthorized
    end

    def authenticate_token
      authenticate_with_http_token do |token|
        User.find_by(auth_token: token)
      end
    end

    def render_unauthorized
      self.headers['WWW-Authenticate'] = 'Token realm="Zombies"'
      respond_to do |format|
        format.json { render json: 'Bad credentials', status: 401 }
        format.xml { render xml: 'Bad credentials', status: 401 }
      end
    end
end
```

#### Task 5: CURL AND TOKEN BASED AUTH
Use curl to make an authenticated request to `http://localhost:3000/zombies` using Token Based Auth. Set the token value to `a45fb396579a25458d23208560742610` and use a request header to specify JSON as the expected format for the response. For this request, we want the response body back, and not only headers.

```
# solution code
$ curl -IH "Authorization: Token token=a45fb396579a25458d23208560742610" -H "Accept: application/json" http://cs-zombies-dev.com:3000/zombies
```
