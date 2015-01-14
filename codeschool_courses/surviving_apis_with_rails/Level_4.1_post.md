#### Task 1: INTEGRATION TESTS FOR POST
Our web API needs an end point to register humans who have survived the Zombie Apocalypse. We will start by writing some integration tests for the POST method. These tests will ensure that only valid humans can be created and that our API generates the proper response. Use the following data for creating a valid human:
`{ human: { name: 'John', brain_type: 'small' } }.to_json`.

1. Use the `post` method to issue a request to the humans resource URI. The request will need to include valid human data as its second argument. The third argument will need to send in a hash that tells the server that our request expects the response to be in JSON, and that the payload we are sending is also in JSON.
2. Assert the response status code is `201` - Created.

```ruby
# task code
# test/integration/creating_humans_test.rb
class CreatingHumansTest < ActionDispatch::IntegrationTest
  test 'creates human' do
    # code here
  end
end


# solution code
class CreatingHumansTest < ActionDispatch::IntegrationTest
  test 'creates human' do
    attributes = { human: { name: 'John', brain_type: 'small' } }.to_json

    post '/humans', attributes, { 'Accept' => Mime::JSON, 'Content-Type' => Mime::JSON.to_s }
    assert_equal response.status, 201
  end
end
```

#### Task 2: BETTER ASSERTIONS
We need our test to be a little bit more precise. Checking for a `201` - Created status code is a good start, but it’s not sufficient. Let’s improve our assertions.

1. Assert the `Content-Type` response header is `Mime::JSON`.
2. Now, assert the Location response header is a URL that points to the newly created human resource. You will need to parse the response body, so check the `test/test_helper.rb` file on the secondary tab for a helper method that can help you save some time.

```ruby
# task code
# test/integration/creating_humans_test.rb
class CreatingHumansTest < ActionDispatch::IntegrationTest
  test 'creates human' do
    post '/humans', { human: { name: 'John', brain_type: 'small' } }.to_json,
      { 'Accept' => Mime::JSON, 'Content-Type' => Mime::JSON.to_s }

    assert_equal 201, response.status
    # your code here
  end
end


# solution code
class CreatingHumansTest < ActionDispatch::IntegrationTest
  test 'creates human' do
    post '/humans', { human: { name: 'John', brain_type: 'small' } }.to_json,
      { 'Accept' => Mime::JSON, 'Content-Type' => Mime::JSON.to_s }

    assert_equal 201, response.status
    assert_equal Mime::JSON, response.content_type

    human = json(response.body)
    assert_equal human_url(human[:id]), response.location
  end
end
```

#### Task 3: RESPONDING TO A SUCCESSFUL POST REQUEST
Let’s make our previous tests pass by implementing the create action and creating a new human record.

1. Use `human_params` to build a new Human and save it to the database.
2. Using the JSON format, render back the new human with a 201 - Created status code.
3. Set the Location response header to the newly created human resource.

```ruby
# task code
# app/controllers/humans_controller.rb
class HumansController < ApplicationController
  def create
    # your code here
  end

  private

  def human_params
    params.require(:human).permit(:name, :brain_type)
  end
end


# solution code
class HumansController < ApplicationController
  def create
    human = Human.create!(human_params)

    if human.save
    	render json: human, status: 201, location: human
    end
  end

  private

  def human_params
    params.require(:human).permit(:name, :brain_type)
  end
end
```

#### Task 4: FORGERY PROTECTION
Our API is stateless. This means we don’t need to worry about managing sessions between requests, or exceptions caused by invalid authenticity tokens.

1. Change the `protect_from_forgery` method to null out the session, in case of invalid authenticity tokens.

```ruby
# task code
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
end


# solution code
class ApplicationController < ActionController::Base
  protect_from_forgery with: :null_session
end
```

#### Task 5: POSTING DATA WITH CURL
Let’s issue some real network calls to verify that our API is able to respond to POST requests. Using curl, POST the data `human[name]=Ash` to the following `url: http://localhost:3000/humans`.

