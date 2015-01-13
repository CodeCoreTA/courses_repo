#### Task 1: CREATING A VERSIONED NAMESPACE
`Route` namespaces help isolate controllers that serve different API versions. Let’s create individual namespaces for the different versions supported by our API server.

1. Create a namespace called `v1` and move the zombies resources into that namespace.
2. Now create a second namespace called `v2`. Inside of `v2`, define another zombies resources.
3. For the zombies resources under the `v2` namespace, prevent the `destroy` action from being exposed.

```ruby
# task code
# config/routes.rb
SurvivingRails::Application.routes.draw do
  resources :zombies
end


# solution code
SurvivingRails::Application.routes.draw do
  namespace :v1 do
  	resources :zombies
  end

  namespace :v2 do
  	resources :zombies, except: :destroy
  end
end
```

#### Task 2: TESTING ROUTES FOR URI VERSIONING
Routes tests help ensure URIs are routed to the proper `controller#action`. Let’s use the `assert_generates` method to verify that versioned URIs are pointing to the right controllers.

1. Create an assertion that checks that `/v1/zombies` points to the index action on the `V1::ZombiesController`.
2. Now create an assertion that checks that `/v2/zombies` is mapped to the index action on the `V2::ZombiesController`.

```ruby
# task code
# test/integration/routes_test.rb
class RoutesTest < ActionDispatch::IntegrationTest
  test 'routes to proper versions' do
    # your code here
  end
end


# solution code
class RoutesTest < ActionDispatch::IntegrationTest
  test 'routes to proper versions' do
    assert_generates '/v1/zombies', { controller: 'v1/zombies', action: 'index' }
    assert_generates '/v2/zombies', { controller: 'v2/zombies', action: 'index' }
  end
end
```

#### Task 3: RENDERING ZOMBIES
With our routes test in place we can now implement the index action for our `V1::ZombiesController` and render back some zombies in JSON format.

1. Create the proper class for `V1::ZombiesController`. Notice the controller needs to be part of the V1 namespace.
2. Implement the index action, returning a JSON list of all zombies and setting the status code to 200 - Success.

```ruby
# solution code
module V1
	class ZombiesController < ApplicationController
		def index
			zombies = Zombie.all
			render json: zombies, status: 200
		end
	end
end
```

#### Task 4: TESTING SETTING THE REMOTE_ADDR HEADER
As our application continues to grow, we are starting to create some unnecessary code duplication. But before we can refactor our code into something cleaner, we need to write some tests. These tests will ensure things are currently working and will also help make sure we don’t break anything as we move forward with the changes.

1. On the test block for `v1`, set the `REMOTE_ADDR` request header with the value from `@ip`.
2. On the test block for `v2`, set the `REMOTE_ADDR` request header with the value from `@ip`.

```ruby
# task code
# test/integration/zombies_with_ip_test.rb
class ZombiesWithIpTest < ActionDispatch::IntegrationTest
  setup { @ip = '192.168.1.12' }

  test '/v1 returns ip and v1' do
    get '/v1/zombies', {}, { '' => ... }
    assert_equal 200, response.status
    assert_equal "#{@ip} and version one", response.body
  end

  test '/v2 returns ip and v2' do
    get '/v2/zombies', {}, { '' => ... }
    assert_equal 200, response.status
    assert_equal "#{@ip} and version two", response.body
  end
end

# solution code
class ZombiesWithIpTest < ActionDispatch::IntegrationTest
  setup { @ip = '192.168.1.12' }

  test '/v1 returns ip and v1' do
    get '/v1/zombies', {}, { 'REMOTE_ADDR' => @ip }
    assert_equal 200, response.status
    assert_equal "#{@ip} and version one", response.body
  end

  test '/v2 returns ip and v2' do
    get '/v2/zombies', {}, { 'REMOTE_ADDR' => @ip }
    assert_equal 200, response.status
    assert_equal "#{@ip} and version two", response.body
  end
end


#### Task 5: USING BEFORE_ACTION TO SET THE IP
Our app now strictly serves a web API, which means we can use `ApplicationController` to share common code across different API versions.

1. On `ApplicationController`, create a `before_action` that sets the instance variable `@user_ip` with the value from the `REMOTE_ADDR` request header.
2. On `V1::ZombiesController`, render the proper JSON message that includes the client’s ip address. Make sure to check the integration test file tab on the far right for the actual message.
3. On `V2::ZombiesController`, render the proper JSON message that includes the client’s ip address.

# task code
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
end

# test/integration/zombies_with_ip_test.rb
require 'test_helper'

class ZombiesWithIpTest < ActionDispatch::IntegrationTest
  setup { @ip = '192.168.1.12' }

  test '/v1 returns ip and v1' do
    get '/v1/zombies', {}, { 'REMOTE_ADDR' => @ip }
    assert_equal 200, response.status
    assert_equal "#{@ip} and version one", response.body
  end

  test '/v2 returns ip and v2' do
    get '/v2/zombies', {}, { 'REMOTE_ADDR' => @ip }
    assert_equal 200, response.status
    assert_equal "#{@ip} and version two", response.body
  end
end

# v1/zombies_controller.rb
module V1
  class ZombiesController < ApplicationController
    def index
      render json: "", status: 200
    end
  end
end

# v2/zombies_controller.rb
module V2
  class ZombiesController < ApplicationController
    def index
      render json: "", status: 200
    end
  end
end


# solution code
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
	before_action ->{ @user_ip = request.headers['REMOTE_ADDR'] }
end

# v1/zombies_controller.rb
module V1
  class ZombiesController < ApplicationController
    def index
      render json: "#{@user_ip} and version one", status: 200
    end
  end
end

# v2/zombies_controller.rb
module V2
  class ZombiesController < ApplicationController
    def index
      render json: "#{@user_ip} and version two", status: 200
    end
  end
end

#### Task 6: EXTRACTING DUPLICATE CODE
Now let’s refactor some code that is currently being repeated across different controllers from the same API version. We’ll create a common base controller that’s specific to v2, and move our duplicate code there.

1. Create a base class called `VersionController` under the `V2` namespace. This class should inherit from `ApplicationController`.
2. Specify this base class as being `abstract!`.
3. Now extract the duplicate code out from the other controllers and into the new base class.
4. Update both `V2::HumansController` and `V2::ZombiesController` to inherit from `VersionController`.

```ruby
# task code
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base

  protected

    def log_survival_request
      SurvivalRequestLogger.log!
    end
end

# app/controllers/v2/zombies_controller.rb
module V2
  class ZombiesController < ApplicationController
    before_action -> { log_survival_request }

    def index
      zombies = Zombie.all
      render json: zombies, status: 200
    end
  end
end

# app/controllers/v2/humans_controller.rb
module V2
  class HumansController < ApplicationController
    before_action -> { log_survival_request }

    def index
      humans = Human.all
      render json: humans, status: 200
    end
  end
end

# app/controllers/v2/version_controller.rb
module V2
  # code here
end


# solution code
# task code
# app/controllers/v2/zombies_controller.rb
module V2
  class ZombiesController < VersionController
    def index
      zombies = Zombie.all
      render json: zombies, status: 200
    end
  end
end

# app/controllers/v2/humans_controller.rb
module V2
  class HumansController < VersionController
    def index
      humans = Human.all
      render json: humans, status: 200
    end
  end
end

# app/controllers/v2/version_controller.rb
module V2
  class VersionController < ApplicationController
    abstract!
    before_action -> { log_survival_request }
  end
end
```
