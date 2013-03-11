Creating Extensions
-------------------

This guide covers the technical details of extensions and is oriented
towards developers. See the [customization guide](customization.html)
for a more basic treatment on how to customize Spree.

After this guide you should understand:

-   The important role extensions play in Spree
-   How to install a third party extension
-   How to write your own custom extensions
-   How to share your extensions with others

endprologue.

### Extension Basics

There are a few basic steps to know when it comes to creating your own
extension. We’ll walk you through how to create, maintain and test your
extensions in the next few sections.

#### Integrated vs. Standalone

Extensions can be created in their own standalone directory (like the
ones shared on GitHub) or they can be created within your Rails
application. We refer to the extensions that sit in an existing Rails
application as “integrated” extensions.

Integrated extensions are useful during the active development phase of
your store. Instead of switching projects all of the time you can just
make little tweaks to your extension inside of your store repo.
Generally you would only do this if you intended to one day share the
extension with someone else (or reuse it elsewhere in a private
project.)

INFO: The distinction between standalone and integrated extensions is
not very important. The instructions for this guide can be assumed to
apply to both cases (unless stated otherwise.)

NOTE: You may want to consider the integrated approach if the extension
is potentially useful on another project but you don’t want to make the
extension publicly available for others to use.

#### Creating

You can create a new extension using the *spree* command. This requires
that you have a gem version of Spree installed that is version 0.70.0 or
greater. Once you’ve navigated to the directory in which you wish to
create the extension, you can create a new extension by typing the
following in your console:

<shell>\
\$ spree extension foofah\
</shell>

If you’re inside of a Rails application that uses Spree, you may have to
prefix this command with *bundle exec* so that the correct version of
Spree is used:

<shell>\
\$ bundle exec spree extension foofah\
</shell>

This approach relies on an executable script inside the installed gem.
You can always build and install the edge gem using the source if you
want to work with the latest and greatest version of the extension
generator.

