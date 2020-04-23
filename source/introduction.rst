Introduction
*****************

Payments made with the m4e system is completed using the m4e switch (call it the TSwitch). The TSwitch is responsible for taking a uniform payment form and completing the transaction with a specified payment provider (see :doc:`payment_providers`).

Note on keywords **(payment and transactions)**: Payments is the transfer of money from one wallet to another. Transactions hold the progress (success or failure) of a payment process. Thus a transaction is a record of payments. A transaction may involve so many payments/movement of money.

The figure below demonstrates a basic overview of payment processing on m4e

.. figure:: _static/m4epayment.jpg

    Figure1: Payment processing from an MTN Momo Wallet to Bank Account wallet


Agents
---------

From the figure above we can derive the following agents involved in a payment transaction.

* Client
* Payment Switch (TSwitch)
* Payment Provider
* Signalee


Client
^^^^^^^^^^
The client is the agent that initiates the payment process. The m4e system *(or specifically the TSwitch)* is agnostic of the client. Therefore the client can be an external server, the m4e payment SDK, a POS device, etc. To begin payment process *(or create a transaction)* the client needs to provide the following form data:


.. code-block:: javascript

    {
        transactionId: "", // uuid
        from: "", // paying wallet id
        fromRef: "", // uuid
        to: "", // receiving wallet id
        amount: 20.00, 
        currency: "GHS",
        meta: {
            callback: "", // empty or url to updated with transaction
            purpose: "Pay for electricity bill",
            data: {}, // custom extra details
        } 
    }

This form can also be referred as an **Invoice** to be paid by the **from** wallet. The invoice is forwarded to the TSwitch to proceed with the payment process.


TSwitch
^^^^^^^^^

Payments can be made **from** different kind of wallets into **another** kind of wallet which, in some cases, may not belong to the same payment provider. For this reason, the TSwitch intervenes by abstracting the movement of money between wallets of possibly different payment providers.

The mechanism of how the TSwitch works is described in :doc:`tswitch`.

However, the extra process involved in completing the transaction requires the TSwitch to respond to the client instantly with an **in-progress** status.

Upon completion of payment process, the **Signalee** then notifies the client on the final status of the transaction (success or failure).


Payment Provider
^^^^^^^^^^^^^^^^^

Payment providers simply handle incoming and outgoing of payments. Provider abstractions implement the interface below.

.. code-block:: java

    // PaymentForm is a generic form class that specifies fields required for
    // a particular payment channel/method
    interface PaymentInterface<PaymentForm> {

        void sendMoney(PaymentForm form);

        // this method is called when the external payment channel makes 
        // a callback with the status of the transaction
        void onPaymentStatusUpdate(String transactionId, Object status);

        void checkPaymentStatus(String transactionId);

    }




Signalee (Post-Processor)
^^^^^^^^^^^^^^^^^^^^^^^^^^

The signalee uses the idea of database signals (in `Django <https://docs.djangoproject.com/en/3.0/topics/signals/>`_) or entity listeners (in `Hibernate <https://docs.jboss.org/hibernate/orm/4.0/hem/en-US/html/listeners.html>`_). That is the Signalee (or say, `S`) listens to persistence events on a transaction and performs additional operations around the transaction.

A prominent use-case with the `S` is to notify the client either by push notification, email, sms or callback URL.

Note: The Signalee should not be confused with an agent that notifies (or signals) clients. The signalee is rather signaled by record persistence which then performs any other kind of additional effects besides just notifying clients.
 