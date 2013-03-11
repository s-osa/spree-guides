Scopes and Product Groups
-------------------------

This section explains Spree’s facilities for selecting lists of products
by various criteria.\
After reading it you should know:\
\* the scopes provided for filtering lists of products\
\* about the concept of product groups\
\* how to use product groups.\
\* how to configure and use the search API.

endprologue.

### Overview

In various applications, we need to create lists of products according
to various criteria, e.g. \
all products in taxon X, or all products costing less than \$20, or
composite criteria like all \
products costing more than \$100 that have brand Y.

This chapter first covers the available **named scopes** in the Spree
core which provide filtering\
by various aspects of the basic data, some of which can be chained or
composed together \
for more complex filters.

The second part explains the facilities for **defining and naming**
groups of products for\
various purposes in an application. For example, you can define a group
called *Ruby products*\
which contains everything with *Ruby* in the product name, or another
group called *Ruby clothing* \
which is the sub-list of the above limited to items in the Clothing
taxon. Product group definitions\
can also take price ranges, product properties, and variant options into
account. It is possible\
to add your own filters by defining new named scopes too.

This chapter is mainly aimed at developers, who will usually create and
hard-code definitions \
and access to certain groups, but administrators will sometimes want to
edit the criteria for \
these existing groups. The [admin interface editing
facilities](#administrationinterface) is\
explained below.

### Product Scopes

#### Primitive selection scopes

-   *active+ - this is the key scope for selecting listable products,
    and is the combination of \
    *available(optional\_date)+ (the product’s availability date is not
    later than the given date or the default current date) and
    *not\_deleted* (not marked as deleted in the database)

-   *keywords* - defaults to SQL text search in the product names and
    descriptions, though you can modify or extend this via the [Searcher
    API](searching.html#searcher-api)

-   *in\_name*,*in\_name\_or\_keywords*,*in\_name\_or\_description+ -
    testing for any of a list of strings passed as a space or comma
    separated string, appearing in the product database
    \
    \**price\_between(low,high)+ - selects for a master price being in
    the limits (using SQL’s *BETWEEN* operator)

-   *in\_taxon+ and*in\_taxon(thing1,thing2,…)+ - selects products in
    the given taxon(s) *and their descendants*. The arguments can be
    taxon ids, taxon objects, names of taxons, or (suffixes of)
    permalinks.

-   *having* a product property (*with\_property*) or option type *)
    associated with the product - but not checking for actual values,
    where*thing* can be a string, id, or property/option\_type object
    \
    \**with\_property\_value(thing,value)+ - testing for a product
    having the given value. *thing* is as above, and *value* is a string
    (or convertible to a string). Note that this can’t test for a
    product lacking a property\
     (ie no database row), but can test for a property that is
    explicitly set to NULL.

-   *with\_option\_value+ - selects for a product having a variant with
    the given value for the option
    \
    \**with(value)+ - selects products having either a property value or
    having a variant with an option value which is equal to the given
    value

You can also use the scopes available via SearchLogic. These follow the
pattern of field plus a test, e.g. *master\_price\_lte+
or*taxons\_name\_eq(“foo”)+ (note the easy access across associations).

#### Primitive ordering scopes

SearchLogic is used to produce most of the commonly used ordering
scopes, e.g. *ascend\_by\_updated\_at*.\
There is a scope *by\_popularity* which orders by the number of orders a
product appears in.

#### Chaining scopes together

The above scopes can be used like any other scopes, and thus be chained
arbitrarily. However, due\
to the type of filters used and (in some cases) the implementation,
certain combinations may “not\
work” or return an empty list:

-   most of the primitive scopes *reduce* (or sometimes, preserve) the
    number of products available, and sometimes the conditions have no
    solution amongst available products
-   some combinations of scopes are of course contradictory, e.g.
    selecting scope price less than \$10 and price greater than \$15
-   for efficiency reasons (balanced against typical use), it is not
    possible to filter by more than one property test or by more than
    one option value at a time (the subsequent tests would clash with
    the first one)

Of course, developers are free to add more complex scopes should they
wish to undertake more complex chaining.

So called “order scopes” should not change the number of items
displayed, just the order in which \
items appear in the list.

### Product Groups

NOTE: Product Groups is longer a part of Spree core, it has been
extracted to\
a standalone extension, which can be found at
[spree\_product\_groups](https://github.com/spree/spree_product_groups)

Product groups are named lists of products which are selected via a
defined combination of criteria.\
They can be created, edited, destroyed by administrators; and viewed by
the user via appropriate\
permalinks or developers can write code to display them as they wish.

In general, product groups are a composition of scopes with an optional
ordering constraint (which is \
also a scope).

Extensive low level documentation can be found in source code of the
*ProductGroup*\
and *ProductScope* models.

INFO: product groups do not automatically include the *active* scope, so
may return out of date products.

#### Administration interface

##### Listing of product groups

The index page allows quick inspection of available groups - primarily
to check for expected \
cardinality, look up the permalink, or get quick access to the preview.
You can also edit\
or delete groups from this page.

##### Creating a new scope

When creating a new product group, you need:

-   name of the group - it should be descriptive, as it’ll be reflected
    in both admin UI and product group url.
-   order by which results will be displayed
-   parameters for zero or more of the available scopes

A scope is activated by checking its tick box, and (where appropriate)
supplying a parameter of \
appropriate type, eg. a string or numeric value. The form provides hints
for most of the scopes.

After you select conditions, you can preview products that will be
selected by your conditions and inspect\
if the result is what you would expect.

After you create the product group, you’ll be presented with information
about the new group,\
including bookmarkable (permalink) URL, current product count, and other
useful information.

#### Access via URLs

You can access named product groups via URLS of the following format.

-   */t/\*taxons/pg/named\_product\_group* - list a product group after
    restricting to a particular taxon
-   ***/pg/named\_product\_group+ - list a product group
    \
    There’s also *nameless* versions of the above, where the URL encodes
    the required scopes and their\
    arguments. After *s/*, there should be a sequence of scope-argument
    pairs followed by an optional\
    ordering scope. Scope arguments should be comma-separated. \
    Example:
    */s/master\_price\_gte/10/taxons\_name\_like\_any/pach,uby/ascend\_by\_master\_price*
    \
    ***/t/\*taxons/s/name\_of\_scope/comma\_separated\_arguments/name\_of\_scope\_that\_doesn\_take\_any//order+
-   ***/s/name\_of\_scope/comma\_separated\_arguments/name\_of\_scope\_that\_doesn\_take\_any//order+


    \
    h4. Implementation overview
    \
    Product groups are effectively a representation of a set of scopes.
    The API is similar to standard product scope values.
    \
    For efficiency, all named product groups are memoized - retrieval
    just\
    consults a single database table. The memoization is updated when a
    product group or product is saved.

    \
    h3. Customization guide
    \
    h4. Translating Product Groups and scopes
    \
    The Product Group functionality relies heavily on Rails I18n
    capabilities to ensure\
    appropriate end user text for name, description, and format string
    for product scopes and groups.\
    So for example, text for the *price\_between* scope that takes two
    arguments looks like this \
    in *en-US*:\
    \<pre\>\
     price\_between:\
     args:\
     high: High\
     low: Low\
     description: “”\
     name: “Price between”\
     sentence: price between <em>%.2f</em> and <em>%.2f</em>\
    \</pre\>\
    ** *Name* if the name of the scope displayed in admin interface.\
    \* *Description* is used only in admin interface UI to provide
    additional information about usage of the scope.\
    \* *Sentence* is used for describing the scope to the end user, \
     and as the name suggests it should be formed as a sentence\
     that can be connected using “and”, sentence is evaluated using
    *String\#%* method and can be formatted using\
     printf escape sequences.
    \
    WARNING: It’s very important that the number of expected arguments
    for the ‘sentence’ string does not exceed the number available for
    the relevant scope, else*printf+ will raise an exception.

The same rules apply for ordering scopes, except that they don’t take
any arguments so do not need a \
‘sentence’ field. \
The documentation of Scopes module provides some additional information
on translating customized scopes.

#### Modifying available scopes

All scopes available in the admin UI are defined in
*lib/scopes/product.rb* file, in the *SCOPES* constant.\
Format of the constant should be self explanatory, but it’s recomended
to read *Scopes* module documentation\
too.\
When using Ruby 1.8, the order of scopes within groups, and order of
groups themselves, is arbitrary,\
Ruby 1.9 preserves order of hash, and in effect reflects order of
declarated scopes in UI.

You can include any valid Product scope (including dynamically created
SearchLogic scopes) in the\
Product Groups mechanism by declaring it in the *SCOPES* constant. \
If you plan to provide your own custom scope make sure to check that:

-   the new scope doesn’t clash with any of the existing scopes
    (especially when you’re using joins)
-   it doesn’t change order of the result (unless it is a new order
    scope)
-   it does specify all neccessary joins and includes (i.e. it is
    self-contained and commutative - you shouldn’t rely on joins or
    includes from other scopes)
-   it doesn’t do unnecessary joins or includes (each extra join may be
    a performance bottleneck)
-   be very careful with scope of names, especially when you’re joining
    with *:variant* or *:master*,\
     e.g. make sure to use “AS” sql statement on joins and in conditions
    to avoid name capture or ambiguity.

Remember also to provide translations for your new scopes; there’s a \
*Scopes.generate\_translation* method that can be helpful here.
