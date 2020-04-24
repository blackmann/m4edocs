Invoicing
************

Invoices are requests to make a payment. They generally indicate the sending and receiving wallet and amount involved. However, the client may need to store some extra details on payments. The invoice makes that possible by using a ``meta`` field.

Invoice are created in the format:

.. code-block:: javascript

    {
        transactionId: "", // uuid
        from: {
            type: "m4e",
            id: "ae890-19800-b87a0"
        },
        fromRef: "", // uuid
        to: {
            type: "mtn",
            id: "02447812093"
        },
        amount: 20.00, 
        currency: "GHS",
        meta: {
            callback: "", // empty or url to updated with transaction
            purpose: "Pay for electricity bill",
            data: {}, // custom extra details
        } 
    }

Meta
--------

Meta provides the ability to include data are auxiliary to the transaction. For instance, a mobile app used to collect donations may indicate the donation type in the ``meta.purpose``. However, if points are given for donations, then this detail can be placed in a namespaced field of ``data``. Example:

.. code-block:: javascript

    {
    ...
        purpose: "Donated to Akwatia Orphanage",
        data: {
            "hm-donations-extra": {
                points: 15,
            }
        }
    }

Note: One reason a client may keep such details in a transaction is because it does not necessarily have a database system (server) to persist this extra data.
