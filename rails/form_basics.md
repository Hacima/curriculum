# Basic Forms

## Introduction

You should be familiar with forms, both as a normal Internet user and as an HTML programmer who has done the [Introduction to Web Development course](/web-development-101).  But how much do you REALLY know about forms?  It may sound strange, but forms are possibly the most complicated thing about learning web development.  Not necessarily because the code itself is difficult, but because you usually want to build forms that accomplish so many different things at once.

Up until now, we've been thinking about Models in Rails on sort of a one-off basis.  The User model.  The Post model.  Sometimes we've had the models relate to each other via associations, like that a Post can `has_many` Comment objects.  Usually, though, we tend to silo our thoughts to only deal with one at a time.

Now think about a web form to buy an airline ticket.  You probably need to enter your name, address, phone number, email, the airline, the flight number, the flight date, your credit card number, your credit card security number, your card expiration date, your card's zipcode, and a bunch of checkboxes for additional things like trip insurance.  That's a whole lot of different models embedded in one form!  But you still submit it with a single button.  Holy macaroni!

Most forms won't be that long or complicated for you, but it's useful to appreciate all the things you can (and one day will) do with them.  It's incredibly easy to make a basic form so the first thing we'll do is make sure you've got an intimate understanding of how forms are created in HTML and then how Rails offers you some helpers to make your life easier.  We'll cover the way data is structured and sent to the controller until you feel pretty comfortable with that.  Then a later lesson will deal with how to take that basic understanding and make forms handle some more firepower. 

## Points to Ponder

*Look through these now and then use them to test yourself after doing the assignment*


* How can you view what was submitted by a form?
* What is a CSRF Token and why is it necessary?
* How do you generate the token in Rails?
* Why is the `name` attribute of a form input element so important?
* How can you nest attributes under a single hash in `params`?
* Why is this useful?
* What do you have to add/modify in your controller to handle nested `params`?
* What special tags does Rails' `#form_tag` helper give you?
* What is the difference between `#form_tag` and `#form_for` helpers?
* How do you access errors on a failed-to-save model object?
* Which form helper automatically adds markup around errors?
* How do you access your Update or Delete actions with a form?

## Forms in HTML

Step one is to be able to create a form in HTML.  Remember how that looks?

    <form action="/somepath" method="post">
      <input type="text">

      <!-- other inputs here -->

      <input type="submit" value="Submit This Form">
    </form>

