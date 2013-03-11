Spree Configuration Options
---------------------------

This section lists all of the configuration options for the current
version of Spree.

NOTE: Spree default configuration options can be viewed in
**spree/core/app/models/app\_configuration.rb**

endprologue.

### *address\_requires\_state*

Will determine if the state field should appear on the checkout page.
Defaults to *true*.

### *admin\_interface\_logo*

The path to the logo to display on the admin interface. Can be different
from *Spree::Config*. Defaults to*/admin/bg/spree\_50.png*

### *admin\_products\_per\_page*

How many products to display on the products listing in the admin
interface. Defaults to 10.

### *allow\_backorder\_shipping*

Determines if an *InventoryUnit* can ship or not. Defaults to *true*.

### *allow\_checkout\_on\_gateway\_error*

Continues the checkout process even if the payment gateway error failed.
Defaults to *false*.

### *allow\_ssl\_in\_development\_and\_test*

Enables SSL support in development and test environments. Defaults to
*false*.

### *allow\_ssl\_in\_production*

Enables SSL support in production environment. Defaults to *true*.

### *allow\_ssl\_in\_staging*

Enables SSL support in production environment. Defaults to *true*.

### *alternative\_billing\_phone*

Determines if an alternative phone number should be present for the
billing address on the checkout page. Defaults to *false*

### *alternative\_shipping\_phone*

Determines if an alternative phone number should be present for the
shipping address on the checkout page. Defaults to *false*

### *always\_put\_site\_name\_in\_title*

Determines if the site name (*Spree::Config*) should be placed into the
title. Defaults to*true*.

### *attachment\_default\_url*

Tells paperclip the form of the URL to use for attachments which are
missing.

### *attachment\_path*

Tells Paperclip the path at which to store images.

### *attachment\_styles*

A JSON hash of different styles that are supported by attachments.

### *attachment\_default\_style*

A key from the list of styles from *Spree::Config+ that is the default
style for images.
\
h3.*auto\_capture+

Depending on whether or not Spree is configured to “auto capture” the
credit card, either a purchase or an authorize operation will be
performed on the card (via the current credit card gateway.) Defaults to
*false*.

### *cache\_static\_content*

Enables or disables caching of static content from
*Spree::ContentController*. Defaults to *true*.

### *checkout\_zone*

Limits the checkout to countries from a specific zone, by name. Defaults
to *nil*.

### *company*

Determines whether or not a field for “Company” displays on the checkout
pages for shipping and billing addresses. Defaults to *false*.

### *create\_inventory\_units*

Determines if inventory units will be created when products are
purchased as part of an order. Defaults to *true*.

### *currency*

The three-letter currency code for the currency that prices will be
dsiplayed in. Defaults to “USD”.

### *currency\_symbol\_position*

The position of the symbol for a currency. Can be either “before” or
“after”

### *display\_currency*

Determines whether or not a currency is displayed with a price. Defaults
to *false*.

### *default\_country\_id*

The default country’s id. Defaults to 214, as this is the id for the
United States within the seed data.

### *default\_meta\_description*

The meta description to include in the *head* tag of the Spree layout.
Defaults to “Spree demo site”

### *default\_meta\_keywords*

The meta keywords to include in the *head* tag of the Spree layout.
Defaults to “spree, demo”.

### *dismissed\_spree\_alerts*

The list of alert IDs that you have dismissed.

### *last\_check\_for\_spree\_alerts*

Stores the last time that alerts were checked for. Alerts are checked
for every 12 hours.

### *layout*

The path to the layout of your application, relative to the *app/views*
directory. Defaults to *spree/layouts/spree\_application*

### *logo*

The logo which to display on your frontend. Defaults to
*admin/bg/spree\_50.png*.

### *max\_level\_in\_taxons\_menu*

The number of levels to descend when viewing a taxon menu. Defaults to
*1*.

### *orders\_per\_page*

The number of orders to display on the orders listing in the admin
backend. Defaults to *15*.

### *prices\_inc\_tax*

Determines if prices are labelled as including tax or not. Defaults to
*false*

### *shipment\_inc\_vat*

Determines if shipments should include VAT calculations. Defaults to
*false*

### *shipping\_instructions*

Determines if shipping instructions are requested or not when checking
out. Defaults to *false*.

### *show\_descendents*

Determines if taxon descendants are shown when showing taxons. Defaults
to *true*.

### *show\_only\_complete\_orders\_by\_default*

Determines if, on the admin listing screen, only completed orders should
be shown. Defaults to *true*

### *show\_zero\_stock\_products*

Determines if zero stock products should be shown along side products
with stock. Defaults to *true*.

### *show\_variant\_full\_price*

Determines if the variant’s full price or price difference from a
product should be displayed on the product’s show page. Defaults to
*false*

### *site\_name*

The name of your Spree Store. Defaults to “Spree Demo Site”

### *site\_url*

The URL for your Spree Store. Defaults to “demo.spreecommere.com”

### *tax\_using\_ship\_address*

Determines if tax information should be based on shipping address,
rather than the billing address. Defaults to *true*

### *track\_inventory\_levels*

Determines if inventory levels should be tracked when products are
purchased in the checkout. This option causes new *InventoryUnit*
objects to be created when a product is bought. Defaults to *true*.

### S3 Support

To configure Spree to upload images to S3, put these lines into
*config/initializers/spree.rb*:

<ruby>\
Spree.config do |config|\
 config.use\_s3 = true\
 config.s3\_bucket = ‘<bucket>’\
 config.s3\_access\_key = “<key>”\
 config.s3\_secret = “<secret>”\
end\
</ruby>

It’s also a good idea to not include the *rails\_root* path inside the
*attachment\_path* configuration option, which by default is this:

<ruby>\
:rails\_root/public/spree/products/:id/:style/:basename.:extension\
</ruby>

To change, this add this line underneath the *s3\_secret* configuration
setting:

<ruby>\
config.attachment\_path =
‘/spree/products/:id/:style/:basename.:extension’\
</ruby>

If you’re using the Western Europe S3 server, you will need to set two
additional options inside this block:

<ruby>\
Spree.config do |config|\
 …\
 config.attachment\_url = “:s3\_eu\_url”\
 config.s3\_host\_alias = “s3-eu-west-1.amazonaws.com”\
end\
</ruby>

And additionally you will need to tell paperclip how to construct the
URLs for your images by placing this code outside the *config* block
inside *config/initializers/spree.rb*:

<ruby>\
Paperclip.interpolates(:s3\_eu\_url) do |attachment, style|\

“\#{attachment.s3\_protocol}://\#{Spree::Config[:s3\_host\_alias]}/\#{attachment.bucket\_name}/\#{attachment.path(style).gsub(%r{\^/},”“)}”\
end\
</ruby>
