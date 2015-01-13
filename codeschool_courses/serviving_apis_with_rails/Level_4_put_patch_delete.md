#### Task 1: INTEGRATION TESTS WITH PATCH
It’s time to add support for updating existing resources on our API. Some of the work has already been started by previous developers, but they were using the old put method. Let’s finish what they started.

1. Replace the `put` method with the proper method for partial updates.
2. Now add an assertion to ensure that the name of our `@human` has been properly updated after the request. Don’t forget to use the reload method.

```ruby
# task code
# test/integration/updating_humans_test.rb
class UpdatingHumansTest < ActionDispatch::IntegrationTest
  setup { @human = Human.create!(name: 'Robert', brain_type: 'small') }

  test 'successful update' do
    put "/humans/#{@human.id}",
      { human: { name: 'Ash', brain_type: 'large' } }.to_json,
      { 'Accept' => Mime::JSON, 'Content-Type' => Mime::JSON.to_s }

    assert_equal 200, response.status
  end
end


# solution code
class UpdatingHumansTest < ActionDispatch::IntegrationTest
  setup { @human = Human.create!(name: 'Robert', brain_type: 'small') }

  test 'successful update' do
    patch "/humans/#{@human.id}",
      { human: { name: 'Ash', brain_type: 'large' } }.to_json,
      { 'Accept' => Mime::JSON, 'Content-Type' => Mime::JSON.to_s }

    assert_equal 200, response.status
    assert_equal 'Ash', @human.reload.name
  end
end
```

#### Task 2: SUCCESSFUL UPDATES WITH PATCH
Now it’s time to make our tests pass. To do this, we need to write an update action that uses `human_params` to update the attributes for the given human, and returns a successful response when a human is successfully updated.

1. Update the fetched object with `human_params` and return its JSON representation.
2. Set the status code for the response to 200 - Success.

```ruby
# task code
# app/controllers/humans_controller.rb
class HumansController < ApplicationController
  def update
    human = Human.find(params[:id])

    # your code here
  end

  private

  def human_params
    params.require(:human).permit(:name, :brain_type)
  end
end


# solution code
class HumansController < ApplicationController
  def update
    human = Human.find(params[:id])

    if human.update(human_params)
    	render json: human, status: 200
    end
  end

  private

  def human_params
    params.require(:human).permit(:name, :brain_type)
  end
end
```

#### Task 3: UNSUCCESSFUL UPDATES WITH PATCH
Now we need to write tests which ensure that clients cannot update existing humans with invalid data. We’ll intentionally issue a PATCH request with bad data to make sure our server reponds with the proper error.

1. On our human’s attribute, set the `name` property to `nil`.
2. Assert that the response status code is 422 - Unprocessable Entity.

```ruby
# task code
# app/models/human.rb
class Human < ActiveRecord::Base
  validates :name, presence: true
end

# test/integration/updating_humans_test.rb
class UpdatingHumansTest < ActionDispatch::IntegrationTest
  setup { @human = Human.create!(name: 'Robert', brain_type: 'small') }

  test 'unsuccessful update on bad name' do
    patch "/humans/#{@human.id}",
      { human: { name: 'Bobby' } }.to_json,
      { 'Accept' => Mime::JSON, 'Content-Type' => Mime::JSON.to_s }
  end
end


# solution code
class UpdatingHumansTest < ActionDispatch::IntegrationTest
  setup { @human = Human.create!(name: 'Robert', brain_type: 'small') }

  test 'unsuccessful update on bad name' do
    patch "/humans/#{@human.id}",
      { human: { name: nil } }.to_json,
      { 'Accept' => Mime::JSON, 'Content-Type' => Mime::JSON.to_s }

    assert_equal 422, response.status
  end
end
```

#### Task 4: RESPONDING TO UNSUCCESSFUL UPDATES
Complete our update action and render errors when a human cannot be updated.

1. If `human.update` returns false, render the errors back as JSON.
2. Set the reponse status code to 422 - Unprocessable Entity.

