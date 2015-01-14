#### Task 1: INTEGRATION TESTING BASIC AUTH
Our zombies resources need to be protected against unauthorized access. Let’s start by writing some integration tests that simulate authenticated requests using HTTP Basic Auth.

1. Issue a `GET` request to the zombies resources URI.
2. Using Basic Auth, authenticate the request using the proper request header.
3. Verify that the response returns a 200 - Successful status code.
4. Now verify that the response Content-Type is set to JSON.

```ruby
# task code
# test_helper.rb
ENV['RAILS_ENV'] = 'test'
require File.expand_path('../../config/environment', __FILE__)
require 'rails/test_help'

class ActiveSupport::TestCase
  ActiveRecord::Migration.check_pending!
  fixtures :all

  def encode_credentials(username, password)
    ActionController::HttpAuthentication::Basic.encode_credentials(username, password)
  end
end

# test/integration/listing_zombies_test.rb
class ListingZombiesTest < ActionDispatch::IntegrationTest
  setup { @user = User.create!(username: 'foo', password: 'secret') }

  test 'valid authentication lists zombies' do
  end
end


# solution code
class ListingZombiesTest < ActionDispatch::IntegrationTest
  setup { @user = User.create!(username: 'foo', password: 'secret') }

  test 'valid authentication lists zombies' do
  	get '/zombies', {}, { 'Authorization' => encode_credentials(@user.username, @user.password) }
  	assert_equal response.status, 200
  	assert_equal Mime::JSON, response.content_type
  end
end
```

#### Task 2: TESTING INVALID AUTHENTICATION
Now that we have tests in place for valid authentication requests, let’s write some tests that will ensure the correct behavior for requests with invalid authentication.

1. Pass in an empty value for the `Authorization` header.
2. Assert the response status code is set to `401` - Unauthorized.

```ruby
# task code
# test/integration/listing_zombies_test.rb
class ListingZombiesTest < ActionDispatch::IntegrationTest
  test 'invalid authentication responds with proper status code' do
    get '/zombies'
  end
end


# solution code
class ListingZombiesTest < ActionDispatch::IntegrationTest
  test 'invalid authentication responds with proper status code' do
    get '/zombies', {}, {'Authorization'=>''}
    assert_equal 401, response.status
  end
end
```

#### Task 3: IMPLEMENTING BASIC AUTH
With both tests in place, it’s now time to implement our API’s authentication strategy using HTTP Basic Auth. We will head back to ApplicationController and add a callback, which will run for each request that comes in.

1. Implement the `authenticate_basic_auth` method. This method should call `authenticate_with_http_basic`, which takes a block with two arguments: `username` and `password`.
2. By using the username and password arguments from inside the block, authenticate the request by calling User.authenticate and passing in those same arguments.

```ruby
# task code
# app/models/user.rb
class User < ActiveRecord::Base
  has_secure_password

  def self.authenticate(username, password)
    user = find_by(username: username)
    user && user.authenticate(password)
  end
end

# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  before_action :authenticate

  protected
    def authenticate
      authenticate_basic_auth
    end

    def authenticate_basic_auth
      # code here
    end
end


# solution code
# app/models/user.rb
class User < ActiveRecord::Base
  has_secure_password

  def self.authenticate(username, password)
    user = find_by(username: username)
    user && user.authenticate(password)
  end
end

# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base

  before_action :authenticate

  protected
    def authenticate
      authenticate_basic_auth
    end

    def authenticate_basic_auth
      authenticate_with_http_basic do |user, password|
        User.authenticate(user, password)
      end
    end
end
```

#### Task 4: CUSTOM BASIC AUTH RESPONSE
Now that we have implemented authentication for our API, we need to implement the response for invalid authentication attempts. It’s our responsibility to set the correct response status code and the correct response header.

1. On the render_unauthorized method, set the `WWW-Authenticate` response header to `'Basic realm="Zombies"'`.
2. Add support for both `JSON` and `XML` responses. Both should return the message Bad credentials with a 401 - Unauthorized status code.

```ruby
# task code
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base

  before_action :authenticate

  protected
    def authenticate
      authenticate_basic_auth || render_unauthorized
    end

    def authenticate_basic_auth
      authenticate_with_http_basic do |username, password|
        User.authenticate(username, password)
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
      authenticate_basic_auth || render_unauthorized
    end

    def authenticate_basic_auth
      authenticate_with_http_basic do |username, password|
        User.authenticate(username, password)
      end
    end

    def render_unauthorized
      self.headers['WWW-Authenticate'] = 'Basic realm="Zombies"'

      respond_to do |format|
        format.json { render json: 'Bad credentials', status: 401 }
        format.xml { render xml: 'Bad credentials', status: 401 }
      end
    end
end
```

#### Task 5: CURL WITH BASIC AUTH
Use curl to make an authenticated request using HTTP Basic Auth to the following `URL: http://cs-zombies-dev.com:3000/zombies`. Set the username to foo and password to secret, and make sure to use a request header to specify JSON as the expected format for the response.

```
# solution code
$ curl -IH "Accept: application/json" -u 'foo:secret' http://cs-zombies-dev.com:3000/zombies
```
