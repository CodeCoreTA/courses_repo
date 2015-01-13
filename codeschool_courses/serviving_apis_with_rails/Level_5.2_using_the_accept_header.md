#### Task 1: TESTING ROUTES FOR HEADER VERSIONING
New requirements have come in and we need to change our API to support versioning through a custom Mime Type. This new Mime Type is going to be called zombies and will read from a specific request header. Let’s start by writing some integration tests.

1. Set the proper request header used for versioning, with the value for our new custom media type of `application/vnd.zombies.v1+json`.
2. Assert the Content-Type on the response is set to JSON.
3. Now parse the response body and assert that there is a message property set to `"This is version one"`

```ruby
# task code
# test/integration/listing_zombies_test.rb
class ListingZombiesTest < ActionDispatch::IntegrationTest
  test 'show zombie from API version 1' do
    get '/zombies/1', {}, { 'Accept' => '' }
    assert_equal 200, response.status
  end
end


# solution code
class ListingZombiesTest < ActionDispatch::IntegrationTest
  test 'show zombie from API version 1' do
    get '/zombies/1', {}, { 'Accept' => 'application/vnd.zombies.v1+json' }
    assert_equal 200, response.status
    assert_equal Mime::JSON, response.content_type

    zombie = json(response.body)
    assert_equal "This is version one", zombie[:message]
  end
end
```

#### Task 2: API VERSION ROUTE CONSTRAINT
Now that we have tests in place, we’ll implement the ApiVersion class. This class will read the `Mime` Type from the Accept request header and then check it against a specific API version that we will pass as the first argument to the constructor.

1. The `default_version` argument on the constructor needs to have a default value of false.
2. The `check_headers` method needs to read from the Accept header, which can return either nil or an array. We’ll return true if it’s not nil and if it includes `application/vnd.zombies.#{@version}+json`.

```ruby
# task code
# lib/api_version.rb
class ApiVersion

  def initialize(version, default_version) # Task 1
    @version, @default_version = version, default_version
  end

  def matches?(request)
    @default_version || check_headers(request.headers)
  end

  private
    def check_headers(headers)
      # Task 2
    end
end


# solution code
class ApiVersion

  def initialize(version, default_version = false)
    @version, @default_version = version, default_version
  end

  def matches?(request)
    @default_version || check_headers(request.headers)
  end

  private
    def check_headers(headers)
      accept = headers['Accept']
      accept && accept.include?("application/vnd.zombies.#{@version}+json")
    end
end
```

#### Task 3: ROUTE VERSION CONSTRAINT
With our ApiVersion class in place, all that’s left to do is use this class on our routes file. We’ll need create two ApiVersion objects - one for each API version that we need to support. Don’t forget to indicate that version 2 is the default API version!

1. First, require the `ApiVersion` class file, which lives under the `lib` folder. The lib folder is currently added to the Rails load path.
2. Set the `v1` module constraint to a new object from the `ApiVersion` class. This object should be initialized to version v1.
3. Set the `v2` module constraint to a new object from the `ApiVersion` class. This object should be initialized to version v2 and this is the default API version.

```ruby
# task code
# config/routes.rb
SurvivingRails::Application.routes.draw do
  scope defaults: { format: 'json' } do
    scope module: :v1, constraints: ... do # Task 2
      resources :zombies
    end
    scope module: :v2, constraints: ... do # Task 3
      resources :zombies
    end
  end
end


# solution code
require 'api_version'

SurvivingRails::Application.routes.draw do
  scope defaults: { format: 'json' } do
    scope module: :v1, constraints: ApiVersion.new('v1') do
      resources :zombies
    end
    scope module: :v2, constraints: ApiVersion.new('v2', true) do
      resources :zombies
    end
  end
end
```

#### Task 4: DEFAULT VERSION TEST
Let’s update our previous RoutesTest file to verify the default version is being used for requests that do not send along a specific API version.

1. The first argument to the `assert_generates` method is the URI for the zombies resources.
2. Now complete the second argument to ensure the previous URI generates the route for `V2::Zombies#index`.

```ruby
# task code
# routes_test.rb
class RoutesTest < ActionDispatch::IntegrationTest
  test 'defaults to v2' do
    assert_generates '', # Task 1
      { controller: '', action: '' } # Task 2
  end
end


# solution code
class RoutesTest < ActionDispatch::IntegrationTest
  test 'defaults to v2' do
    assert_generates '/zombies',
    	{ controller: 'v2/zombies', action: 'index' }
  end
end
```

