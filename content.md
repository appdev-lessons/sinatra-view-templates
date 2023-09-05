# Sinatra: View templates

Now that we have our dice rolling app working, let's learn a few ways to improve our code.

Continue working in your existing sinatra-dice-roll codespace. For reference, [my code currently looks like this](https://github.com/raghubetina/sinatra-dice-roll/tree/cfb7733f66f3e9ab6b91b2183ce1bfa9b9c28698). We'll be building on top of this starting point, so make sure you're caught up.

Use the code in my `dice.rb` file for reference to get your app close to the target (it shouldn't match exactly _yet_):

[dice-roll.matchthetarget.com](https://dice-roll.matchthetarget.com/)

---

Currently, the route for my root URL looks like this:

```ruby
# /dice.rb

get("/") do
  "
  <h1>Dice Roll</h1>
	
  <ul>
    <li><a href=\"/dice/2/6\">Roll two 6-sided dice</a></li>
    <li><a href=\"/dice/2/10\">Roll two 10-sided dice</a></li>
    <li><a href=\"/dice/1/20\">Roll one 20-sided die</a></li>
    <li><a href=\"/dice/5/4\">Roll five 4-sided dice</a></li>
  </ul>
  "
end
```

This action produces a page that looks like this:

 ![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688676994/homepage-with-nav_snzqzu.png)
 
Since the last evaluated expression in the action is what Sinatra sends back as the body of the HTTP response, I created a `String` literal on line 4 with the opening `"`, closed it on line 12 with the closing `"`, and stuck all my HTML inside.

Question: what are all of the back slashes in code for? Why _didn't_ I write it like this instead:

```ruby
get("/") do
  "
  <h1>Dice Roll</h1>
	
  <ul>
    <li><a href="/dice/2/6">Roll two 6-sided dice</a></li>
    <li><a href="/dice/2/10">Roll two 10-sided dice</a></li>
    <li><a href="/dice/1/20">Roll one 20-sided die</a></li>
    <li><a href="/dice/5/4">Roll five 4-sided dice</a></li>
  </ul>
  "
end
```

The above would be invalid Ruby, because:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688839344/sinatra-unescaped-string_wrmnbk.png)

In order to have `"` within our `String`, we had to _escape_ them with a backslash. Then, Ruby knows not to terminate the `String` there, and instead treats the `"` as just another character to be included in the `String`. [We learned about this already, so hopefully it is somewhat familiar](https://learn.firstdraft.com/lessons/113#escape-characters).

The HTML that we want to send back in our responses is usually going to be much longer than this — dozens, hundreds, sometimes thousands of lines of HTML. Writing our HTML within a Ruby `String` literal becomes tedious quickly:

- HTML contains lots of `"` — around attribute values. Escaping them all is a pain.
- We don't get nice HTML syntax highlighting or autocompletions from our editor.
- The Ruby program gets long and unwieldy.
- And a bunch of other headaches.
 
Fortunately, there's a better way: **view templates**.

## Our first view template

Instead of writing our HTML within our Ruby file, we're going to create a separate file for it. These files are called "view templates". Then, we'll load that file from within our action using the `erb` method. Here's how it works:

- Create a folder called `views`. The name `views` is special to Sinatra — we don't get to pick its name.
- Within that folder, create a file called `elephant.erb`. The name `elephant` is _not_ special — we can call the file anything we want to.

  However, the filename must end in the extension `.erb`, which stands for **e**mbedded **r**u**b**y.
- `elephant.erb` is our _view template_. Within it, we can write regular HTML. Let's move the HTML for our homepage into this file:

```erb
<!-- /views/elephant.erb -->

<h1>Dice Roll</h1>

<ul>
  <li><a href="/dice/2/6">Roll two 6-sided dice</a></li>
  <li><a href="/dice/2/10">Roll two 10-sided dice</a></li>
  <li><a href="/dice/1/20">Roll one 20-sided die</a></li>
  <li><a href="/dice/5/4">Roll five 4-sided dice</a></li>
</ul>
```

  We don't have to escape the quotes, because we're not within a Ruby string literal anymore. Go through and remove the back slashes.
- Last, we'll load this file in our action using the `erb` method:

	```ruby
  # /dice.rb
	
  get("/") do
    erb(:elephant)
  end
	```

  The argument to the `erb` method must be a `Symbol` containing the name of a file within the `views/` folder. We don't have to include the `.erb` part of the filename because it's assumed that all view template names will end in `.erb`.

If you visit the root URL of your live app preview, it should work just the same as before.

---

**Checkpoint:**

- [Here you can see the changes that I've made since the last commit.](https://github.com/raghubetina/sinatra-dice-roll/commit/6017140bb84099ca3bb4e24dae23b5385957b7ad)
- [Here you can browse my entire codebase at this point in time.](https://github.com/raghubetina/sinatra-dice-roll/tree/6017140bb84099ca3bb4e24dae23b5385957b7ad)

## Embedding Ruby

Let's try using view templates for our other routes. Let's start by moving the HTML for `GET /dice/2/6` into a template called `two_six.erb`.

We'll replace the big `String` at the bottom of the action with `erb(:two_six)`

```ruby
# /dice.rb

get("/dice/2/6") do
  first_die = rand(1..6)
  second_die = rand(1..6)
  sum = first_die + second_die

  outcome = "You rolled a #{first_die} and a #{second_die} for a total of #{sum}."

  erb(:two_six)
end
```

And move the HTML into a template file:

```erb
<!-- /views/two_six.erb -->

<h1>2d6</h1>

<p>#{outcome}</p>
```

If you visit `/dice/2/6` in your live app preview, you should now see something like this:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688850615/erb-1_axd2od.png)

Not great! This is because Ruby's `String` interpolation operator, `#{ }`, has no special significance in HTML; it's just more copy, as far as HTML is concerned.

However, `.erb` files have one very special superpower relative to `.html` files: you can use **embedded Ruby tags** (or "erb tags") within them.

ERB tags start with an opening `<%=` (no spaces are allowed between the three characters) and end with a closing `%>`. Between those two, you can write any Ruby code. Sinatra will execute the code, take the last evaluated expression, and inject it into the HTML source code before sending out the final HTML response.

For example, try this:

```erb
<!-- /views/two_six.erb -->

<%= rand(100) %>

<h1>2d6</h1>

<p>#{outcome}</p>
```

If you refresh the page, you should now see a random number appear at the very top. If you View Source on the page, you should see something like this:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688851045/erb-2_sjn70m.png)

As far as the browser is concerned, there was never any Ruby there; it still only receives HTML in the response of the body. Sinatra **pre-processes** the ERB template, evaluates all the Ruby, and sends out a clean HTML file.

Let's use an ERB tag instead of the string interplator:

```erb
<!-- /views/two_six.erb -->

<h1>2d6</h1>

<p><%= outcome %></p>
```

However, if you try visiting your live app preview now, you will see an error "undefined local variable or method 'outcome' for #<Sinatra::Application>":

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688851108/erb-3_bvwa4o.png)

The good news is that the ERB tag is working — Sinatra is executing the code inside.

The bad news is that the local variable `outcome` is undefined in `two_six.erb`. We created it back in `dice.rb`, within the block for the route `GET /dice/2/6`. Since it is a local variable, it is local to that block — i.e., it can't be used outside of that block.

To resolve this, we can use an **instance variable** instead. Instance variables can be used anywhere else within the _object_ that they are created in. For our purposes, this means that if we create an instance variable within an action, it will be available to us within whichever view template we call with the `erb` method.

How do we make a variable an instance variable rather than a local variable? We start the variable name with an `@`, rather than a lowercase letter:

```ruby
# /dice.rb

get("/dice/2/6") do
  first_die = rand(1..6)
  second_die = rand(1..6)
  sum = first_die + second_die

  @outcome = "You rolled a #{first_die} and a #{second_die} for a total of #{sum}."

  erb(:two_six)
end
```

And update the view template also:

```erb
<!-- /views/two_six.erb -->

<h1>2d6</h1>

<p><%= @outcome %></p>
```

Now everything should be working again!

Try adding view templates for the remaining actions as well.

---

**Checkpoint:**

- [Here you can see the changes that I've made since the last commit.](https://github.com/raghubetina/sinatra-dice-roll/commit/b4ad6123ef97a6c37f7dec816ffe81121ee73069)
- [Here you can browse my entire codebase at this point in time.](https://github.com/raghubetina/sinatra-dice-roll/tree/b4ad6123ef97a6c37f7dec816ffe81121ee73069)

## Adding a layout

So far, we've been completely neglecting the boilerplate that we need to make our responses valid HTML documents: the doctype, `<html>`, `<head>`, `<body>`, etc. Let's add all that now to `two_six.erb`:

```erb
<!-- /views/two_six.erb -->

<!DOCTYPE html>
<html>
<head>
  <title>Dice Roll</title>
</head>

<body>
  <h1>2d6</h1>

  <p><%= @outcome %></p>
</body>
</html>
```

And add it to all your other view templates, as well. Visit the pages, View Source, and use [the W3C validator](https://validator.w3.org/#validate_by_input) to make sure the code is valid now.

Next, let's add some navigation links to make it easier for our users to get from place to place:

```erb
<!-- /views/two_six.erb -->

<!DOCTYPE html>
<html>
<head>
  <title>Dice Roll</title>
</head>

<body>
  <div>
    <div><a href="/">Home</a></div>
    <div><a href="/dice/2/6">2d6</a></div>
    <div><a href="/dice/2/10">2d10</a></div>
    <div><a href="/dice/1/20">1d20</a></div>
    <div><a href="/dice/5/4">4d5</a></div>
  </div>

  <h1>2d6</h1>

  <p><%= @outcome %></p>
</body>
</html>
```

And add it to all your other view templates, as well. Your pages should now look something like this:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688853233/vertical-nav_jsqary.png)

At this point, you might be noticing that our view templates share 90% of their code; it's just a couple of lines that differ between each of them.

Sinatra provides a great way to reduce this duplication: **layouts**.

When you have some HTML that you want to appear on _every_ template (the standard boilerplate, navbar, footer, etc), we can put it all in one file called a "layout". Create a file named `/views/wrapper.erb` and then stick all of the common view code in it:

```erb
<!-- /views/wrapper.erb -->

<!DOCTYPE html>
<html>
<head>
  <title>Dice Roll</title>
</head>

<body>
  <div>
    <div><a href="/">Home</a></div>
    <div><a href="/dice/2/6">2d6</a></div>
    <div><a href="/dice/2/10">2d10</a></div>
    <div><a href="/dice/1/20">1d20</a></div>
    <div><a href="/dice/5/4">4d5</a></div>
  </div>

  <!-- This is the part that is not the same across all actions-->
	
</body>
</html>
```

Now, we can use this layout when rendering by adding a second argument when calling `erb` method:

```ruby
# /dice.rb

get("/dice/2/6") do
  first_die = rand(1..6)
  second_die = rand(1..6)
  sum = first_die + second_die

  @outcome = "You rolled a #{first_die} and a #{second_die} for a total of #{sum}."

  erb(:two_six, { :layout => :wrapper })
end
```

The second argument to `erb` is a `Hash`. If we include the key `:layout` and provide the name of an ERB template, that ERB template will be used for the response.

If you visit the live app preview right now, you'll only see the contents of `wrapper.erb`; we've lost the contents of the action's view template.

To fix this, we must include the `yield` key word in the layout:

```erb
<!-- /views/wrapper.erb -->

<!DOCTYPE html>
<html>
<head>
  <title>Dice Roll</title>
</head>

<body>
  <div>
    <div><a href="/">Home</a></div>
    <div><a href="/dice/2/6">2d6</a></div>
    <div><a href="/dice/2/10">2d10</a></div>
    <div><a href="/dice/1/20">1d20</a></div>
    <div><a href="/dice/5/4">4d5</a></div>
  </div>

  <%= yield %>
	
</body>
</html>
```

Now if you refresh the live app preview, you should see something like this:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688853848/layout-2_c19azf.png)

The contents of each view template is injected into the layout at the spot where we included the `<%= yield %>`. Since we have the boilerplate in both the template and the layout, it is in source code twice. We can now delete the boilerplate from the template file.

Go through each of your routes, add the `{ :layout => :wrapper }`  to the `erb` call, and delete the boilerplate from the view template. Phew!

Now that we have our shared boilerplate in one place, it's much easier to make updates to it and have those changes be reflected across all of our routes automatically. For example, let's add some CSS to make the navigation appear horizontally:

```erb
<!-- /views/wrapper.erb -->

<!DOCTYPE html>
<html>
<head>
  <title>Dice Roll</title>

  <style>
    .navbar {
      display: flex;
      justify-content: space-between;
    }
  </style>
</head>

<body>
  <div class="navbar">
    <div><a href="/">Home</a></div>
    <div><a href="/dice/2/6">2d6</a></div>
    <div><a href="/dice/2/10">2d10</a></div>
    <div><a href="/dice/1/20">1d20</a></div>
    <div><a href="/dice/5/4">4d5</a></div>
  </div>

  <%= yield %>

</body>
</html>
```

Click around the app and you'll see that the shiny new navbar is there on all routes.

### Default layout name

Since we are going to want to use a layout file for nearly every route, Sinatra makes it a little easier for us:

If we name our layout file `/views/layout.erb`, then we don't have to explicitly add the `:layout` option to the `erb` call; Sinatra will use it by default if it sees a file with that special name.

So, let's rename `wrapper.erb` to `layout.erb`, and then we can delete all of the `{ :layout => :wrapper }` arguments from the `erb` method calls.

Much better! From now on, for all of our apps, the first thing we'll do will be to create a file called `/views/layout.erb` and add the standard HTML boilerplate to it. Then we'll build out our templates in peace without having to worry about adding all the boilerplate to each and every one.

---

**Checkpoint:**

- [Here you can see the changes that I've made since the last commit.](https://github.com/raghubetina/sinatra-dice-roll/commit/b4ad6123ef97a6c37f7dec816ffe81121ee73069)
- [Here you can browse my entire codebase at this point in time.](https://github.com/raghubetina/sinatra-dice-roll/tree/b4ad6123ef97a6c37f7dec816ffe81121ee73069)

---

### Silent ERB tags

There is another variant of ERB tags that we can use: the **silent ERB tag**. It looks like `<%`, rather than the `<%=` of the regular ERB tag — i.e., it doesn't have an `=` sign after the `%`.

The difference: the silent ERB tag doesn't inject anything into the HTML source code. Silent ERB tags are handy when you want to run some Ruby to control flow while generating HTML — in particular, for `if` statements and loops.

For example, for `GET /dice/1/20`, suppose we want the text to be red if the roll is 1, green if the roll is 20, and black for all other rolls. I can achieve this using `if` statements within silent ERB tags.

First, let's modify `dice.rb` to change the local variable `die` into an instance variable `@die`, so that we can use it in the view template:

```ruby
# /dice.rb

get("/dice/1/20") do
  @die = rand(1..20)

  @outcome = "You rolled a #{@die}."

  erb(:one_twenty)
end
```

Now let's modify the view template to use `@die` along with `if`/`elsif`/`else`/`end`:

```erb
<!-- /views/one_twenty.erb -->

<h1>1d20</h1>

<% if @die == 1 %>
  <p style="color: red"><%= @outcome %></p>
<% elsif @die == 20 %>
  <p style="color: green"><%= @outcome %></p>
<% else %>
  <p><%= @outcome %></p>
<% end %>
```

The silent ERB tag allows us to use Ruby in view templates even when we don't want to inject that Ruby's value into the HTML source code.

### Looping in view templates

An even more common use of the silent ERB tag is to loop within a view template.

Let's create a new route, `GET /dice/100/6`. This route should display one hundred rolls of 6-sided dice.

We could create one hundred individual variables, but that would be quite tedious. Let's make an `Array` and populate it using a `.times` loop instead:

```ruby
# /dice.rb

get("/dice/100/6") do
  @rolls = []

  100.times do
    die = rand(1..6)

    @rolls.push(die)
  end

  erb(:one_hundred_six)
end
```

Now, we can use this `Array` , `.each` , and silent ERB tags to efficiently display all 100 values:

```erb
<!-- /views/one_hundred_six.erb -->

<h1>100d6</h1>

<ul>
  <% @rolls.each do |a_roll| %>
    <li>
      <%= a_roll %>
    </li>
  <% end %>
</ul>
```

First, make sure this works. Then:

- Try adding an `=` to the silent ERB tag on line 6 above. What happens?
- Try adding an `=` to the silent ERB tag on line 10 above. What happens?
- Try _removing_ the `=` from the regular ERB tag on line 8 above. What happens?

In your own words, if you had to explain the difference between `<%=` and `<%`, how would you do it?

We're going to be using `.each` within view templates _a lot_. Most of web development involves getting a list of data (usually from our database, but sometimes from an external API), and then looping through that list in a view template to add formatting around each piece of data.

## The target

Try and copy the steps I just demonstrated for the `"/dice/100/6"` route for the other routes and view templates in your project. For example, start with the route `"/dice/2/6"` and the corresponding view template `views/two_six.erb`. 

After you do that your app should look _exactly_ like the target:

[dice-roll.matchthetarget.com](https://dice-roll.matchthetarget.com/)

- I tried the steps and my app looks...
- just like the target!
  - Nice!
- different from the target...
  - Try and go back through the lesson and look for your mistake. Use the links to my code as an example in places. If you need help, reach out to a teacher!
{: .choose_best #match_target title="Match the target" points="1" answer="1" }

---

- Approximately how long (in minutes) did this lesson take you to complete?
{: .free_text_number #time_taken title="Time taken" points="1" answer="any" }

---
