Preferences
-----------

This guide covers how to manage and configure preferences within Spree.
After reading it, you should be familiar with:

-   How preferences are managed by Spree
-   How to change the value of a preference from its default
-   How to add a new preference to the Spree core
-   How to create a new set of preferences for your custom extensions.

endprologue.

### Overview

Spree Preferences support general application configuration and
preferences per model instance. Spree comes with preferences for your
store like site name and description. Additional preferences can be
added by your application or included extensions.

Preferences for models can be added without modifying the database. All
instances will use the default value unless a value has been set for a
specific record. For example, you could add a preference to User for
“e-mail notifications”. Users would have the ability to modify this
value without adding a column to your database table.

Extensions may add to the Spree General Settings or create their own
namespaced preferences.

The first several sections of this guide describe preferences in a very
general way. If you’re just interested in making modifications to the
existing preferences, you can skip ahead to the [Configuring Spree
Preferences](#configuring-spree-preferences) section. If you would like
a more in-depth understanding of the underlying concepts used by the
preference system, please read on.

### Motivation

Preferences for models within an application are very common. Although
the rule of thumb is to keep the number of preferences available to a
minimum, sometimes it’s necessary if you want users to have optional
preferences like disabling e-mail notifications.

General Settings for an application like “site name” are also needed to
customize your Spree store.

Both use cases are handled by the Spree Preferences. They are easy to
define, provide quick cached reads, persist across restarts and do not
require additional columns to be added to your models.

The system was heavily refactored in Spree 1.0 for performance.

### General Settings

Spree comes with many application wide preferences. They are defined in
core/app/models/spree/app\_configuration.rb and made available to your
code through Spree::Config, e.g., **Spree::Config.site\_name**.

A limited set of the general settings are available in the admin
interface (/admin/general\_settings).

You can add additional preferences under the spree/app\_configuration
namespace or create your own subclass of **Preferences::Configuration**.

<shell>\
 \# These will be saved with key: spree/app\_configuration/hot\_salsa\
 Spree::AppConfiguration.class\_eval do\
 preference :hot\_salsa, :boolean\
 preference :dark\_chocolate, :boolean, :default =\> true\
 preference :color, :string\
 preference :favorite\_number\
 preference :language, :string, :default =\> ‘English’\
 end

\# Spree::Config is an instance of Spree::AppConfiguration\
 Spree::Config.hot\_salsa = false

\# Create your own class\
 \# These will be saved with key: kona/store\_configuration/hot\_coffee\
 Kona::StoreConfiguration \< Preferences::Configuration\
 preference :hot\_coffee, :boolean\
 preference :color, :string, :default =\> ‘black’\
 end

KONA::STORE\_CONFIG = Kona::StoreConfiguration.new\
 puts KONA::STORE\_CONFIG.hot\_coffee

</shell>

### Models

#### Defining Preferences

You can define preferences for a model within the model itself:

<shell>\
 class User \< ActiveRecord::Base\
 preference :hot\_salsa, :boolean\
 preference :dark\_chocolate, :boolean, :default =\> true\
 preference :color, :string\
 preference :favorite\_number, :integer\
 preference :language, :string, :default =\> ‘English’\
 end\
</shell>

In the above model, five preferences have been defined:

-   hot\_salsa
-   dark\_chocolate
-   color
-   favorite\_number
-   language

For each preference, a data type is provided. The types available are:

-   boolean
-   string
-   password
-   integer
-   text

An optional default value may be defined.

### Accessing preferences

Once preferences have been defined for a model, they can be accessed
either using the shortcut methods that are generated for each preference
or the generic methods that are not specific to a particular preference.

#### Shortcut methods

There are several shortcut methods that are generated. They are shown
below.

Query methods:

<shell>\
 user.prefers\_hot\_salsa? \# =\> false\
 user.prefers\_dark\_chocolate? \# =\> false\
</shell>

Reader methods:

<shell>\
 user.preferred\_color \# =\> nil\
 user.preferred\_language \# =\> “English”\
</shell>

Writer methods:

<shell>\
 user.prefers\_hot\_salsa = false \# =\> false\
 user.preferred\_language = ‘English’ \# =\> “English”\
</shell>

Check if a preference is available

<shell>\
 user.has\_preference? :hot\_salsa\
</shell>

#### Generic methods

Each shortcut method is essentially a wrapper for the various generic
methods shown below:

Query method:

<shell>\
 user.prefers?(:hot\_salsa) \# =\> false\
 user.prefers?(:dark\_chocolate) \# =\> false\
</shell>

Reader method:

<shell>\
 user.preferred(:color) \# =\> nil\
 user.preferred(:language) \# =\> “English”

user.get\_preference :color\
 user.get\_preference :language\
</shell>

Write method:

<shell>\
 user.set\_preference(:hot\_salsa, false) \# =\> false\
 user.set\_preference(:language, “English”) \# =\> “English”\
</shell>

#### Accessing all preferences

You can get a hash of all stored preferences by accessing the
*preferences* helper:

<shell>\
 user.preferences \# =\> {“language”=\>“English”, “color”=\>nil}\
</shell>

This hash will contain the value for every preference that has been
defined for the model, whether that’s the default value or one that has
been previously stored.

#### Type and Default

You can access the default value for a preference:

<shell>\
 user.preferred\_color\_default \# =\> ‘blue’\
</shell>

Types are used to generate forms or display the preference. You can also
get the type defined for preference:

<shell>\
 user.preferred\_color\_type \# =\> :string\
</shell>

### Saving preferences

Preferences are saved immediately for models. They are keyed by the id
of the model.

### Configuring Spree Preferences

Up until now we’ve been discussing the general preference system that
was adapted to Spree. This has given you a general idea of what types of
preference features are theoretically supported. Now, let’s start to
look specifically at how Spree is using these preferences for
configuration.

#### Reading the Current Preferences

At the heart of Spree preferences lies the *Spree::Config* constant.
This object provides general access to the configuration settings
anywhere in the application.

These settings can be accessed from initializers, models, controllers,
views, etc.

The *Spree::Config* constant returns an instance of
*Spree::AppConfiguration* which is where the default values for all of
the general Spree preferences are defined.

You can access these preferences directly in code. To see this in
action, just fire up *script/console* and try the following.

<shell>\
 \>\> Spree::Config.site\_name \
 =\> “Spree Demo Site”\
 \>\> Spree::Config.admin\_products\_per\_page\
 =\> 10\
</shell>

The above examples show the default configuration values for these
preferences. The defaults themselves are coded within the
*Spree::AppConfiguration* class.

<shell>\
 class Spree::AppConfiguration \< Configuration\
 \#… snip …\
 preference :site\_name, :string, :default =\> ‘Spree Demo Site’\
 \#… snip …\
 end\
</shell>

If you are using the default preferences without any modifications, then
nothing will be stored in the database. If you set a value for the
preference it will save it to spree\_preferences. It will use a memory
cached version to maintain performance.

#### Overriding the Default Preferences

The default Spree preferences in *Spree::AppConfiguration* can be
changed using the *set* method of the *Spree::Config* module. For
example to set the default locale for Spain we could do the following:

<shell>\
 \$ script/console\
 \>\> Spree::Config.admin\_products\_per\_page = 20\
 =\> 20\
 \>\> Spree::Config.admin\_products\_per\_page\
 =\> 20\
</shell>

Here we are changing a preference to something other then the default as
specified in *Spree::AppConfiguration*. In this case the preference
system will persist the new value in the *spree\_preferences* table.

INFO: Console settings may be lost if the database is rebuilt. See the
[advice above](#persisting-modificationsto-preferences) about
preservation of settings.

#### Configuration Through the Spree Initializer

During the Spree installation process an initializer file is created
within your applications source code. The initializer is found under
*config/initializers/spree.rb*:

<ruby>\
 Spree.config do |config|\
 \# Example:\
 \# Uncomment to override the default site name.\
 \# config.site\_name = “Spree Demo Site”\
 end\
</ruby>

The *Spree.config* block acts as a shortcut to setting *Spree::Config*
multiple times. If you have multiple default preferences you would like\
to override within your code you may override them here. Using the
initializer for setting the defaults is a nice shortcut, and helps keep
your\
preferences organized in a standard location.

For example if you would like to change the site name and default locale
you can accomplish this by doing the following:

<ruby>\
 Spree.config do |config|\
 config.site\_name = ‘My Awesome Spree Site’\
 config.admin\_products\_per\_page = 20\
 end\
</ruby>

INFO: Initializing preferences available within the Admin will overwrite
any changes that were made through the user interface when you restart.

#### Configuration Through the Admin Interface

The Spree admin interface has several different screens where various
settings can be configured. For instance, the *admin/general\_settings*
URL in your Spree application can be used to configure the values for
the site name and the site URL. This is basically equivalent to calling
*Spree::Config.set site\_name =\> “Whatever”, :site\_url =\>
“http://whatever.com”)* directly in your Ruby code.

### Site-Wide Preferences

#### Defining Site-Wide Preferences

You can define preferences that are site wide and don’t apply to a
specific instance of a model by creating a configuration file that
inherits from Spree::Preferences::Configuration.

<ruby>\
 class Spree::MyApplicationConfiguration \<
Spree::Preferences::Configuration\
 preference :theme, :string, :default =\> ‘Default’\
 preference :show\_splash\_page, :boolean\
 preference :number\_of\_articles, :integer\
 end\
</ruby>

In the above configuration file, three preferences have been defined:

-   theme
-   show\_splash\_page
-   number\_of\_articles

It is recommended to create the configuration file in the **lib/**
directory.

NOTE: Extensions can also define site-wide preferences. For more
information on using preferences like this with extensions, check out
the [Creating Extensions Guide](creating_extensions.html).

#### Configuring Site-Wide Preferences

The recommended way to configure site wide preferences is through an
initializer. Let’s take a look at configuring the preferences defined in
the previous configuration example.

<ruby>\
 module Spree\
 Spree::MyApp::Config = Spree::MyApplicationConfiguration.new\
 end

Spree::MyApp::Config[:theme] = “blue\_theme”\
 Spree::MyApp::Config[:show\_spash\_page] = true\
 Spree::MyApp::Config[:number\_of\_articles] = 5\
</ruby>

The above example will configure the preferences we defined earlier.
Take note of the second line. In order to set and get preferences using
Spree::MyApp::Config, we must first instantiate the configuration
object.

<!-- hack comment since guides gem doesn't allow you to end with INFO, NOTE, etc. -->

