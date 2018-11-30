---
layout: post
title: "Hanami with Sidekiq"
date: 2018-11-30 09:05:00 +0100
categories: hanami sidekiq
---

I've been working on a small Hanami app as a side project recently and wanted to use Sidekiq to implement some of the functionality.

Adding Sidekiq to a Rails project is pretty straightforward and adding it to a Hanami application is not any harder.

There is a good guide written by David Strauß on his [blog](https://www.strauss.io/blog/2016-use-sidekiq-with-hanami.html) which I mostly followed.
His guide is a couple of years old now and there are some small changes which I had to discover by searching github issues and the [Hanami Gitter](https://gitter.im/hanami/chat) so I will detail here what I did to get everything working.

## Sidekiq Setup

This part is identical to the process details in David's blog.

Add sidekiq to your gemfile and `bundle install`.

{% highlight ruby %}
# Gemfile
gem 'sidekiq'
{% endhighlight %}

Create a `config/sidekiq.rb` file:

{% highlight ruby %}
# config/sidekiq.rb
Sidekiq.configure_server do |config|
  config.redis = { url: ENV.fetch('REDIS_URL') }
end

Sidekiq.configure_client do |config|
  config.redis = { url: ENV.fetch('REDIS_URL') }
end
{% endhighlight %}

Require Sidekiq in your `config/environment.rb`

{% highlight ruby %}
require_relative '../apps/web/application'
require_relative './sidekiq'
{% endhighlight %}

Then we need to add the `REDIS_URL` to our `.env` files and the setup is complete.

{% highlight ruby %}
# .env.development
REDIS_URL=redis://localhost:6379/0
{% endhighlight %}


## Adding Workers

With the setup complete we can now add our workers. I went back and forth over whether they should go in `apps/web` or `lib` and eventually decided to go with `lib/<app name>/workers`.

If you do put your workers in `apps/web/workers` you will need to add the workers directory to your load paths in `apps/web/application.rb`

Lets create a simple worker that prints whatever is passed to it.


{% highlight ruby %}
# lib/<your app name>/workers/example_worker.rb
class ExampleWorker
  include Sidekiq::Worker

  def perform(argument)
    puts '*'*50
    puts argument
    puts '*'*50
  end
end
{% endhighlight %}

## Running Sidekiq

There is a small change to David's instructions on how to run Sidekiq. We need to pass the `config/boot.rb` file to Sidekiq and not the `config/environment.rb` file.
To run the Sidekiq server we need to use `bundle exec sidekiq -e development -r ./config/boot.rb`. This boots up the Hanami app and gives us access to our Entities, repositories etc.

If we start a Hanami console with `bundle exec hanami c` we can now call our worker:

{% highlight ruby %}
ExampleWorker.perform_async("Hello Sidekiq!")
{% endhighlight %}

Once called we should see output like this in the Sidekiq log:

{% highlight bash %}
**************************************************
Hello Sidekiq!
**************************************************
{% endhighlight %}


## Mounting the Sidekiq Dashboard

I couldn't get David's instructions to work for this part so here is what I did instead.

Add Sinatra to your Gemfile and `bundle install`. You do not need to require it as Sidekiq will require what it needs.

{% highlight ruby %}
# Gemfile
gem 'sinatra', require: false
{% endhighlight %}

Require Sidekiq from the routes file and pass it your session secret environment variable.

{% highlight ruby %}
# apps/web/config/routes.rb
require 'sidekiq/web'
Sidekiq::Web.set :session_secret, ENV['WEB_SESSIONS_SECRET']

# Configure your routes here
# See: http://hanamirb.org/guides/routing/overview/
#
# Example:
# get '/hello', to: ->(env) { [200, {}, ['Hello from Hanami!']] }
mount Sidekiq::Web, at: '/sidekiq'
{% endhighlight %}

I've left the default comments in place to illustrate that I placed this right at the top of the file.

The session secret environment variable needs to be set or requests to cancel or retry jobs etc. you make in the Sidekiq dashboard will error.

If your Hanami app is running on the default port you should now be able to visit `localhost:2300/sidekiq` and see the Sidekiq dashboard.

There it is, Sidekiq running in a Hanami application. You can now go wild with your required background processing workflows.

## Final Note

As it stands the Sidekiq dashboard is not secured in any way. It really should be behind a login of some kind. Whether this is basic auth or the authorisation your app already uses is up to you.

My little side project hasn't got any kind of authentication/authorisation yet so I haven't included any instruction on how to do that, perhaps I'll write a future blog to do that when I get to it.
