Hooks
-----

The following is a list of all hooks included in Spree’s default
templates. These allow your own content to be inserted into the
templates without having to override them. For instructions on using
hooks, refer to the [customization guide](customization.html#hooks)

endprologue.

### Public views

#### Layout (layouts/spree\_application.html.erb)

-   inside\_head (allows you to modify content of head tag)
-   sidebar (for any pages that have a sidebar)

#### Homepage (products/index.html.erb)

-   homepage\_sidebar\_navigation
-   homepage\_products

#### Taxon (taxons/show.html.erb)

-   taxon\_sidebar\_navigation
-   taxon\_products
-   taxon\_children

#### View Product (products/show.html.erb products/\_taxons.html.erb products/\_cart\_form.html.erb)

-   product\_description
-   product\_properties
-   product\_taxons (‘Look for similar items’)
-   product\_price
-   inside\_product\_cart\_form

#### Cart (orders/edit.html.erb)

-   inside\_cart\_form
-   cart\_items

#### Login (user\_sessions/new.html.erb)

-   login

#### Signup (users/new.html.erb, users/\_form.html.erb)

-   signup
-   signup\_inside\_form
-   signup\_below\_password\_fields (within form, below password
    confirmation field)

### Admin Views

#### Layout (layouts/admin.html.erb)

-   admin\_inside\_head

#### Navigation

The following hooks allow list items to be added to various admin menus

-   admin\_tabs
-   admin\_product\_sub\_tabs
-   admin\_order\_tabs (sidebar menu for individual order)
-   admin\_product\_tabs (sidebar menu for individual product)

