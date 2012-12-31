---
layout: post
title: "Fitbit Logins via Omniauth in Rails"
date: 2012-12-31 10:54
comments: true
categories:
- oauth
- omniauth
- omniauth-fitbit
- fitbit
- fitgem
- ruby
- rails
---
It all started with a [comment](http://whazzing.com/blog/2012/03/20/becoming-the-finisher/#comment-743146618) and a [pull request](https://github.com/whazzmaster/fitgem-client/pull/4).

I hadn't put time into maintaining my [fitbit gem](http://github.com/whazzmaster/fitgem) [reference app](http://www.fitbitclient.com) in quite awhile, and when I got a pull request to update it to the latest rails version I at first hesitated. After talking with some folks at [my coworking space](http://www.bendyworks.com), however, I decided to use some holiday downtime to pull in the changes and see if I could build on them to improve the quality of the site.

At first I was horrified to learn that I'd stopped in mid-refactoring and I realized it was no wonder that Marcel (the submitter) reported he'd had trouble getting tests to run. The first step was to delete whole directories of files and tests that no longer conformed to the current design of the site and ensure that the files that remained all contributed to that design.

I grabbed Marcel's pull request and got to work merging it in, at which point I looked at the login code and got embarrassed again. The truth is, the [fitgem](http://github.com/whazzmaster/fitgem) library wasn't really built for managing oauth logins; instead it is optimized for using a token/secret for a given user to fetch data from the API. Yes, you could use fitgem to facilitate an oauth login process, but it certainly wasn't built with rails in mind and you had to roll your own controllers, views, and token and secret management.

The thing is, we already have a library that does that sort of thing: [OmniAuth](https://github.com/intridea/omniauth) is a fantastic library for consuming [pluggable login strategies](https://github.com/intridea/omniauth/wiki/List-of-Strategies). I didn't have to search long before I found a [fitbit strategy for omniauth](https://github.com/tkgospodinov/omniauth-fitbit), and I set to work integrating it into the reference app.

Luckily I had been working a lot with OmniAuth lately, as I'd implemented logins via Facebook and Twitter in another project just the week before. As always I leaned heavily on Ryan Bates' excellent [Railscasts](http://railscasts.com) in getting up to speed on how [OmniAuth worked with Devise](http://railscasts.com/episodes/235-devise-and-omniauth-revised), and specifically with Twitter and [Facebook](http://railscasts.com/episodes/360-facebook-authentication). &lt;small-plug&gt;Railscasts Pro ($9/month) has been totally worth it for me. The Pro episodes go into a lot of depth and I've learned a ton from them.&lt;/small-plug&gt;

Using the omniauth-fitbit gem I was able to delete a lot of now-redundant code for signing into/up for the application. There were, however, a few hiccups along the way that I wanted to note:

* Unlike Twitter and Facebook, logging into Fitbit via omniauth-fitbit doesn't allow devise to remember you from session to session, so you end having to re-login every time. I haven't yet figured out why but it may be something about how Fitbit conducts OAuth logins or it may be a configuration issue with omniauth-fitbit.
* Logging in via Fitbit is fine and dandy, but if you want to use the token/secret for the logged-in user to fetch data from the API through the fitgem interface you'll need to store them. I just added fields to my user object:

{% codeblock Add Oauth fields to User model lang:ruby https://github.com/whazzmaster/fitgem-client/blob/master/db/migrate/20121225193557_add_oauth_fields_to_users.rb _add_oauth_fields_to_users.rb %}
class AddOauthFieldsToUsers < ActiveRecord::Migration
  def change
    add_column :users, :oauth_token, :string
    add_column :users, :oauth_secret, :string
  end
end
{% endcodeblock %}

* I made some, ahem, unorthodox decisions that will have to refactored later. Chief among them was to add the FitbitClient to the User model:

{% codeblock Access to FitbitClient through User model lang:ruby https://github.com/whazzmaster/fitgem-client/blob/master/app/models/user.rb user.rb %}
class User < ActiveRecord::Base

  # Elided for conciseness

  def linked?
    oauth_token.present? && oauth_secret.present?
  end

  def fitbit_data
    raise "Account is not linked with a Fitbit account" unless linked?
    @client ||= Fitgem::Client.new(
                :consumer_key => ENV["FITBIT_CONSUMER_KEY"],
                :consumer_secret => ENV["FITBIT_CONSUMER_SECRET"],
                :token => oauth_token,
                :secret => oauth_secret,
                :user_id => uid
              )
  end

  def has_fitbit_data?
    !@client.nil?
  end

  # Elided for conciseness

end
{% endcodeblock %}

Eventually I should use a better pattern for this, but for now it's simplistic and anywhere you have a logged-in user in your application you will be able to call `fitbit_data` to get access to a configured FitbitClient instance.

And with that we're logging in via Fitbit and using the token/secret returned via omniuath-fitbit to fuel data retrieval from the Fitbit API through fitgem.

### List of Resources

* [OmniAuth](https://github.com/intridea/omniauth): Base authentication library
* [omniauth-fitbit](https://github.com/tkgospodinov/omniauth-fitbit): Fitbit-specific pluggable OmniAuth strategy
* [fitgem](http://github.com/whazzmaster/fitgem): Fitbit API library
* Fitgem Reference Application ([app](http://fitbitclient.com)|[code](https://github.com/whazzmaster/fitgem-client)): An example application using omniauth-fitbit and fitgem
* [Fitbit.com Developer Site](http://dev.fitbit.com): Manage your developer keys for use with omniauth-fitbit and fitgem
* [OmniAuth and Devise Railscast](http://railscasts.com/episodes/235-devise-and-omniauth-revised): Integrating OmniAuth, Devise, and Twitter
* [Facebook Authentication with OmniAuth Railscast](http://railscasts.com/episodes/360-facebook-authentication): Setting up OmniAuth to work with Facebook