```
# solution code
curl -i -X POST -d 'episode[title]=ZombieApocalypseNow' http://localhost:3000/episodes
```

#### Task 6: POST METHOD
Benchmarks have pointed out that our API response to creating new humans is too big. One way we can make the response smaller (and faster), is by returning an empty response body.

1. Still using render, change our action to respond with an empty body and a 204 - No Content status code.
2. Set the Location response header to the newly created human.

```ruby
# task code
# app/controllers/humans_controller.rb
class HumansController < ApplicationController
  protect_from_forgery with: :null_session

  def create
    human = Human.new(human_params)

    if human.save
      render json: human, status: 201
    end
  end

  private

  def human_params
    params.require(:human).permit(:name, :brain_type)
  end
end


# solution code
class HumansController < ApplicationController
  protect_from_forgery with: :null_session

  def create
    human = Human.new(human_params)

    if human.save
      render nothing: true, status: 204, location: human
    end
  end

  private

  def human_params
    params.require(:human).permit(:name, :brain_type)
  end
end
```

#### Task 7: USING HEAD FOR HEADERS-ONLY RESPONSE
Responding with an empty body solved our performance issue. Now let’s go back and refactor our response to be a bit more expressive.

1. Use the head method to respond with a 204 - No Content status code.
2. Set the Location response header to the newly created human.

# task code
# app/controllers/humans_controller.rb
class HumansController < ApplicationController
  protect_from_forgery with: :null_session

  def create
    human = Human.new(human_params)

    if human.save
      # code here
    end
  end

  private

  def human_params
    params.require(:human).permit(:name, :brain_type)
  end
end


# solution code
class HumansController < ApplicationController
  protect_from_forgery with: :null_session

  def create
    human = Human.new(human_params)

    if human.save
      head 204, location: human
    end
  end

  private

  def human_params
    params.require(:human).permit(:name, :brain_type)
  end
end
```

#### Task 8: TESTING UNSUCCESSFUL POST REQUESTS
We want to be sure that our API does not allow creating humans with invalid data. In our tests, we’ll intentionally try and create a new human with no name to make sure our server reponds with the proper error.

1. On our human’s attribute, set the name property to nil.
2. Add an assertion that checks for the response status code of 422 - Unprocessable Entity.

# task code
# app/models/human.rb
class Human < ActiveRecord::Base
  validates :name, presence: true
end

# test/integration/creating_humans_test.rb
class CreatingHumansTest < ActionDispatch::IntegrationTest
  test 'does not create human with name nil' do
    post '/humans',
      { human:
        { name: 'Johnny', brain_type: 'large' }
      }.to_json,
      { 'Accept' => Mime::JSON, 'Content-Type' => Mime::JSON.to_s }

    assert_equal Mime::JSON, response.content_type
  end
end


# solution code
class CreatingHumansTest < ActionDispatch::IntegrationTest
  test 'does not create human with name nil' do
    post '/humans',
      { human:
        { name: nil, brain_type: 'large' }
      }.to_json,
      { 'Accept' => Mime::JSON, 'Content-Type' => Mime::JSON.to_s }

    assert_equal Mime::JSON, response.content_type
    assert_equal 422, response.status
  end
end
```

#### Task 9: RESPONDING TO UNSUCCESSFUL POST REQUESTS
For unsuccessful POST requests, our API needs to respond with the errors that prevented the request from being fulfilled, along with the proper status code.

1. If a new human cannot be saved, then respond with the errors in JSON.
2. Now set the 422 - Unprocessable Entity status code when responding with the errors.

# task code
# app/controllers/humans_controller.rb
class HumansController < ApplicationController
  def create
    human = Human.new(human_params)

    if human.save
      head 204, location: human
    else
      # code here
    end
  end

  private

  def human_params
    params.require(:human).permit(:name, :brain_type)
  end
end



# solution code
class HumansController < ApplicationController
  def create
    human = Human.new(human_params)

    if human.save
      head 204, location: human
    else
      render json: human.errors, status: 422
    end
  end

  private

  def human_params
    params.require(:human).permit(:name, :brain_type)
  end
end
```
