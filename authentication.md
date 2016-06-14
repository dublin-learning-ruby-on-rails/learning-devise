![](https://i.imgur.com/SFGB5e5.png)
# Set-up authentication with devise & omniauth

Open the docs for quick reference : 

[devise](https://github.com/plataformatec/devise)
[omniauth](https://github.com/intridea/omniauth)

### Create a new rails app

    rails new learning-devise

### Install devise

Add the devise gem in your ``Gemfile``

    gem 'devise'

Bundle the gems and run devise's install generator.

    bundle install
    rails generate devise:install


### Generate your devise model

     rails generate devise user

Run migrations

    bundle exec rake db:migrate

Copy devise views to your app for later configuration

    rails generate devise:views

### Let's do it

We now have everything ready to use the basics of devise.
Let's create an account controller with a view, this part is going to be accessible to users only !

To restrict a controller or action to logged-in users : 

    before_action :authenticate_user!

To access the current user model : 

    current_user


### Adding an email confirmation

Email configuration is a pain, depending on your internet provider, etc, it might not work or get caugh into spam.

We're going to use the mailcatcher gem to circumvent this issue in development.

In your Gemfile, add : 

    gem 'mailcatcher'

run ``bundle`` to install it.

To let our rails app know that it has to speak with mail catcher, add the following to ``config/environments/development.rb``

    config.action_mailer.delivery_method = :smtp
    config.action_mailer.smtp_settings = { :address => "localhost", :port => 1025 }

Also add a default host to link to : 

    config.action_mailer.default_url_options = {:host => "localhost:3000"}


And run mailcatcher in another tab : 

    bundle exec mailcatcher

It will create a web interface accessible on  http://localhost:1080/, all the emails that you send through your app are going to end up here, handy !

We're going to use the "confirmable" devise module, it needs a few more fields in the database, let's create a migration to add them : 

    rails g migration add_confirmable_to_devise


Here's the migration file recommended by devise : 

    class AddConfirmableToDevise < ActiveRecord::Migration
      # Note: You can't use change, as User.update_all will fail in the down migration
      def up
        add_column :users, :confirmation_token, :string
        add_column :users, :confirmed_at, :datetime
        add_column :users, :confirmation_sent_at, :datetime
        add_column :users, :unconfirmed_email, :string # Only if using     reconfirmable
        add_index :users, :confirmation_token, unique: true
        # User.reset_column_information # Need for some types of updates, but not     for update_all.
        # To avoid a short time window between running the migration and updating     all existing
        # users as confirmed, do the following
        execute("UPDATE users SET confirmed_at = NOW()")
        # All existing user accounts should be able to log in after this.
        # Remind: Rails using SQLite as default. And SQLite has no such function     :NOW.
        # Use :date('now') instead of :NOW when using SQLite.
        # => execute("UPDATE users SET confirmed_at = date('now')")
        # Or => User.all.update_all confirmed_at: Time.now
      end
    
      def down
        remove_columns :users, :confirmation_token, :confirmed_at,     :confirmation_sent_at
        # remove_columns :users, :unconfirmed_email # Only if using reconfirmable
      end
    end

Adapt it depending on if you're using sqlite or mySQL

Run it :

    bundle exec rake db:migrate


Restart your server and you should be all set
