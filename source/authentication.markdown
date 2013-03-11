Custom Authentication
---------------------

This guide covers using a custom authentication setup with Spree, such
as one provided by your own application. This is ideal in situations
where you want to handle the sign-in or sign-up flow of your application
uniquely, outside the realms of what would be possibly with Spree. After
reading this guide, you will be familiar with:

-   Setting up Spree to work with your custom authentication

endprologue.

### Background

Traditionally, applications that use Spree have needed to use the
*Spree::User* model that came with the *spree\_auth* component of Spree.
With the advent of 1.2, this is no longer a restriction. The
*spree\_auth* component of Spree has been removed and is now purely
opt-in. If you have an application that has used the *spree\_auth*
component in the past and you wish to continue doing so, you will need
to add this extra line to your *Gemfile*:

<ruby>\
gem ‘spree\_auth\_devise’, :git =\>
“git://github.com/spree/spree\_auth\_devise”\
</ruby>

By having this authentication component outside of Spree, applications
that wish to use their own authentication may do so, and applications
that have previously used *spree\_auth*‘s functionality may continue
doing so by using this gem.
\
h4. The User Model
\
This guide assumes that you have a pre-existing model inside your
application that represents the users of your application already. This
model could be provided by gems such as
[Devise](https://github.com/plataformatec/devise) or
[Sorcery](https://github.com/NoamB/sorcery). This guide also assumes
that the application that this *User* model exists in is already a Spree
application.
\
This model **does not** need to be called *User*, but for the purposes
of this guide the model we will be referring to **will** be called
*User*. If your model is called something else, do some mental
substitution wherever you see *User*.
\
h3. Initial Setup
\
To begin using your custom *User* class, you must first edit Spree’s
initializer located at *config/initializers/spree.rb* by changing this
line:
\
<ruby>\
Spree.user\_class = “Spree::User”\
</ruby>
\
To this:
\
<ruby>\
Spree.user\_class = “User”\
</ruby>
\
Next, you need to run the custom user generator for Spree which will
create two files. The first is a migration that will add the necessary
Spree fields to your users table, and the second is an extension that
lives at *lib/spree/authentication\_helpers.rb* to the
*Spree::Core::AuthenticationHelpers* module inside of Spree.
\
Run this generator with this command:
\
<shell>\
rails g spree:custom\_user User\
</shell>
\
This will tell the generator that you want to use the *User* class as
the class that represents users in Spree. Run the new migration by
running this:
\
<shell>\
rake db:migrate\
</shell>
\
Next you will need to define some methods to tell Spree where to find
your application’s authentication routes.
\
h3. Authentication Helpers
\
There are some authentication helpers of Spree’s that you will need to
possibly override. The file at *lib/spree/authentication\_helpers.rb*
contains the following code to help you do that:
\
<ruby>\
module Spree\
 module AuthenticationHelpers\
 def self.included(receiver)\
 receiver.send :helper\_method, :spree\_login\_path\
 receiver.send :helper\_method, :spree\_signup\_path\
 receiver.send :helper\_method, :spree\_logout\_path\
 receiver.send :helper\_method, :spree\_current\_user\
 end
\
 def spree\_current\_user\
 current\_person\
 end
\
 def spree\_login\_path\
 main\_app.login\_path\
 end
\
 def spree\_signup\_path\
 main\_app.signup\_path\
 end
\
 def spree\_logout\_path\
 main\_app.logout\_path\
 end\
 end\
end
\
Spree::BaseController.send :include, Spree::AuthenticationHelpers\
ApplicationController.send :include, Spree::AuthenticationHelpers\
</ruby>
\
Each of the methods defined in this module return values that are the
most common in Rails applications today, but you may need to customize
them. In order, they are:
\
\* ***spree\_current\_user***: Used to tell Spree what the current user
of a request is.\
\* ***spree\_login\_path***: The location of the login/sign in form in
your application.\
\* ***spree\_signup\_path***: The location of the sign up form in your
application.\
\* ***spree\_logout\_path***: The location of the logout feature of your
application.
\
NOTE: URLs inside the *spree\_login\_path*, *spree\_signup\_path* and
*spree\_logout\_path* methods **must** have *main\_app* prefixed if they
are inside your application. This is because Spree will otherwise
attempt to route to a *login\_path*, *signup\_path* or *logout\_path*
inside of itself, which does not exist. By prefixing with *main\_app*,
you tell it to look at the application’s routes.
\
You will need to define the *login\_path*, *signup\_path* and
*logout\_path* routes yourself, by using code like this inside your
application’s *config/routes.rb* if you’re using Devise:
\
<ruby>\
devise\_scope :person do\
 get’/login’, :to =\> “devise/sessions\#new”\
 get ‘/signup’, :to =\> “devise/registrations\#new”\
 delete ‘/logout’, :to =\> “devise/sessions\#destroy”\
end\
</ruby>

Of course, this code will be different if you’re not using Devise.
Simply do not use the *devise\_scope* method and change the controllers
and actions for these routes.

You can also customize the *spree\_login\_path*, *spree\_signup\_path*
and *spree\_logout\_path* methods inside
*lib/spree/authentication\_helpers.rb* to use the routing helper methods
already provided by the authentication setup you have, if you wish.

NOTE: Any modifications made to *lib/spree/authentication\_helpers.rb*
while the server is running will require a restart, as wth any other
modification to other files in *lib*.

### The User Model

Once you have specified *Spree.user\_class* correctly, there will be
some new methods added to your *User* class. The first of these methods
are the ones added for the *has\_and\_belongs\_to\_many* association
called “spree\_roles”. This association will retreive all the roles that
a user has for Spree.

The second of these is the *spree\_orders* association. This will return
all orders associated with the user in Spree. There’s also a
*last\_incomplete\_spree\_order* method which will return the last
incomplete spree order for the user. This is used internal to Spree to
persist order data across a user’s login sessions.

The third and fourth associations are for address information for a
user. When a user places an order, the address information for that
order will be linked to that user so that it is available for subsequent
orders.

The next method is one called *has\_spree\_role?* which can be used to
check if a user has a specific role. This method is used internally to
Spree to check if the user is authorized to perform specific actions,
such as accessing the admin section. Admin users of your system should
be assigned the Spree admin role, like this:

<ruby>\
user = User.find\_by\_email(“master@example.com”)\
user.spree\_roles \<\< Spree::Role.find\_or\_create\_by\_name(“admin”)\
</ruby>

To test that this has worked, use the *has\_spree\_role?* method, like
this:

<ruby>\
user.has\_spree\_role?(“admin”)\
</ruby>

If this returns *true*, then the user has admin permissions within
Spree.

Finally, if you are using the API component of Spree, there are more
methods added. The first is the *spree\_api\_key* getter and setter
methods, used for the API key that is used with Spree. The next two
methods are *generate\_spree\_api\_key![](+ and +clear_spree_api_key)*
which will both generate and clear the Spree API key respectively.

### Login link

To make the login link appear on Spree pages, you will need to use a
Deface override. Create a new file at
*app/overrides/auth\_login\_bar.rb* and put this content inside it:

<ruby>\
Deface::Override.new(:virtual\_path =\> “spree/shared/\_nav\_bar”,\
 :name =\> “auth\_shared\_login\_bar”,\
 :insert\_before =\> “li\#search-bar”,\
 :partial =\> “spree/shared/login\_bar”,\
 :disabled =\> false, \
 :original =\> ‘eb3fa668cd98b6a1c75c36420ef1b238a1fc55ad’)\
</ruby>

This override references a partial called “spree/shared/login\_bar”.
This will live in a new partial called
*app/views/spree/shared/\_login\_bar.html.erb* in your application. You
may choose to call this file something different, the name is not
important. This file will then contain this code:

<erb>\
\<% if spree\_current\_user %\>\

<li>
\<%= link\_to t(:logout), spree\_logout\_path, :method =\> :delete %\>

</li>
\<% else %\>\

<li>
\<%= link\_to t(:login), spree\_login\_path %\>

</li>

<li>
\<%= link\_to t(:signup), spree\_signup\_path %\>

</li>
\<% end %\>\
</erb>

This will then use the URL helpers you have defined in
*lib/spree/authentication\_helpers.rb* to define three links, one to
allow users to logout, one to allow them to login, and one to allow them
to signup. These links will be visible on all customer-facing pages of
Spree.

### Signup promotion

In Spree, there is a promotion that acts on the user signup which will
not work correctly automatically when you’re not using the standard
authentication method with Spree. To fix this, you will need to trigger
this event after a user has successfully signed up in your application
by setting a session variable after successful signup in whatever
controller deals with user signup:

<ruby>\
session[:spree\_user\_signup] = true\
</ruby>

This line will cause the Spree event notifiers to be notified of this
event and to apply any promotions to an order that are triggered once a
user signs up.
