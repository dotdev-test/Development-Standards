# Test Cases

Test cases design should be completely matching with [sequence diagram](technical-specification.md). To perform efficient and repeatable testing, setting and testing products need to be setup in [client requirement](client-requirements.md).

## Customers

### Create

* Create an account from admin panel
* Register an account with address from the site
* Register an account without address from the site
* Create an account from the other platform

### Update

* Update an account from admin panel
* Edit an account from the site
* Edit an account from the other platform
* Edit an account's address from the site

### Special Case

* Create an account from Shopify which has same email (identifier) existing in the other platform
* Create/Edit an account with long name
* Create/Edit an account with long address
* Create/Edit an account with invalid email
* Create/Edit an account with only email information (guest checkout or subscription)
* Create/Edit an account with only phone information

## Products/Variants

### Create Products

* Create a product with no variant from the other platform
* Create a product with variants of 1 option from the other platform
* Create a  product with variants of more than 1 option from the other platform

### Update Products

* Update a product description from the other platform
* Hide/Unpublished a product from the other platform

### Delete Products

* Delete a product from the other platform

### Create Variants

* Create a variant under an existing product from the other platform

### Update Variants

* Edit price of a variant under an existing product from the other platform
* Edit stock level of a variant under an existing product from the other platform

### Delete Variants

* Remove a variant under an existing product from the other platform

### Special Case

* Remove a variant under an existing product from the other platform, and add it back

## Orders

### Create

* Create an order from admin panel, mark it as paid
* Create an order from admin panel, give it free shipping
* Place a real order from the site, pay with cash
* Place a real order from the site, pay with other payment method instead of cash. (e.g. credit card)
* Place a real order from the site, using guest checkout
* Place a real order that qualified to free shipping from the site

### Create Click-and-Collect Order

* Create a click-and-collect order from admin panel, mark it as paid
* Place a click-and-collect real order from the site, pay with cash
* Place a click-and-collect real order from the site, pay with other payment method instead of cash. (e.g. credit card)
* Place a click-and-collect real order from the site, using guest checkout

### Fully-fulfill

* Fulfill all line items of an existing order from Shopify
* Fulfill all line items of an existing order from the other platform

### Partial-fulfill

* Fulfill 1 line item of an existing order from Shopify
* Fulfill 1 line item of an existing order from the other platform

### Fully-refund

* Refund all line items of an existing order from Shopify
* Refund all line items of an existing order from the other platform

### Partial-refund

* Refund 1 line item of an existing order from Shopify
* Refund 1 line item of an existing order from the other platform

### Special Case

* Refund an existing order without refunding shipping fee from Shopify
* Refund an existing order without refunding shipping fee from the other platform
