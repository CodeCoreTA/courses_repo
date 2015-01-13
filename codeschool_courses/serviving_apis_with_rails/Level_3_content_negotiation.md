#### Task 1: TESTING WITH MIME TYPE
It’s time to improve the way our API determines the best response representation for different types of clients.
Let’s start by writing a test to ensure our API is able to serve humans resources in JSON.

1. Issue a GET request to the humans resource URI. Use the proper request header to ask for the JSON Mime Type.
2. Assert that the response status is 200 - Success and the response Content-Type is set to JSON.

```ruby
# task code
# test/integration/listing_humans_test.rb
class ListingHumansTest < ActionDispatch::IntegrationTest
  test 'returns humans in JSON' do
  end
end


# solution code
class ListingHumansTest < ActionDispatch::IntegrationTest
  test 'returns humans in JSON' do
    get '/humans', {}, { 'Accept' => Mime::JSON }

  	assert_equal response.status, 200
  	assert_equal Mime::JSON, response.content_type
  end
end
```

#### Task 2: USING RESPOND_TO
We will now move onto production code, where we will add the ability to respond to different formats from our HumansController.

1. Call the respond_to method, which takes a block with a single argument named format.
2. Inside the block, use the format object to respond back with humans in JSON format and with a 200 - Success status code.
3. Now use the format object to respond back with humans in XML format and with a 200 - Success status code.

```ruby
# task code
# app/controllers/humans_controller.rb
class HumansController < ApplicationController
  def index
    humans = Human.all

    # your code here
  end
end


# solution code
class HumansController < ApplicationController
  def index
    humans = Human.all

    respond_to do |format|
    	format.json { render json: humans, status: 200 }
    	format.xml { render xml: humans, status: 200 }
    end
  end
end
```

#### Task 3: TESTING WITH THE LANGUAGE SET TO ENGLISH
During the Zombie Apocalypse we were contacted by both English and Portuguese speaking human survivors. Unfortunately, our API doesn’t support responses in multiple languages yet. We will need to change that if we want to communicate with everyone. In order to do this, we need to make sure our application accepts different language options as part of each request. Let’s start with some tests.

1. Issue a GET request to the humans resources URI. Specify the accepted language as en and the accepted Mime Type as `JSON`.
2. Using `assert_equal`, check for a 200 - Success status code.
3. We’ve selected the first human out of the array for you, and assigned it to the human variable. Using that human, assert its `:message` property is set to `My name is #{human[:name]} and I am alive!`.

```ruby
# task code
# test/integration/changing_locales_test.rb
class ChangingLocalesTest < ActionDispatch::IntegrationTest
  test 'returns list of humans in English' do
    get '/humans', {}, {}
    # assertion here
    human = json(response.body).first
    # assertion here
  end
end


# solution code
class ChangingLocalesTest < ActionDispatch::IntegrationTest
  test 'returns list of humans in English' do
    get '/humans', {}, { 'Accept-Language' => 'en', 'Accept' => Mime::JSON }
    assert_equal response.status, 200
    human = json(response.body).first
    assert_equal "My name is #{human[:name]} and I am alive!", human[:message]
  end
end
```

#### Task 4: SETTING THE LANGUAGE FOR THE RESPONSE
With all tests in place, it is now time to implement our feature. We’ll start with a method that sets the application’s locale with the value from the Accept-Language request header.

1. Create a controller callback that calls the `set_locale` method everytime a new request comes in. Define this method, but don’t worry about implementing it just yet. By following the convention for controller callback methods, mark this method as protected.
2. Now it is time to implement the set_locale method. This method reads from the Accept-Language request header and sets the application’s locale with the value from it.

```ruby
# task code
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
end


# solution code
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
  before_action :set_locale

  protected
  	def set_locale
  		I18n.locale = request.headers['Accept-Language']
  	end
end
```

#### Task 5: USING JBUILDER TO RETURN LOCALIZED JSON
Next, we will use `jBuilder` to implement our view template which returns JSON.

1. Out of each human, extract the properties id, name, and brain_type.
2. Create a member item called message that uses our human_message translation key. Don’t forget to pass the human name as the value for the name option.

```ruby
# task code
# app/controllers/humans_controller.rb
class HumansController < ApplicationController
  def index
    @humans = Human.all
    respond_to do |format|
      format.json
    end
  end
end

# app/views/humans/index.json.jbuilder
json.array(@humans) do |human|
  # your code here
end


# solution code
json.array(@humans) do |human|
  json.extract! human, :id, :name, :brain_type
	json.message I18n.t('human_message', name: human.name)
end
```

#### Task 6: USING LOCALE TEMPLATES
Both English and Portuguese speaking humans will need to be able to understand the message in our response. If you check the Resources tab, the file for Portugese is already filled out for us. The file for English is still pending, so we will need to implement that.

1. Add an entry for `human_message` to the locale file.
2. The value for the `human_message` entry should say, `My name is %{name} and I am still alive!`. Don’t forget about the proper placeholder.

```
# task code
# config/locales/en.yml
en:
  # your code here


# solution code
en:
  human_message: "My name is %{name} and I am still alive!"
```