NOTE: See the [Source
Code](source_code.html#building-the-gem-from-the-source) guide which
explains how to build the gem from the source.

You probably also need to create a test application (inside the
extension) which is necessary for using the Rails generators or running
the tests.

<shell>\
\$ cd foofah\
\$ bundle exec rake test\_app\
</shell>

NOTE: The test app is not necessary when you have an integrated
extension. The Rails app that you are integrated with will suffice.

#### Resources

Once you’ve created your extension you can now create resources in the
typical Rails way. For example, to create a new resource SomeBar you
would use the standard Rails generator.

<shell>\
\$ rails g resource some\_bar\
</shell>

This will create a typical Rails resource with output similar (not
identical) to the following:

<shell>\
invoke active\_record\
create db/migrate/20110907191441\_create\_some\_bars.rb\
create app/models/some\_bar.rb\
invoke rspec\
create spec/models/some\_bar\_spec.rb\
invoke controller\
create app/controllers/some\_bars\_controller.rb\
invoke erb\
create app/views/some\_bars\
invoke rspec\
create spec/controllers/some\_bars\_controller\_spec.rb\
invoke helper\
create app/helpers/some\_bars\_helper.rb\
invoke rspec\
create spec/helpers/some\_bars\_helper\_spec.rb\
invoke assets\
invoke js\
create app/assets/javascripts/some\_bars.js\
invoke css\
create app/assets/stylesheets/some\_bars.css\
 route resources :some\_bars\
</shell>

If you’re working on an integrated extension then you should make sure
your *Gemfile* includes the following before attempting to generate a
new resource.

<shell>\
gem ‘rspec-rails’\
</shell>

#### Testing

You can test your extensions in the manner you’re accustomed to with a
standard rails application

<shell>\
\$ bundle exec rake test\_app\
\$ bundle exec rspec spec\
</shell>

NOTE: The *rake test\_app* step is not necessary for integrated
extensions.

### Sharing Your Extension

#### Publishing Your Source

The first order of business is to get your extension on
[GitHub](https://github.com) where everybody can see it. Most
importantly, you will want to allow others to have access to the source
code. GitHub provides a convenient (and free) place to store your source
code along with the ability to track issues and accept code patches.

INFO: It is convention to use the *spree-* naming convention for your
GitHub repository and *spree\_* for your gem name. So for example, if
you are creating a “foofah” extension the GitHub project would be named
*spree-foofah* and the gem would be *spree\_foofah*.

#### Publishing Your Gem

If your extension is ready to be released into the wild you can publish
it as a gem on [RubyGems.org](http://rubygems.org). Assuming you used
the extension generator to build your extension, it’s already a gem and
ready to be published. You’ll just want to edit a few details in your
gemspec file before you proceed.

<ruby>\
s.name = ‘foofah’\
s.version = ‘1.0.0’\
s.summary = ‘Add gem summary here’\
\#s.description = ‘Add (optional) gem description here’\
s.required\_ruby\_version = ‘\>= 1.8.7’

1.  s.author = ‘David Heinemeier Hansson’
2.  s.email = ‘david@loudthinking.com’
3.  s.homepage = ‘http://www.rubyonrails.org’
4.  s.rubyforge\_project = ‘actionmailer’\
    </ruby>

#### Registering Your Extension

In order for your extension to show up in the [Extension
Registry](http://spreecommerce.com/extensions) you need to register it.
To do so, sign into your spreecommerce.com account and navigate to the
[My Extensions](http://spreecommerce.com/account/extensions) page. You
may have to create an account first, or you can log-in with GitHub.

#### Adding the Extension to Your Application

To use your extension with your application, you will need to add it to
the *Gemfile* of your application like this:

<ruby>\
gem ‘spree\_flag\_promotions’, :path =\> ‘../spree\_flag\_promotions’\
</ruby>

NOTE: This is a relative path to the ‘spree\_flag\_promotions’
directory. You could also refer to a Git location or any other
convention that is acceptable to bundler. See the [bundler
site](http://gembundler.com) for more details.

<shell>\
\$ bundle install\
</shell>

If your extension has any migrations, they can be copied over from the
extension into your application (along with any other migrations from
other railties) by running this command:

<shell>\
\$ bundle exec rake railties:install:migrations\
</shell>

To run those migrations, run this command:

<shell>\
\$ bundle exec rake db:migrate\
</shell>

That’s it! Your store should be ready to go with your new custom
extension.

### Versionfile

Spree is moving at a rapid pace with the code constantly evolving as new
releases are being pushed out. These changes sometimes make it difficult
to keep track of which version of Spree is compatible with which version
of an extension. To solve this problem extension authors are encouraged
to add a *Versionfile* to their extension.

#### Versionfile Basics

*Versionfile* is a plain text file which keeps information about which
version of Spree an extension is compatible with. It was inspired by the
*Gemfile* used by bundler. Let’s take a look at a sample *Versionfile*.

<ruby>\
“0.50.x” =\> { :branch =\> “master” }\
“0.40.x” =\> { :tag =\> “v1.0.0”, :version =\> “1.0.0” }\
</ruby>

The above *Versionfile* is saying that if you are using Spree version
“0.50.x” then use “master” branch. If you are using Spree version
“0.40.x” then you should use tag “v1.0.0”.

You should note that we are dealing with two different concepts of
“version” here. One is the version of Spree supported and the other is
the version of the extension.

INFO: All Spree extensions should have a version. This version number
has nothing to do with what version of Spree is supported. It is listed
purely as a convenient way to refer to an extension. For example, “I’m
having a problem with spree\_active\_shipping 1.0.1”

#### Syntax

A *Versionfile* can have any number of lines. It is recommended that you
put the latest Spree releases at the top. The very first thing a line
should include is the Spree version it is supporting. Next, within curly
braces, how to get the extension supporting the specified Spree version.

There are four different ways of specifying an extension’s compatibility
information.

NOTE: All four variations for identifying extension compatibility can be
mapped to more than one Spree version. In other words, it’s entirely
possible that the same version of an extension can be compatible with
multiple versions of Spree.

**Gem Version**

<ruby>\
“0.30.x” =\> { :version =\> “1.2” }\
</ruby>

In the above case, if you are using Spree version “0.30.x” then use the
extension as a gem and the version of gem you should be using is “1.2”.

Gem versions are considered “stable.” They will show up in the extension
registry as such.

**Git Branch**

<ruby>\
“0.30.x” =\> { :branch =\> “0-30-stable” }\
</ruby>

In the above example, if you are using Spree version “0.30.x” then use
the branch named “0-30-stable”. Notice that when you suggest the
“branch” the “version” should not be specified.

Versions identified by Git branch are considered “unstable” or “edge.”
They will be identified as such in the extension registry.

**Git Tag**

<ruby>\
“0.30.x” =\> { :tag =\> “v0.30”, :version =\> ‘1.1’ }\
</ruby>

In this example, if you are using Spree version “0.30.x” then use the
tag named “v0.30”. The version information indicates that the author of
the extension is calling that release as version “1.1”.

Versions identified by tags are considered “stable.” They will show up
in the extension registry as such. This is also a nice alternative to
having to release an extension as an actual gem.

**Git SHA**

<ruby>\
“0.30.x” =\> { :ref =\> “4aedfg”, :version =\> ‘1.2’ }\
</ruby>

In the above case, if you are using Spree version “0.30.x” then use ref
named “4aedfg” . The version information indicates that the author of
the extension is calling that release as version “1.2”

Versions identified by a Git SHA are considered “stable.” They will show
up in the extension registry as such. This is also a nice alternative to
having to release an extension as an actual gem.

<ruby>\
\#“0.30.x” =\> { :ref =\> “4aedfg”, :version =\> ‘1.2’ }\
</ruby>

The last example indicates how you can comment out a line. A line
beginning with a hash “\#” is treated as a comment.

#### Using the Versionfile

*Versionfile* has no impact on the actual ability of an extension to
work with Spree. It is simply a declaration by the extension author that
a certain version of the extension is known to work with a particular
version(s) of Spree. It’s also possible that an extension version might
be compatible with versions of Spree not listed. If you discover this to
be the case please send a pull request to the extension author so they
can update their *Versionfile*.

There is a crawler which crawls the *Versionfile* of all the registered
extensions once every day. So any changes you make to your extension’s
*Versionfile* should be captured within 24 hours.

INFO: You must actually register your extension in the [Extension
Registry](http://spreecommerce.com/extensions) before it can be
discovered by the crawler.

After the *Versionfile* info has been processed you will see it listed
in the [Extension Registry](http://spreecommerce.com/extensions).
Depending on the syntax you used to specify the version you will see it
listed under “Stable Release” or “Dev Release”. In both cases there will
also be a link labeled “Show”. Clicking that link will show you the
exact information you need to paste into your *Gemfile* in order to use
the extension.

#### Troubleshooting

While parsing the lines gathered from *Versionfile* if a line is invalid
then that line is skipped. So make sure that entries in *Versionfile*
are valid. We’ll soon be releasing a tool that can verify the
*Versionfile* and provide instant feedback if something is not right.

### Testing Against a Release Candidate

The Spree core team will often announce a so-called [release
candidate](http://en.wikipedia.org/wiki/Software_release_life_cycle#Release_candidate)
shortly before an official release. This is to allow Spree developers to
test their sites and extensions against the upcoming release code.

In order to test your extension against the release candidate you will
need to first update your gemspec file. Let’s look at how we would
update the [spree\_social](https://github.com/spree/spree_social) gem to
use the new 0.60.0.rc1 version of Spree. We’ll need to modify
*spree\_social.gemspec* as follows:

<ruby>\
Gem::Specification.new do |s|\
 …\
 s.add\_dependency(‘spree\_core’, ‘\>= 0.60.0.rc1’)\
 s.add\_dependency(‘spree\_auth’, ‘\>= 0.60.0.rc1’)\
 …\
end\
</ruby>

You’ll also want to update your *Gemfile* that belongs to the Spree
application using the extension.

<ruby>\
 gem ‘spree’, ‘0.60.0.rc1’\
</ruby>

Next, you’ll need to update the dependencies locally using bundler and
commit the resulting *Gemfile.lock* to your source code repository.

<shell>\
\$ bundle update spree\_social\
</shell>

Now it’s time to fire up Spree and make sure everything is working
properly. Assuming everything looks good you’ll also want to update the
*Versionfile* associated with your extension so that people can use the
new and improved version.

In our example we’re adding support for a new 0.60.0RC1 release
candidate which is equivalent to 0.60.x in the extension directory.

<ruby>\
“0.60.x” =\> { :tag =\> “v1.2”, :version =\> “1.2” }\
“0.50.x” =\> { :tag =\> “v1.0.2”, :version =\> “1.0.2” }\
</ruby>

If we’re not 100% sure the extension will support the release candidate
(or if it’s a work in progress) we could reference the “edge” branch
(i.e. master) in the *Versionfile* instead.

<ruby>\
“0.60.x” =\> { :branch =\> “master” }\
“0.50.x” =\> { :tag =\> “v1.0.2”, :version =\> “1.0.2” }\
</ruby>

INFO: Using a branch in the *Versionfile* designates the gem as a
“development release” which is possibly unstable and subject to change.
Once things are stable you can point to a specific Git SHA or tag so
that people can rely on the extension not changing in a production
release.

### Extension Tutorial

This tutorial assumes a basic familiarity with Spree extensions. For
more detailed information on how to use extensions, please see the
previous sections. This tutorial will, however, walk you through a
complete example touching on all of the major aspects of an extension.
If you like to learn through “step by step” instructions you may want to
start here.

#### Getting Started

Let’s start by building a simple extension. Suppose we want the ability
to mark certain products as part of a promotion. We’d like to add an
admin interface for marking certain items as being part of the
promotion. We’d also like to highlight these products in our store view.
This is a great example of how an extension can be used to build on the
solid Spree foundation. We’ll be adding our own custom models, views,
controllers, routes and locales via the new extension.

We’re going to assume you already have a functioning Spree application.
If you have not yet achieved this you should read the [Getting Started
Guide](getting_started.html) first.

So let’s start by generating the new extension

<shell>\
 \$ spree extension FlagPromotions\
</shell>

This creates a *spree\_flag\_promotions* directory with several
additional files and directories as the following generator output
shows:

<shell>\
 create spree\_flag\_promotions\
 create spree\_flag\_promotions/app\
 create
spree\_flag\_promotions/app/assets/javascripts/admin/spree\_flag\_promotions.js\
 create
spree\_flag\_promotions/app/assets/javascripts/store/spree\_flag\_promotions.js\
 create
spree\_flag\_promotions/app/assets/stylesheets/admin/spree\_flag\_promotions.css\
 create
spree\_flag\_promotions/app/assets/stylesheets/store/spree\_flag\_promotions.css\
 create spree\_flag\_promotions/app/controllers\
 create spree\_flag\_promotions/app/helpers\
 create spree\_flag\_promotions/app/models\
 create spree\_flag\_promotions/app/views\
 create spree\_flag\_promotions/config\
 create spree\_flag\_promotions/db\
 create spree\_flag\_promotions/lib\
 create spree\_flag\_promotions/lib/spree\_flag\_promotions.rb\
 create spree\_flag\_promotions/lib/spree\_flag\_promotions/engine.rb\
 create spree\_flag\_promotions/lib/spree\_flag\_promotions/hooks.rb\
 create spree\_flag\_promotions/script\
 create spree\_flag\_promotions/script/rails\
 create spree\_flag\_promotions/spec\
 create spree\_flag\_promotions/LICENSE\
 create spree\_flag\_promotions/Rakefile\
 create spree\_flag\_promotions/README\
 create spree\_flag\_promotions/.gitignore\
 create spree\_flag\_promotions/spree\_flag\_promotions.gemspec\
 create spree\_flag\_promotions/Versionfile\
 create spree\_flag\_promotions/config/routes.rb\
 create spree\_flag\_promotions/Gemfile\
 create spree\_flag\_promotions/spec/spec\_helper.rb\
 create spree\_flag\_promotions/.rspec\
</shell>

#### Creating a Resource

Let’s create a new *PromotedItem* resource for our extension. Since
Spree lets us take advantage of normal Rails generators we can do this
pretty easily.

First we’ll make sure we have a test app created.

<shell>\
\$ cd spree\_flag\_promotions\
\$ bundle exec rake test\_app\
</shell>

INFO: The test app is needed as a context for the Rails generators and
your extension tests to run in. It is not needed if you are creating a
so-called integrated extension.

Now we’re ready to create the resource

<shell>\
\$ rails g resource promoted\_item\
</shell>

This should generate output similar to the following:

<shell>\
 invoke active\_record\
 create db/migrate/20110907202658\_create\_promoted\_items.rb\
 create app/models/promoted\_item.rb\
 invoke rspec\
 create spec/models/promoted\_item\_spec.rb\
 invoke controller\
 create app/controllers/promoted\_items\_controller.rb\
 invoke erb\
 create app/views/promoted\_items\
 invoke rspec\
 create spec/controllers/promoted\_items\_controller\_spec.rb\
 invoke helper\
 create app/helpers/promoted\_items\_helper.rb\
 invoke rspec\
 create spec/helpers/promoted\_items\_helper\_spec.rb\
 invoke assets\
 invoke js\
 create app/assets/javascripts/promoted\_items.js\
 invoke css\
 create app/assets/stylesheets/promoted\_items.css\
 route resources :promoted\_items\
</shell>

#### Verifying the Results

The *PromotedItem* model will represent products that are “flagged” for
promotion on the front page along with a schedule of when the promotion
begins and ends.

<ruby>\
\# app/models/promoted\_item.rb\
class PromotedItem \< ActiveRecord::Base

end\
</ruby>

And of course there’s also the corresponding migration.

<ruby>\
\# db/migrate/20110907202658\_create\_promoted\_items.rb\
class CreatePromotedItems \< ActiveRecord::Migration\
 def change\
 create\_table :promoted\_items do |t|

t.timestamps\
 end\
 end\
end\
</ruby>

INFO: The exact timestamp used in the filename will differ on your
system depending on when you generate it.

There’s also a new controller file so that we can manage these promoted
items.

<ruby>\
\#app/controllers/promoted\_items\_controller.rb\
class PromotedItemsController \< ApplicationController\
end\
</ruby>

NOTE: You may want to add this controller to the *Admin* namespace as
well as extend *Admin::ResourceController*. This step is omitted in
order to keep the tutorial as simple as possible.

Finally, the routes have also been configured for us

<ruby>\
\#config/routes.rb\
Rails.application.routes.draw do\
 resources :promoted\_items\
 \# Add your extension routes here\
end\
</ruby>

#### Customizing an Existing Spree View

Please see the [View Customization](view_customization.html) pages for
details.

#### Internationalization (I18n)

Spree extensions can also provide their own locales/translations. If
you’re adding additional view text and you wish to support multiple
locales, or if you are interested in sharing your extension with others,
it’s a good idea to enable your extension with the i18n features of
Spree.

In order to properly display the “Promoted Items” tab we’ll need to
provide an English localization in the *config/locales/en.yml* file.

<ruby>\
 —\
 en:\
 promoted\_items: Promoted Items\
</ruby>

#### Overriding an Existing Spree Class

Please refer to the [Logic Customization](logic_customization.html)
page.

#### Defining Extension Preferences

Let’s now define a preference for our extension.

<ruby>\
 \#lib/spree/flag\_promotion\_configuration.rb\
 class Spree::FlagPromotionConfiguration \<
Spree::Preferences::Configuration\
 preference :show\_flags, :boolean\
 end\
</ruby>

The above example defines a single boolean preference for our extension,
:show\_flags.

In order to configure these preferences within the extension, we must
first instantiate the configuration object in our engine file.

<ruby>\
 \#lib/spree\_flag\_promotions/engine.rb\
 module Spree::ActiveShipping; end

….

module SpreeFlagPromotions\
 class Engine \< Rails::Engine\
 initializer “spree.flag\_promotions.preferences”, :after =\>
“spree.environment” do |app|\
 Spree::FlagPromotions::Config = Spree::FlagPromotionConfiguration.new\
 end

….\
 end\
 end\
</ruby>

Now this preference can be accessed using
**Spree::FlagPromotions::Config**

<ruby>\
 Spree::FlagPromotions::Config[:show\_flags] = false\
 Spree::FlagPromotions::Config[:show\_flags] \#=\> false\
</ruby>

INFO: Please refer to the
[Preferences](preferences.html#site_wide_preferences) guide for more
information.

#### Adding the Extension to Your Application

Let’s now add the extension to our application. We’ll create a simple
Spree application for this purpose just to illustrate how it’s done.

We’ll start by creating a new Rails app

<shell>\
\$ rails new my-store\
</shell>

Then edit the Gemfile and add dependencies for Spree and the
spree\_flag\_promotions extension

<ruby>\
\#Gemfile

gem ‘spree’\
gem ‘spree\_flag\_promotions’, :path =\> ‘../spree\_flag\_promotions’

</ruby>

NOTE: This is a relative path to the ‘spree\_flag\_promotions’
directory. You could also refer to a Git location or any other
convention that is acceptable to bundler. See the [bundler
site](http://gembundler.com) for more details.

<shell>\
\$ bundle install\
</shell>

Now that the extension is setup it’s time to install (and then run) the
migrations.

<shell>\
\$ bundle exec rake railties:install:migrations\
\$ bundle exec rake db:migrate\
</shell>

That’s it! Your store should be ready to go with your new custom
extension.
