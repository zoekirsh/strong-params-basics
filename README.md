# Strong Params Basics

## What are Strong Params?

To understand the goal of strong params, let's pretend that you run a pharmacy. What would happen if you let all prescription orders come through without checking for: valid prescriptions, driver licenses, etc.? (Spoiler alert: you'd probably end up in jail). It would be insane to run a pharmacy without verifying that orders were legitimate. In the same way Rails application (starting in Rails 4+) wanted to shore up some security vulnerabilities and now require that you whitelist the parameters that are permitted when you're sending form data to the database.


## Setup

To prevent confusion, in previous lessons I manually turned off the strong parameter requirement, to start this lesson I've gone with the Rails 4+ default that requires you to use strong parameters. If for any reason you ever need to turn them off you can add the line `config.action_controller.permit_all_parameters = true` to the `config/application.rb` file.


## Code Implementation

Let's begin by running the rails server and navigating to `localhost:3000/posts/new`, once there fill out the form and click `submit`. You'll see we get the following `ForbiddenAttributesError`:

![ForbiddenAttributesError](https://s3.amazonaws.com/flatiron-bucket/readme-lessons/ForbiddenAttributesError.png)

What this means is that Rails needs to be told what parameters are allowed to be submitted through the form to the database. Remembering back to our pharmacy example, this is essentially like a pharmacist not giving out a prescription out unless they have:

1. A valid prescription from a doctor

2. The customer requesting the item matches the name on the prescription

The same error would occur if you were trying to update a record, so how do we fix this? Let's update the `create` and `update` methods to look like the code below:

```ruby
# app/controllers/posts_controller.rb

def create
  @post = Post.new(params.require(:post).permit(:title, :description))
  @post.save
  redirect_to post_path(@post)
end

def update
  @post = Post.find(params[:id])
  @post.update(params.require(:post).permit(:title, :description))
  redirect_to post_path(@post)
end
```

If you go back to the web browser and click refresh you'll see everything is working for both the `create` and `update` actions. Running the Rspec tests reveals that our specs are now passing again as well.


## DRYing up Strong Params

The code we wrote above is great if you only have a `create` method in your controller, however if you have a standard CRUD setup you will also need to implement the same code in your `update` action. It's a standard Rails practice to remove code repetition, so let's abstract the strong parameter call into its own method in the controller:

```ruby
# app/controllers/posts_controller.rb

private

  def post_params
    params.require(:post).permit(:title, :description)
  end
```

Now, both our `create` and `update` methods in the `posts` controller can simply call `post_params`.

```ruby
# app/controllers/posts_controller.rb

def create
  @post = Post.new(post_params)
  @post.save
  redirect_to post_path(@post)
end

def update
  @post = Post.find(params[:id])
  @post.update(post_params)
  redirect_to post_path(@post)
end
```

This is a very helpful method since if you duplicated the strong parameter call in both the `create` and `update` methods you would need to change both method arguments every time you change the database schema for the `posts` table... and that sounds like a bad way to live. However by creating this `post_params` method we can simply make one change and both methods will automatically be able to have the proper attributes whitelisted.

Test this out in the browser and you can see that you can now create and updated posts without any errors. And you will also notice that all of the Rspec tests are still passing.