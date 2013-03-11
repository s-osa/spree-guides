Adjustments
-----------

This guide covers the use of adjustments in Spree. After reading it, you
should be familiar with:

-   The important role calculators play in the creation of charges and
    credits.
-   The basic role adjustments play in Spree
-   How to create your own custom calculators and adjustments

endprologue.

### Overview

Adjustments play a crucial role in Spree. They are the means by which an
order total changes to reflect any type of change that is necessary
after calculating the *item\_total*. They are the building blocks for
creating shipping and tax-related charges as well as for applying
discounts for promotions and coupons.

Adjustments can be either positive or negative. Adjustments with a
positive value are sometimes referred to as “charges” while adjustments
with a negative value are sometimes referred to as “credits.” These are
just terms of convenience since there is only one *Spree::Adjustment*
model in Spree which handles this by allowing either positive or
negative values.

INFO: Prior to Spree 0.30.x there were separate *Credit* and *Charge*
models. This distinction has been abandoned since it made things
unnecessarily complicated.

### Totals

Spree has several important total values that are recorded at the
*Spree::Order* level. The following table represents a list of
attributes which are available to the Spree application and stored in
the database for reporting purposes.

  --------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  \_\<.Name             \_.Description
  *item\_total*         The sum total of all line items (*price* \* *quantity* for all line items)
  *adjustment\_total*   The sum total of all adjustments (which can be positive or negative.) Examples include shipping charges and coupon credits.
  *total*               This represents the final total of the order (*item\_total* &\#43; *adjustment\_total*).
  *payment\_total*      The sum total of all valid payments (including possible negative payments due to credit card refunds.) This total automatically excludes payments in the *processing*, *pending* and *failed* states.
  --------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

There are also a few interesting “calculated totals.” These totals are
not stored in the database (and thus not easily searchable) but they can
be displayed on a per record basis through the Ruby methods provided by
*Spree::Order*.

  --------------- --------------------------------------------------------------------------
  \_\<.Name       \_.Description
  *ship\_total*   The sum total of all adjustments where the adjustment label = ‘shipping’
  *tax\_total*    The sum total of all adjustments where the adjustment label = ‘tax’
  --------------- --------------------------------------------------------------------------

### Types of Adjustments

Spree supports several types of adjustments - in fact, it supports an
open-ended number of adjustment types to handle pretty much any type of
situation. From a programming standpoint these are all represented by a
single *Spree::Adjustment* class with a *label* attribute used to
identify what type of adjustment it is.

The following Ruby console output helps to illustrate this fact.

<shell>\
\>\> Spree::Adjustment.last\
=\>
\#<Spree::Adjustment id: 684130, source_id: 968652773, amount: #<BigDecimal:7fff25851228,'0.5E1',9(36)>,
label: “Shipping”, source\_type: “Spree::Shipment”, adjustable\_id: 0,
created\_at: “2012-03-25 04:12:08”, updated\_at: “2012-03-25 04:12:08”,
mandatory: true, locked: true, originator\_id: 574015644,
originator\_type: “Spree::ShippingMethod”, eligible: true,
adjustable\_type: “Spree::Order”\>\
</shell>

This may seem somewhat simplistic at first but our experience has shown
that most scenarios can be handled with adequate labels and a few other
tricks (including providing some handy
[scopes](http://api.rubyonrails.org/classes/ActiveRecord/Scoping.html)
at the model level.)

#### Taxation

Tax adjustments are any adjustments with a label of “Tax.” This is just
a naming convention but it’s a standard enough case that we have a built
in scope for them in Spree. You can use the following console command to
list an order’s tax adjustments:

<shell>\
\>\> Spree::Order.last.adjustments.tax\
=\> [ … ]\
</shell>

INFO: Tax related charges are considered
[frozen](adjustments.html#frozen-adjustments) by default.

#### Shipping

Shipping adjustments are any adjustments with a label of “Shipping.”
This is just a naming convention but it’s a standard enough case that we
have a built in scope for them in Spree. You can use the following
console command to list an order’s shipping adjustments:

<shell>\
\>\> Spree::Order.last.adjustments.shipping\
=\> [ … ]\
</shell>

INFO: Shipping related charges are considered
[frozen](adjustments.html#frozen-adjustments) by default.

#### Coupons and Promotions

[TODO - provide a brief summary and link to new promotions documentation
here once that is complete]

### Calculating Adjustments

Adjustments are typically calculated according to a specified set of
rules (although it is possible to create adjustments with a fixed,
arbitrary amount.) The following sections will detail the use of
*Spree::Calculators* as well as the rules for when calculations are
performed.

#### Calculators

Spree makes extensive use of the *Spree::Calculator* model and there are
several subclasses provided to deal with various types of calculations
(flat rate, percentage discount, sales tax, VAT, etc.) All calculators
extend the *Spree::Calculator* class and must provide the following
methods

<ruby>\
def self.description\
 \# Human readable description of the calculator\
end

def compute(object=nil)\
 \# Returns the value after performing the required calculation\
end\
</ruby>

#### Registration

The core calculators for Spree are stored in the
*app/models/spree/calculator* directory. There are several calculators
included that meet many of the standard store owner needs. Developers
are encouraged to write their own [extensions](extensions.html) to
supply additional functionality or to consider using a [third party
extension](http://spreecommerce.com/extensions) written by members of
the Spree community.

Calculators need to be “registered” with Spree in order to be made
available in the admin interface for various configuration options. The
recommended approach for doing this is via an extension. Custom
calculators will typically be written as extensions so you need to add
some registration logic to the extension containing the calculator. This
will allow the calculator to do a one time registration during the
standard extension activation process.

Spree itself contains a good example of how this can be achieved. For
instance, in the *spree\_core* gem there is logic to register
calculators. The following code can be found in the Rails
*spree.register.calculators* initializer defined in
*spree/core/lib/spree/core/engine.rb*:

<ruby>\
initializer ‘spree.register.calculators’ do |app|\
 app.config.spree.calculators.shipping\_methods = [\
 Spree::Calculator::FlatPercentItemTotal,\
 Spree::Calculator::FlatRate,\
 Spree::Calculator::FlexiRate,\
 Spree::Calculator::PerItem,\
 Spree::Calculator::PriceBucket]

app.config.spree.calculators.tax\_rates = [\
 Spree::Calculator::SalesTax,\
 Spree::Calculator::Vat]\
end\
</ruby>

This registers calculators to the
*Rails.application.config.spree.calculators* array. Extension authors
should be sure to register any new calculators within the
*Rails.application.config.spree.calculators* by defining new class of
aray in *lib/extension\_name/engine.rb*.

<ruby>\
app.config.spree.calculators.add\_class(‘product\_customization\_types’)

app.config.spree.calculators.product\_customization\_types = [ \
 Spree::Calculator::CustomCalculatorOne, \
 Spree::Calculator::CustomCalculatorTwo \
] \
</ruby>

If you do not intend to distribute your calculator as an extension you
can simply register it inside a Rails initializer
*spree.register.calculators* within the *application.rb* Application
class.

<ruby>\
initializer ‘spree.register.calculators’ do |app|\
 app.config.spree.calculators.shipping\_methods \<\<
Spree::Calculator::CustomCalculator\
 app.config.spree.calculators.tax\_rates \<\<
Spree::Calculator::CustomTaxCalculator\
end\
</ruby>

You can also register it using the
*Rails.application.config.spree.calculators* method of the default site
engine.

INFO: Spree automatically configures your calculators for you when using
the basic install and/or third party extensions. This discussion is
intended to help developers and others interested in understanding the
design decisions surrounding calculators.

NOTE: The *register* method is removed. Calculators are now registered
by appending custom array to
*Rails.application.config.spree.calculators*.

#### *Spree::CalculatedAdjustments* Module

Spree includes a helpful *Spree::CalculatedAdjustments* module that can
be used to introduce calculator-like functionality into a Rails model.
Classes that require this type of functionality simply need to declare
*calculated\_adjustments* in their class definition.

<ruby>\
module Spree\
 class ShippingMethod \< ActiveRecord::Base\
 …\
 calculated\_adjustments\
 …\
 end\
end\
</ruby>

##### Class Methods

When a Spree class uses this helper method it automatically adds the
following class functionality:

-   The ability to associate a calculator with an instance of that class
    (through a *has\_one :calculator* association)
-   Validation rules and the ability to accept configuration params
    through Rails *accepts\_nested\_attributes\_for*

Let’s look at how this works with a specific example taken from the
Spree source code itself. Specifically we’ll be looking at
*Spree::ShippingMethod*.

<ruby>\
module Spree\
 class ShippingMethod \< ActiveRecord::Base\
 …\
 calculated\_adjustments\
 …\
 end\
end\
</ruby>

Here we see that *Spree::ShippingMethod* declares that it’s going to be
involved in calculations by declaring *calculated\_adjustments* in the
class definition. What does that give us exactly?

Let’s start by firing up the Ruby console and creating a new instance of
*Spree::ShippingMethod*.

<shell>\
\>\> s = Spree::ShippingMethod.new\
=\>
\#<Spree::ShippingMethod id: nil, name: nil, zone_id: nil, created_at: nil, updated_at: nil, display_on: nil, shipping_category_id: nil, match_none: nil, match_all: nil, match_one: nil>\
</shell>

Now we can verify that this instance of *Spree::ShippingMethod* in fact
has a *Spree::Calculator* association (although it’s currently nil
because we haven’t assigned anything to it.)

<shell>\
\>\> s.calculator\
=\> nil\
</shell>

NOTE: If you are creating new Rails models you should never declare
*has\_one :calculator* in order to take advantage of calculators. Just
use *calculated\_adjustments* instead since you get this association and
several other goodies for free. You’ll also be doing things the standard
Spree way which will make it easier to maintain over the long run.

Prior to Spree 1.0, classes that extend *Spree::Calculator* that
declares *calculated\_adjustments* can be registered with
*self.register* method. This method is removed in favor of registering
classes by via *Rails.application.config.spree.calculators*. Let’s look
at how *spree\_core* engine define the registration in
*lib/extension\_name/engine.rb*:

<ruby>\
initializer ‘spree.register.calculators’ do |app|\
 app.config.spree.calculators.shipping\_methods = [\
 Spree::Calculator::FlatPercentItemTotal,\
 Spree::Calculator::FlatRate,\
 Spree::Calculator::FlexiRate,\
 Spree::Calculator::PerItem,\
 Spree::Calculator::PriceSack\
 ]

….\
end\
</ruby>

You see that the initializer define a class *shipping\_methods* for
*app.config.spree.calculators*. The *shipping\_method* is an array
enlist all shipping calculators classes. This registers the calculator
with Spree but the following line registers the calculator with
*Spree::ShippingMethod* as well.

You may wonder to yourself “why is that important?” The answer is that
you can perform operations such as the following:

<shell>\
\>\> Spree::ShippingMethod.calculators\
=\> { … }\
\>\> Spree::ShippingMethod.calculators.size\
=\> 5\
</shell>

By registering your calculators in this way, then they will become
available as options in the appropriate admin screens.

![Choosing a calculator](images/calculators/choosing_calculator.png "Choosing a calculator")

##### Instance Methods

Use of the *calculated\_adjustments* helper also provides additional
methods to instances of the class declaring it. Specifically they gain
the following instance methods:

  ---------------------- ---------------------------------------------------------------------------------------------------------------------
  \_\<.Method Name       \_.Description
  *create\_adjustment*   Responsible for creating a new adjustment for the specified ‘target’
  *update\_adjustment*   Updates the amount of the adjustment using the instance’s *Calculator*
  *calculator\_type*     Utility method to help with admin screens that need to set the *Calculator* for a particular instance of the class.
  ---------------------- ---------------------------------------------------------------------------------------------------------------------

The *create\_adjustment* is the process by which all adjustments should
be created. Let’s take a look at how this mechanism works inside the
*Spree::Order* class.

<ruby>\
def create\_tax\_charge!\
 \# destroy any previous adjustments (eveything is recalculated from
scratch)\
 adjustments.tax.each { |e| e.destroy }\
 price\_adjustments.each { |p| p.destroy }

Spree::TaxRate.match(self).each { |rate| rate.adjust(self) }\
end\
</ruby>

Here we see that an instance of *Spree::TaxRate* is being used to create
the adjustment for the order. *Spree::TaxRate* declares
*calculated\_adjustments* in its class definition which makes this all
possible.

INFO: You do not need to understand this process to make use of
adjustments in Spree. The detailed information provided here for those
who wish to understand the inner workings of Spree a little better. If
you are interested in creating a new Spree extension that will create or
affect adjustments in some way, this understanding is crucial.

#### Computation

Computation for adjustments comes down to a very straightforward
*compute* method that is provided by an instance of *Spree::Calculator*.
In some cases the calculation can be quite straight forward. Take a look
at *Spree::Calculator::FlatRate* to see such a simple case.

<ruby>\
def compute(object = nil)\
 self.preferred\_amount\
end\
</ruby>

This calculator basically just returns the same amount every time it’s
called (note the amount is configurable of course.) Other calculators
(such as *Spree::Calculator::DefaultTax*) are a bit more interesting.

<ruby>\
def compute(computable)\
 case computable\
 when Spree::Order\
 compute\_order(computable)\
 when Spree::LineItem\
 compute\_line\_item(computable)\
 end\
end

private

def compute\_order(order)\
 matched\_line\_items = order.line\_items.select do |line\_item|\
 line\_item.product.tax\_category  rate.tax\_category
    end

    line\_items\_total = matched\_line\_items.sum(&:total)
    round\_to\_two\_places(line\_items\_total \* rate.amount)
  end

  def compute\_line\_item(line\_item)
    if line\_item.product.tax\_category  rate.tax\_category\
 deduced\_total\_by\_rate(line\_item.total, rate)\
 else\
 0\
 end\
 end\
</ruby>

You see the *compute* is more complex with polymorphic handler.

INFO: Calculators should never return a value of *nil* in their
*compute* method. Return *0* in these situations instead.

#### Locked Adjustments

Spree also has the notion of so-called “locked” adjustments. The concept
behind this is there are certain types of adjustments which should never
be changed once calculated. There are also situations where it is
appropriate for an adjustment that was once subject to recalculation to
be recalculated.

WARNING: Locked adjustments are currently considered experimental. We’re
currently exploring the ability to configure Spree to honor certain
locking strategies. For instance, locking a promotion calculation in
place so that it’s not lost if the order is updated after the promotion
expires. There are also some interesting options to allow admins with
the appropriate permissions to have the ability to lock/unlock
adjustments when appropriate.

### Configuration

Since calculators are an instances of *ActiveRecord::Base* they can be
configured with preferences. Each instance of *Spree::ShippingMethod* is
now stored in the database along with the configured values for its
preferences. This allows the same calculator (ex.
*Spree::Calculator::FlatRate*) to be used with multiple
*Spree::ShippingMethods*, and yet each can be configured with different
values (ex. different amounts per calculator.)

Calculators are configured using Spree’s flexible [preference
system](preferences.html). Default values for the preferences are
configured through the class definition. For example, the flat rate
calculator class definition specifies an amount with a default value of
0.

<ruby>\
module Spree\
 class Calculator::FlatRate \< Calculator\
 preference :amount, :decimal, :default =\> 0\
 …\
 end\
end\
</ruby>

Spree now contains a standard mechanism by which calculator preferences
can be edited. The screenshot below shows how the amounts for the flat
rate calculator are editable directly in the admin interface.

![Changing Shipping Config](images/calculators/shipping_config.png "Changing Shipping Config")