There are plenty of `input` tags to choose from, including `button`, `checkbox`, `date`, `hidden`, `password`, `radio` and many more (see [the w3 schools list under the `type` attribute](http://www.w3schools.com/tags/tag_input.asp)).

## Viewing What Your Form Submits

If you want to see what your forms are submitting to your Rails app, look through the output that gets printed into your console when you run your `$ rails server`.  Whenever you submit a very basic form for a user email signup, it should include lines that look something like:

    Started POST "/user" for 127.0.0.1 at 2013-11-21 19:10:47 -0800
    Processing by UsersController#create as HTML
    Parameters: {"utf8"=>"✓", "authenticity_token"=>"jJa87aK1OpXfjojryBk2Db6thv0K3bSZeYTuW8hF4Ns=", "email"=>"foo@bar.com", "commit"=>"Submit Form"}

*Note: this is from a form that might be generated by a Rails helper method, as explained in a later section below*

The first line tells us which HTTP method was used and which route the form went to.  The second line tells us which controller and action the form will be handled by.  The third line contains everything that will get stuffed into the `params` hash for the controller to use.  We'll talk about the contents in the next sections.  

You'll find yourself looking at this server output a lot when you start building forms.  It'll keep you sane because it tells you exactly what the browser sent back to your application so you can see if there's been a... misunderstanding.

## Railsifying Your Form

The first thing you'll realize if you try to create a plain vanilla form in a Rails view is that it won't work.  You'll either get an error or your user session will get zeroed out (depending on your Rails version).  That's because of a security issue with [cross-site scripting](http://en.wikipedia.org/wiki/Cross-site_scripting), so Rails requires you to verify that the form was actually submitted from a page you generated.  In order to do so, it generates an ["authenticity token"](http://guides.rubyonrails.org/security.html#cross-site-request-forgery-csrf) which looks like gibberish but helps Rails match the form with your session and the application.

You'll notice the token in the server output from above:

    ...
    Parameters: {"utf8"=>"✓", "authenticity_token"=>"jJa87aK1OpXfjojryBk2Db6thv0K3bSZeYTuW8hF4Ns=", "email"=>"foo@bar.com", "commit"=>"Submit Form"}
    ...

So, if you want to create your own form that gets handled by Rails, you need to provide the token somehow as well.  Luckily, Rails gives you a method called `form_authenticity_token` to do so, and we'll cover it in the project.  

    <input type="hidden" name="authenticity_token" value="<%= form_authenticity_token %>">

## Making Forms into `params`

What about the other form inputs, the ones we actually care about?

Each one of these inputs is structured slightly differently, but there are some commonalities.  One important thing to note is the `name` attribute that you can give to an input tag.  In Rails, that's very important.  The `name` attribute tells Rails what it should call the stuff you entered in that input field when it creates the `params` hash.  For instance,

    ...
    <input type="text" name="description">
    ...

Will result in your `params` hash containing a key called `description` that you can access as normal, e.g. `params[:description]`, inside your controller.  That's also why some inputs like radio buttons (where `type="radio"`) use the `name` attribute to know which radio buttons should be grouped together such that clicking one of them will unclick the others.  The `name` attribute is surprisingly important!

Now another thing we talked about in the controller section was nesting data.  You'll often want to tuck submitted data neatly into a hash instead of keeping them all at the top level.  This can be useful because, as we saw with controllers, it lets you do a one-line `#create` (once you've whitelisted the parameters with `#require` and `#permit`).  When you access `params[:user]`, it's actually a hash containing all the user's attributes, for instance `{:first_name => "foo", :last_name => "bar", :email=>"foo@bar.com"}`.  How do you get your forms to submit parameters like this?  It's easy!

It all comes back to the `name` attribute of your form inputs. Just use hard brackets to nest data like so:

    ...
    <input type="text" name="user[first_name]">
    <input type="text" name="user[last_name]">
    <input type="text" name="user[email]">
    ...

Those inputs will now get transformed into a nested hash under the `:user` key.  The server output becomes:

    Parameters: {"utf8"=>"✓", "authenticity_token"=>"jJa87aK1OpXfjojryBk2Db6thv0K3bSZeYTuW8hF4Ns=", "user"=>{"first_name"=>"foo","last_name"=>"bar","email"=>"foo@bar.com"}, "commit"=>"Submit Form"}

Specific parameters of the `params` hash are accessed like any other nested hash `params[:user][:email]`.  

Don't forget that you have to whitelist the params now in your controller using `require` and `permit` because they are a hash instead of just a flat string.  See the Controller section below for a refresher on the controller side of things.

This is cool stuff that you'll get a chance to play with in the project.

## Form Helpers

Rails tries to make your life as easy as it can, so naturally it provides you with helper methods that automate some of the repetitive parts of creating forms.  That doesn't mean you don't need to know how to create forms the "old fashioned" way... it's actually MORE important to know your form fundamentals when using helpers because you'll need to really understand what's going on behind the scenes if something breaks.

Start by making a form using the `form_tag` helper, which takes a block representing all the inputs to the form.  It takes care of the CSRF security token we talked about above by automatically creating the hidden input for it so you don't have to.  You pass it arguments to tell it which path to submit to (the default is the current page) and which method to use.  Then there are tag helpers that create the specified tags for you, like `text_field_tag` below.  All you need to specify there is what you want to call the field when it is submitted.

    <%= form_tag("/search", method: "get") do %>
      <%= label_tag(:q, "Search for:") %>
      <%= text_field_tag(:q) %>
      <%= submit_tag("Search") %>
    <% end %>

Creates the form:

    <form accept-charset="UTF-8" action="/search" method="get">
      <label for="q">Search for:</label>
      <input id="q" name="q" type="text" />
      <input name="commit" type="submit" value="Search" />
    </form>

The ID of the inputs matches the name.  

There are tag helpers for all the major tags and the options they accept are all a bit different.  See the reading assignment for more detail.

### Handy Shortcuts: `form_for`

No one wants to remember to specify which URL the form should submit to or write out a whole bunch of `*_tag` methods, so Rails gives you a shortcut in the form of the slightly more abstracted `form_for` method.  It's a whole lot like `form_tag` but does a bit more work for you.  

Just pass `form_for` a model object, and it will make the form submit to the URL for that object, e.g. `@user` will submit to correct URL for creating a User.  Remember from the lesson on controllers that the `#new` action usually involves creating a new (unsaved) instance of your object and passing it to the view... now you finally get to see why by using that object in your `#form_for` forms!

Where `form_tag` accepted a block without any arguments and the individual inputs had to be specified with `something_tag` syntax, `form_for` actually passes the block a form object and then you create the form fields based off that object.  It's conventional to call the argument simply `f`. 

From the Rails Guide:

    # app/controllers/articles_controller.rb
    def new
      @article = Article.new
    end

    #app/views/articles/new.html.erb
    <%= form_for @article do |f| %>
      <%= f.text_field :title %>
      <%= f.text_area :body, size: "60x12" %>
      <%= f.submit "Create" %>
    <% end %>

And the generated HTML is:

    <form accept-charset="UTF-8" action="/articles/create" method="post">
      <input id="article_title" name="article[title]" type="text" />
      <textarea id="article_body" name="article[body]" cols="60" rows="12"></textarea>
      <input name="commit" type="submit" value="Create" />
    </form>

Note that this helper nests the Article's attributes (the hard brackets in the `name` attribute should be the dead giveaway).

The best part about `form_for` is that if you just pass it a model object like `@article` in the example above, Rails will check for you if the object has been saved yet.  If it's a new object, it will send the form to your `#create` action.  If the object has been saved before, so we know that we're **edit**ing an existing object, it will send the object to your `#update` action instead.  This is done by automatically generating the correct URL when the form is created.  Magic!

## Forms and Validations

What happens if your form is submitted but fails the validations you've placed on it?  For instance, what if the user's password is too short?  Well, first of all, you should have had some Javascript validations to be your first line of defense and they should have caught that... but we'll get into that in another course.  In any case, hopefully your controller is set up to re-render the current form.  

You'll probably want to display the errors so the user knows what went wrong.  Recall that when Rails tries to validate an object and fails, it attaches a new set of fields to the object called `errors`.  You can see those errors by accessing `your_object_name.errors`.  Those errors have a couple of handy helpers you can use to display them nicely in the browser -- `#count` and `#full_messages`.  See the code below:

    <% if @post.errors.any? %>
      <div id="error_explanation">
        <h2><%= pluralize(@post.errors.count, "error") %> prohibited this post from being saved:</h2>
     
        <ul>
        <% @post.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
        </ul>
      </div>
    <% end %>

That will give the user a message telling him/her how many errors there are and then a message for each error.

The best part about Rails form helpers... they handle errors automatically too!  If a form is rendered for a specific model object, like using `form_for @article` from the example above, Rails will check for errors and, if it finds any, it will automatically wrap a special `<div>` element around that field with the class `field_with_errors` so you can write whatever CSS your want to make it stand out.  Cool!

## Making PATCH and DELETE Submissions

Forms aren't really designed to natively delete objects because browsers only support GET and POST requests.  Rails gives you a way around that by sticking a hidden field named "_method" into your form.  It tells Rails that you actually want to do either a PATCH (aka PUT) or DELETE request (whichever you specified), and might look like `<input name="_method" type="hidden" value="patch">`.

You get Rails to add this to your form by passing an option to `form_for` or `form_tag` called `:method`, e.g.:

    form_tag(search_path, :method => "patch")

## Controller-Side Refresher

Just as a refresher, here's a very basic controller setup for handling `#new` actions and `#create` actions.

    # app/controllers/users_controller.rb
    ...
    def new
      @user = User.new
    end

    def create
      @user = User.new(user_params)
      if @user.save
        redirect_to @user
      else
        render :new
      end
    end

    def user_params
      params.require(:user).permit(:first_name, :last_name, :other_stuff)
    end
    ...

I wanted to show this again so you could remind yourself what's going on in the form's lifecycle.  The user presumably went to the path for the `#new` action, likely `http://www.yourapp.com/users/new`.  That ran the `#new` action, which created a new user object in memory (but not yet saved to the database) and rendered the `new.html.erb` view.  The view probably used `form_for` on `@user` to make things nice and easy for the developer.

Once the form gets submitted, the `#create` action will build another new User object with the parameters we explicitly tell it are okay.  Recall that our custom `#user_params` method will return the `params[:user]` hash for us, which lets `User.new` build us a complete new instance.  If that instance can be saved to the database, we're all good and we go to that user's `show.html.erb` page.

If the `@user` cannot be saved, like because the `first_name` contains numbers, we will jump straight back to rendering the `new.html.erb` view, this time using the `@user` instance that will still have errors attached to it.  Our form should gracefully handle those errors by telling the user where he/she screwed up.

## Your Assignment

1. Read the [Rails Guide on Form Helpers](http://guides.rubyonrails.org/form_helpers.html), sections 1 to 3.2.  
2. Skim 3.3 to 7 to see what kinds of things are out there.  One day you'll need them, and now you know where to look.
3. Read sections 7.1 and 7.2 for the official explanation of how parameters are created from the `name` attribute.
4. Read the [Rails Guide on Validations](http://guides.rubyonrails.org/active_record_validations.html#displaying-validation-errors-in-views) section 8 for a quick look at displaying errors.

## Conclusion

At this point, you should have a solid understanding of how forms work in general and a pretty good feel for how Rails helps you out by generating them for you.  You'll get a chance to build a whole bunch of forms in the next few projects, so don't worry if it's not totally stuck for you yet.  Seeing it in action will make things click.

## Additional Resources

*This section contains helpful links to other content. It isn't required, so consider it supplemental for if you need to dive deeper into something*

* 