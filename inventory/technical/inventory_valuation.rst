===============================
Inventory Valuation In Detail
===============================

The following chapter is more technical and explains how specific elements are computed.
Basically the elements below are computed for a product.product not for a product.template.
When calculated for different elements, it may differs.

On Hand quantities
*******************

- For product.template

sum of the quantities of all the product.product (variants)

- For product.product

TODO:


Inventory Valuation (Current Inventory)
*****************************************
This is specified in odoo/addons/stock_account/models/product.py mostly in the method _compute_stock_value() (see refA_)

Fifo Manual
-------------------

TODO:

Fifo Automated
-------------------

The quantity is the sum of the quantity for stock.move of this product with a specific domain.
The value is the sum of the remaining_value for stock.move of this product with a specific domain.

The domain is defined in odoo/addons/stock_account/models/stock.py in the method _get_all_base_domain (see (see refB_))

It is hard to translate this in english but it can looks like this:

- state is done
- Then, either 'in' or 'out' moves.
    * 'in' moves:
        + coming from a location without company, or an inventory location within the same company
        + going to a location within the same company
    * 'out' moves:
        + coming from to a location within the same company
        + going to a location without company, or an inventory location within the same company

    .. code-block:: javascript

          domain = [
            ('state', '=', 'done'),
            '|',
                '&',
                    '|',
                        ('location_id.company_id', '=', False),
                        '&',
                            ('location_id.usage', 'in', ['inventory', 'production']),
                            ('location_id.company_id', '=', company_id or self.env.user.company_id.id),
                    ('location_dest_id.company_id', '=', company_id or self.env.user.company_id.id),
                '&',
                    ('location_id.company_id', '=', company_id or self.env.user.company_id.id),
                    '|',
                        ('location_dest_id.company_id', '=', False),
                        '&',
                            ('location_dest_id.usage', '=', 'inventory'),
                            ('location_dest_id.company_id', '=', company_id or self.env.user.company_id.id),


Inventory Valuation (At a Specific Date)
*****************************************
This is specified in odoo/addons/stock_account/models/product.py mostly in the method _compute_stock_value()

Fifo Manual:
-----------------

TODO:

Fifo Automated:
-----------------

The quantity is computed from the Inventory Valuation (Current Inventory). If the quantity is negative or null the line is not displayed.
The value is computed from the account.move.lines and is simply the sum balances of all the account.move.lines where the account is the valuation account of the product.



References
************

.. _refA:

https://github.com/odoo/odoo/blob/48e93a10a1d1dbb0c6301eb33cf433db6d59f02b/addons/stock/models/product.py#L92-L93

.. _refB:

https://github.com/odoo/odoo/blob/a5a4b154ee4e4d83f99b35b8899ad406dbee6b12/addons/stock_account/models/stock.py#L178-L206
