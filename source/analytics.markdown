Spree Analytics
---------------

Spree Analytics provides deep insight into your store’s ecommerce
performance and sales conversion funnel. Visitor data is collected to
report on visitor behavior, orders, abandoned carts, revenue and
conversion rates. The data is analyized, rolled up and archived by the
service saving valuable compute resources for your store.

You will have to register your store with Spree Commerce to obtain your
token but the service is free to use.

endprologue.

### Registration

You will have to [register your
store](http://spreecommerce.com/stores/new) with Spree Commerce. This
requires a user account.

On [spreecommerce.com](http://spreecommerce.com/stores), configure the
Analytics Add On for your store to generate your token. The service
requires a strict url in the format “http://www.mystore.com”. This does
not have to be a live url and may be changed later.

Your will need the App Id, Site Id and Token to configure your Spree
Store. The token should be entered on the Overview tab in the admin
section. (/admin). See screenshots:

![Admin Overview](images/analytics/overview.png "Admin Overview")

![Configuring Admin Token](images/analytics/token.png "Configuring Admin Token")

### Development

You should only enter your token on the production store. If you enter
the token on a development server, the analytics will be skewed. Data
can not be reset on the service.

### Older Versions of Spree

For older versions of Spree (0.70 and 0.60), you can install the Spree
Analytics gem. You will obtain a token by registering with Spree
Commerce but the Site Id, App Id and Token are entered into an
initializer file. Please see the README for additional information.

[https://github.com/spree/spree\_analytics](https://github.com/spree/spree_analytics)

Add the gem to your Gemfile

<ruby>\
 gem ‘spree\_analytics’, :git =\>
“git@github.com:spree/spree\_analytics.git”\
</ruby>

then create an initializer file in
config/initializers/spree\_analytics.rb

<ruby>\
 SpreeAnalytics.app\_id = <APP_ID>\
 SpreeAnalytics.site\_id = <SITE_ID>\
 SpreeAnalytics.token = “<TOKEN>”\
</ruby>

### Legacy Dashboard

The original dashboard (previous to 1.0) has been extracted into the
[Spree Dash gem](https://github.com/spree/spree_dash) .
