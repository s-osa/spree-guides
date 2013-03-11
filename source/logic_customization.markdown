Logic Customization
-------------------

This guide explains how to customize the internal Spree code to meet
your exact business requirements including:

-   Extending and overriding existing Spree models and controllers
-   Changing the output from an existing Spree controller action
-   Customizing the image handling functionality.

Be sure to read the other customization methods available in the
[Customization Overview](customization.html) page.

endprologue.

### Extending Classes

All of Spree’s business logic (models, controllers, helpers, etc) can
easily be extended / overridden to meet your exact requirements using
standard Ruby idioms.

Standard practice for including such changes in your application or
extension is to create a file within the relevant **app/models** or
**app/controllers** directory with the original class name with
**\_decorator** appended.

**Adding a custom method to the Product model:**
app/models/product\_decorator.rb

<ruby>\
 Spree::Product.class\_eval do\
 def some\_method\
 …\
 end\
 end\
</ruby>

**Adding a custom action to the ProductsController:**
app/controllers/products\_controller\_decorator.rb

<ruby>\
 Spree::ProductsController.class\_eval do\
 def some\_action\
 …\
 end\
 end\
</ruby>

NOTE: The exact same format can be used to redefine an existing method.

#### Accessing Product Data

If you extend the Products controller with a new method, you may very
well want to access product data in that method. You can do so by using
the :load\_data before\_filter.

<ruby>\
 Spree::ProductsController.class\_eval do\
 before\_filter :load\_data, :only =\> :some\_action

def some\_action\
 …\
 end\
 end\
</ruby>

NOTE: :load\_data will use params[:id] to lookup the product by its
permalink.

### Overriding Controller Action Responses

With the release of 0.60.0 Spree now supports a new way of overriding or
changing the output of an existing controller’s action without needing
to completely override the method (while also easily avoiding double
render exceptions).

#### respond\_override method

The **respond\_override** method is used to customize the response from
any action, and is built on top of Rails 3’s **respond\_with** method
(that all Spree controllers are now using). It accepts a hash of options
using the following syntax:

<ruby>\
 respond\_override :action\_name =\> { :format =\> \
 { :result =\> lambda { … response … } } }\
</ruby>

-   **:action\_name** - Can be any existing action within a controller
    (i.e. :update, :create, :new), provided that action is using
    respond\_with to define its response.
-   **:format** - The format of the request, (i.e. :html, :json, :js,
    etc). All Spree controllers have a class level **respond\_to**
    method call that defines which formats the controller’s actions will
    respond to.
-   **:result** - Two possible results are available on any given
    response, :success or :failure. :success being the default for most
    actions. However, actions that change or create models will have a
    :failure response if validation fails for the model being updated.
-   **lambda** - The lambda passed contains the actual code to create
    the desired custom response (i.e. render or redirect\_to). A lambda
    must be passed to ensure the code is evaluated at the correct time.

#### Example Usage

If you wanted to render a custom partial for the index action of
ProductsController, you could include the following in your
**app/controllers/products\_controller\_decorator.rb** file.

<ruby>\
 Spree::ProductsController.class\_eval do\
 respond\_override :index =\> { :html =\> { :success =\> lambda { render
:partial =\> “shared/some\_file” } } }\
 end\
</ruby>

Or if you wanted to redirect on the failure to create in
Admin::ProductsController, you would use:

<ruby>\
 Spree::Admin::ProductsController.class\_eval do\
 respond\_override :create =\> { :html =\> { :failure =\> lambda {
redirect\_to some\_url } } }\
 end\
</ruby>

#### Caveats

-   If an action does not use **respond\_with** to define its response
    the **respond\_override** will not work.
-   Some actions contain several **respond\_with** calls so any
    **respond\_override** defined on it will be executed for any of the
    **respond\_with** instances, so it’s important to check the model
    state / logic within the lambda passed to prevent overriding all
    possible responses with the same override.

### Product Images

Spree uses Thoughtbot’s
[paperclip](https://github.com/thoughtbot/paperclip) gem to manage
images for products. All the normal paperclip options are available on
the Image class. If you want to modify the default Spree product and
thumbnail image sizes, simply create an image\_decorator.rb file in your
app model directory, and override the attachment sizes:

<ruby>\
 Spree::Image.class\_eval do\
 attachment\_definitions[:attachment][:styles] = {\
 :mini =\> ‘48x48\>’, \# thumbs under image\
 :small =\> ‘100x100\>’, \# images on category view\
 :product =\> ‘240x240\>’, \# full product image\
 :large =\> ‘600x600\>’ \# light box image\
 }\
 end\
</ruby>

You may also add additional image sizes for use in your templates
(:micro for shopping cart view, for example).

#### Image resizing option syntax

Default behavior is to resize the image and maintain aspect ratio (i.e.
the :product version of a 480×400 image will be 240×200). Some commonly
used options are:\
 \* trailing \#, image will be centrally cropped, ensuring the requested
dimensions\
 \* trailing \>, image will only be modified if it is currently larger
than the requested dimensions. (i.e. the :small thumb for a 100×100
original image will be unchanged)
