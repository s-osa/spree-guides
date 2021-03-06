---
title: "Orders"
section: core
---

## Overview

The Order model is one of the central models in Spree, providing a central place to collect information about the order, including line items, adjustments, payments, addresses, return authorizations, <%= link_to "inventory units", "#" %>, and shipments.

Every order that is created within Spree is given its own unique identifier, beginning with the letter R and ending in a 9-digit number. This unique number is shown to the users, and can be used to find the Order by calling `Spree::Order.find_by_number(number)`.

Orders have the following attributes:

* `number`: The unique identifier for this order.
* `item_total`: The sum of all the line items for this order.
* `adjustment_total`: The sum of all adjustments on this order.
* `total`: The sum of `item_total` minus the `adjustment_total`.
* `state`: The current state of the order.
* `email`: The email address for the user who placed this order. Stored in case this order is for a guest user.
* `user_id`: The ID for the corresponding user record for this order. Stored only if the order is placed by a signed-in user.
* `completed_at`: The timestamp of when the order was completed.
* `bill_address_id`: The ID for the related `Address` object with billing address information.
* `ship_address_id`: The ID for the related `Address` object with shipping address information.
* `shipment_state`: The current shipment state of the order. For possible states, please see the <%= link_to "Shipping guide", "#" %>
* `payment_state`: The current payment state of the order. For possible states, please see the <%= link_to "Payments guide", "#" %>
* `special_instructions`: Any special instructions for the store to do with this order. Will only appear if `Spree::Config[:shipping_instructions]` is set to `true`.
* `currency`: The currency for this order. Determined by the `Spree::Config[:currency]` value that was set at the time of order.
* `last_ip_address`: The last IP address used to update this order in the frontend.

## The Order State Machine

Orders flow through a state machine, beginning at a "cart" state and ending up at a "complete" state. The intermediary states can be configured using the <%= link_to "Checkout Flow API", 'checkout_flow' %>.

The default states are as follows:

* Cart
* Address
* Delivery
* Payment
* Confirm
* Complete

The "Payment" state will only be triggered if `payment_required?` returns `true`.

The "Confirm" state will only be triggered if `confirmation_required?` returns `true`.

The "Complete" state can only be reached in one of two ways:

1. No payment is required on the order.
2. Payment is required on the order, and at least the order total has been received as payment.

Assuming that an order meets the criteria for the next state, you will be able to transition it to the next state by calling `next` on that object. If this returns `false`, then the order does *not* meet the criteria. To work out why itcannot transition, check the result of an `errors` method call.

## Line Items

Line items are used to keep track of items within the context of an order. These records provide a link between orders, and <%= link_to "Variants", 'products', 'variants' %>.

When a variant is added to an order, the price of that item is tracked along with the line item to preserve that data. If the variant's price were to change, then the line item would still have a record of the price at the time of ordering.

* Inventory tracking notes
***TODO:*** Update this section after Chris+Brian have done their thing.

## Addresses

An order can link to two address objects. The shipping address indicates where the order's product(s) should be shipped to. This address is used to determine which shipping methods are available for an order.

The billing address indicates where the user who's paying for the order is located. This can alter the tax rate for the order, which in turn can change how much the final order total can be.

For more information about addresses, please read the <%= link_to "Addresses", :addresses %> guide.

## Adjustments

Adjustments are used to affect an order's final cost, either by decreasing it (<%= link_to "Promotions", :promotions %>) or by increasing it (<%= link_to "Shipping", :shipping %>, <%= link_to "Taxes", :taxation %>).

For more information about adjustments, please see the <%= link_to "Adjustments", :adjustments %> guide.

## Payments

Payment records are used to track payment information about an order. For more information, please read the <%= link_to "Payments", :payments %> guide.

## Return Authorizations

TODO: document return authorizations.

## OrderPopulator

TODO: Add documentation about the OrderPopulator class here

## Updating an order

If you change any aspect of an `Order` object within code and you wish to update the order's totals -- including associated adjustments and shipments -- call the `update!` method on that object, which calls out to the `OrderUpdater` class.

For example, if you create or modify an existing payment for the order which would change the order's `payment_state` to a different value, calling `update!` will cause the `payment_state` to be recalculated for that order.

Another example is if a `LineItem` within the order had its price changed. Calling `update!` will cause the totals for the order to be updated, the adjustments for the order to be recalculated, and then a final total to be established.