```ruby
# task code
# app/controllers/humans_controller.rb
class HumansController < ApplicationController
  def update
    human = Human.find(params[:id])

    if human.update(human_params)
      render json: human, status: 200
    end
  end

  private

  def human_params
    params.require(:human).permit(:name, :brain_type)
  end
end


# solution code
class HumansController < ApplicationController
  def update
    human = Human.find(params[:id])

    if human.update(human_params)
      render json: human, status: 200
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

#### Task 5: INTEGRATION TESTS WITH DELETE
Now it is time to write some integration tests to ensure our API responds properly to `DELETE` requests to the zombies resources. Our previous developers started to write these tests before falling victim to the Zombie Apocalypse. Let’s finish it!

1. Issue a `DELETE` request to the zombie URI that points to Undead Jack.
2. Assert that the response status code is 204 - No Content.

```ruby
# task code
# test/integration/deleting_zombies.rb
class DeletingZombiesTest < ActionDispatch::IntegrationTest
  setup { @zombie = Zombie.create!(name: 'Undead Jack', brain_type: 'large') }

  test 'deletes existing zombie' do
    # your code here
  end
end


# solution code
class DeletingZombiesTest < ActionDispatch::IntegrationTest
  setup { @zombie = Zombie.create!(name: 'Undead Jack', brain_type: 'large') }

  test 'deletes existing zombie' do
    delete "/zombies/#{@zombie.id}"

    assert_equal 204, response.status
  end
end
```

#### Task 6: RESPONDING TO DELETE BY DESTROYING RECORDS
Now let’s make our failing tests pass by implementing the delete action on our `ZombiesController`. This action needs to destroy a zombie and return a 204 - No Content status code.

1. Destroy the existing zombie record from the database.
2. Use the head method to return a response with an empty body and with the status code 204 - No Content.

```ruby
# task code
# app/controllers/zombies_controller.rb
class ZombiesController < ApplicationController
  def destroy
    zombie = Zombie.find(params[:id])

    # your code here
  end
end


# solution code
class ZombiesController < ApplicationController
  def destroy
    zombie = Zombie.find(params[:id])

    if zombie.destroy
    	head 204
    end
  end
end
```

#### Task 7: RESPONDING TO DELETE BY ARCHIVING RECORDS
We’ve decided we don’t want to allow zombies resources to be removed from the database. Instead, they’ll be flagged as archived. Let’s create methods that’ll allow us to archive zombies and also allow us to find unarchived ones.

1. Create an archive method on the Zombie class. This method should set the available property to false and then save the record.
2. Create a `self.find_available` method (class method) that takes an id as its only argument. This method should find a record by its id and with the available property set to true.

```ruby
# task code
# app/models/zombie.rb
class Zombie < ActiveRecord::Base
end


# solution code
class Zombie < ActiveRecord::Base
	def self.find_available(id)
		Zombie.find_by!(id: id, available: true)
	end

	def archive
		self.available = false
		self.save
	end
end
```

#### Task 8: RESPONDING TO DELETE METHODS BY ARCHIVING RECORD
One final step to prevent zombies from being deleted from the database upon `DELETE` requests, is to update the `ZombiesController` to use our newly created methods.

1. Change the destroy action to use the `find_available` class method on the `Zombie` class.
2. Change the destroy action to use the archive method on the zombie instance.
3. Update the show action to use the `find_available` class method on Zombie.

```ruby
# task code
# app/controllers/zombies_controller.rb
class ZombiesController < ApplicationController
  def show
    zombie = Zombie.find(params[:id])
    render json: zombie, status: 200
  end

  def destroy
    zombie = Zombie.find(params[:id])
    zombie.destroy
    head 204
  end
end


# solution code
class ZombiesController < ApplicationController
  def show
    zombie = Zombie.find_available(params[:id])
    render json: zombie, status: 200
  end

  def destroy
    zombie = Zombie.find_available(params[:id])
    zombie.archive
    head 204
  end
end
```
