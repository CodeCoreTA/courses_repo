#### Task 1: INTEGRATION INTRO
Integration tests help drive the design of our API endpoints and simulate clients interacting with our application. Let’s start writing some integration tests for our humans resources.

1. Our API resources live under their own api subdomain constraint. Add the proper setup code to add subdomain support for our integration tests. Remember, Rails uses example.com as the domain for test runs.
2. If you look inside of our test block, the first thing we must do is to issue a GET request to the humans resources URI.
3. Use `assert_equal` to check for a 200 - Success response status, and `refute_empty` to check for a non-empty response body.

```ruby
# task code
# config/routes.rb
class ListingHumansTest < ActionDispatch::IntegrationTest
  # setup code here

  test 'returns a list of humans' do
    # test code here
  end
end

# test/integration/listing_humans_test.rb
class ListingHumansTest < ActionDispatch::IntegrationTest
  # setup code here

  test 'returns a list of humans' do
    # test code here
  end
end


# solution code
# test/integration/listing_humans_test.rb
class ListingHumansTest < ActionDispatch::IntegrationTest
  # setup code here
  setup { host! 'api.example.com' }

  test 'returns a list of humans' do
    # test code here
    get '/humans'
    assert_equal 200, response.status
    refute_empty response.body
  end
end
```

#### Task 2: LISTING RESOURCES
Now that our initial integration tests are in place, it’s time to write some production code. Inside of `API::HumansController#index`, we’ll need to respond with a json representation of our humans and a 200 - Success status code.

1. Render a json representation of humans. Don’t worry about the status code right now.
2. Now let’s make sure the response includes a 200 - success status code.

```ruby
# task code
# app/controllers/humans_controller.rb
module API
  class HumansController < ApplicationController
    def index
      humans = Human.all

      # start your code here
    end
  end
end


# solution code
module API
  class HumansController < ApplicationController
    def index
      humans = Human.all

      render json: humans, status: 200
    end
  end
end
```

#### Task 3: TEST LISTING RESOURCES WITH QUERY STRINGS
Let’s add the ability to filter the list of humans we get back from our API. Before we implement this feature, we’ll need to write some integration tests.

1. On the setup method, set the host to `api.example.com`.
2. Now, let’s create two humans. Set the first one’s name to Allan with the `brain_type` set to large. Then set the second one’s name to John with the `brain_type` set to small.
3. Issue a GET request to the humans resources URI and pass a query string with `brain_type` set to small. Assert the response status code is 200 - Success.
4. Parse the `response.body` from json into a Ruby hash. Make sure John is included in the body, and that Allan is not.

```ruby
# task code
# test/integration/listing_humans_test.rb
class ListingHumansTest < ActionDispatch::IntegrationTest
  setup { }

  test 'returns a list of humans by brain type' do
    # test code here
  end
end


# solution code
class ListingHumansTest < ActionDispatch::IntegrationTest
  setup { host! 'api.example.com' }

  test 'returns a list of humans by brain type' do
    allan = Human.create!(name: 'Allan', brain_type: 'large')
    john = Human.create!(name: 'John', brain_type: 'small')

    get '/humans?brain_type=small'
    assert_equal response.status, 200

    humans = JSON.parse(response.body, symbolize_names: true)
    names = humans.collect { |h| h[:name] }
    assert_includes names, 'John'
    refute_includes names, 'Allan'
  end
end
```

#### Task 4: RESOURCES WITH FILTER
Now it’s time to write production code and make our tests pass. From `API::HumansController#index`, we need to check for a specific parameter sent via URI query strings.

1. Using the params object, check if `brain_type` is passed in as a parameter. If it is, then narrow down the list of humans to only those with that specific `brain_type`. Don’t forget to assign the new result back to the humans variable.
2. Finally, render a json representation of humans with a 200 - Success* status code.

```ruby
# task code
# app/controllers/api/humans_controller.rb
module API
  class HumansController < ApplicationController
    def index
      humans = Human.all

      # your code here
    end
  end
end


# solution code
module API
  class HumansController < ApplicationController
    def index
      humans = Human.all

      if params[:brain_type]
      	humans = humans.where(brain_type: params[:brain_type])
      end

      render json: humans, status: 200
    end
  end
end
```

#### Taks 5: TEST RETRIEVING DATA FOR ONE HUMAN
We will now add the ability to retrieve one specific human by its id. Let’s start with a test.

1. Create a Human named Ash. Issue a GET request to the humans’ show endpoint using Ash’s id, and assert that the response status is 200 - Success.
2. Parse the `response.body` and assert the name returned matches our recently created human. Check the `test/test_helper.rb` tab for a helper method that might be useful.

```ruby
# task code
# test/test_helper.rb
ENV['RAILS_ENV'] = 'test'
require File.expand_path('../../config/environment', __FILE__)
require 'rails/test_help'

class ActiveSupport::TestCase
  ActiveRecord::Migration.check_pending!
  fixtures :all

  def json(body)
    JSON.parse(body, symbolize_names: true)
  end
end

# test/integration/listing_humans_test.rb
class ListingHumansTest < ActionDispatch::IntegrationTest
  setup { host! 'api.example.com' }

  test 'returns human by id' do
  end
end


# solution code
class ListingHumansTest < ActionDispatch::IntegrationTest
  setup { host! 'api.example.com' }

  test 'returns human by id' do
  	human = Human.create!(name: 'Ash')

  	get "/humans/#{human.id}"

  	assert_equal response.status, 200

  	ash = json(response.body)
    assert_equal human.name, ash[:name]
  end
end
```

#### Task 6: RETURNING ONE HUMAN
With tests in place, now let’s implement the show action for our `API::HumansController`. We’ll fetch a specific Human by its id and return its JSON representation.

1. Find a Human by its id and render it back as JSON.
2. Respond with a 200 - Success status code.

```ruby
# task code
# app/controllers/api/humans_controller.rb
module API
  class HumansController < ApplicationController
    def show
      # code here
    end
  end
end


# solution code
module API
  class HumansController < ApplicationController
    def show
      human = Human.find(params[:id])
      render json: human, status: 200
    end
  end
end
```
