==========================
Inventory: Under the wood
==========================

The inventory valuation example
********************************

Let's take a look at the inventory valuation report.
The report can be found in (Inventory -> Reporting -> Inventory Valuation)
A popup appears asking if you want to compute the current inventory or the inventory at a specific date.

inventory valuation of current inventory will redirect you to
http://amazing_stock.odoo.com/web?#action=912&active_id=1&model=product.product&view_type=list&menu_id=

inventory valuation of current inventory will redirect you to
http://amazing_stock.odoo.com/web?#active_id=2&model=product.product&view_type=list&menu_id=

Both are in fact different views on the model product.product.

Let's set the context
***********************
The main differences are in the context:

The context is a strange odoo concept: we use for communication between the client and the server.

You can 'easily' see the context sent to the server.
When you look at the https(s) requests and inspect the headers:
(in chrome, press F12 then open the tab network and look for the search_read)

see the images for more details in the referenceA_ :

context of inventory valuation of current inventory:


.. code-block:: json

    {
    "jsonrpc":"2.0",
    "method":"call",
    "params":{
        "model":"product.product",
        "domain":[
            ["type","=","product"],["qty_available","!=",0]
        ],
        "fields":[
            "display_name",
            "qty_at_date",
            "uom_id",
            "currency_id",
            "cost_currency_id",
            "stock_value",
            "cost_method"
        ],
        "limit":80,
        "sort":"",
        "context":{
            "lang":"en_US",
            "tz":"Europe/Brussels",
            "uid":2,
            "valuation":true,
            "active_model":"stock.quantity.history",
            "active_id":7,
            "active_ids":[7],
            "company_owned":true,
            "create":false,
            "edit":false,
            "search_disable_custom_filters":true
        }
    },
    "id":431504480
    }


context of inventory valuation of current inventory:

.. code-block:: javascript

    {
    "jsonrpc":"2.0",
    "method":"call",
    "params":{
        "model":"product.product",
        "domain":[
            ["type","=","product"],["qty_available","!=",0]
        ],
        "fields":[
            "display_name",
            "qty_at_date",
            "uom_id",
            "currency_id",
            "cost_currency_id",
            "stock_value",
            "cost_method"
        ],
        "limit":80,
        "sort":"",
        "context":{
            "lang":"en_US",
            "tz":"Europe/Brussels",
            "uid":2,
            "valuation":true,
            "active_model":"stock.quantity.history",
            "active_id":8,
            "active_ids":[8],
            "default_compute_at_date":0,
            "to_date":"2019-08-09 10:59:29",
            "company_owned":true,
            "create":false,
            "edit":false,
            "search_disable_custom_filters":true
        }
    },
    "id":925403830
    }

Well but how are computed the reports.
****************************************

The reports shows the fields in the search (same reports)

.. code-block:: javascript

    "fields":[
        "display_name",
        "qty_at_date",
        "uom_id",
        "currency_id",
        "cost_currency_id",
        "stock_value",
        "cost_method"]


Everything is about compute
*****************************

In fact we are just displaying the fields of the product.product model.
In the fields of this model, we can see the following;

- ``qty_at_date = fields.Float('Quantity', compute='_compute_stock_value')``
- ``stock_value = fields.Float('Value', compute='_compute_stock_value')``

At this point, you did understand that most of the logic is in ``_compute_stock_value``

.. code-block:: python

    @api.multi
    @api.depends('stock_move_ids.product_qty', 'stock_move_ids.state', 'stock_move_ids.remaining_value', 'product_tmpl_id.cost_method', 'product_tmpl_id.standard_price', 'product_tmpl_id.property_valuation', 'product_tmpl_id.categ_id.property_valuation')
    def _compute_stock_value(self):
        StockMove = self.env['stock.move']
        to_date = self.env.context.get('to_date')
        ...

Indeed at last line, we will use the value provided by the context to determine how to compute the stock value. (see referenceC_)

References
************

.. _referenceA:

A. context details
^^^^^^^^^^^^^^^^^^^

- Inventory valuation (current inventory)

    .. image:: media/context_current_inventory.png

- Inventory valuation (inventory at date)

    .. image:: media/context_inventory_at_date.png

B. Github links
^^^^^^^^^^^^^^^^^
.. _referenceB:

- qty_at_date

    https://github.com/odoo/odoo/blob/62f98e34c8aa0fbc1755fec9919396c853c171dc/addons/stock_account/models/product.py#L107

- stock_value

    https://github.com/odoo/odoo/blob/62f98e34c8aa0fbc1755fec9919396c853c171dc/addons/stock_account/models/product.py#L105-L106

.. _referenceC:

- context use

    https://github.com/odoo/odoo/blob/62f98e34c8aa0fbc1755fec9919396c853c171dc/addons/stock_account/models/product.py#L105-L106




