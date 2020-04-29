Payment Switch (Broker)
*************************

Payments can be made **from** different kind of wallets into **another** kind of wallet which, in some cases, may not belong to the same payment provider. For this reason, the Broker intervenes by abstracting the movement of money between wallets of possibly different payment providers.

The switch has references to internal (merchant) accounts/wallets for each payment provider supported. The reason for this will be seen the following sections.

Movement of money between wallets of the same payment provider
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is a simpler form of a transaction. When making a payment of **GHS 60.00** from ``Wallet A: Provider X`` to ``Wallet B: Provider X``, the switch undergoes the following steps:

1. Initiates a payment of GHS 60.00 from ``Wallet A`` into ``Merchant Wallet: Provider X``. 

2. Upon success, the switch then initiates a payment of GHS 60 from ``Merchant Wallet: Provider X`` into ``Wallet B: Provider X``.

.. .. code-block:: java

..     final ProviderX providerX;

..     final MerchantWallet<ProviderX> merchantWallet;

..     ...

..     // Send money from [a] to [b]
..     void send(Wallet a, Wallet b, double amount) {
..         // step 1
..         final paymentForm = {
..             "from": a.id,
..             "to": merchantWallet.id,
..             "amount": amount
..         }

..         providerX.sendMoney(paymentForm);

..         // step 2 (demonstration purposes)
..         // technically, the payment call does not respond 
..         // immediately. Therefore, the following instruction
..         // will be completed on a separate thread

..         // upon success

..         send(merchantWallet, b);
        
..     }


Movement of money between wallets of different payment providers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When making payment from ``Wallet X: Provider A`` to ``Wallet Y: Provider B`` the following steps are taken:

1. Initiate payment from ``Wallet X`` into ``Merchant Wallet: Provider A``

2. Upon success of stage 1, transfer money from ``Merchant Wallet: Provider B`` into ``Wallet Y``


General rule
^^^^^^^^^^^^^^^

From both scenarios above, firstly money is moved from the senders wallet into m4e merchant wallet of the same provider. Upon success, money is moved from an m4e merchant wallet of the same provider as the receiver into the receiver wallet.

.. code-block:: java

    class Broker {

        final Map<Provider, Wallet> walletGraph;

        ...

        // Load money from [transaction.from] into merchant account 
        void cashIn(Transaction transaction) {
            // step 1: receive money into merchant account of 
            // the same provider as the sender

            Wallet receiptWallet = walletGraph.get(transaction.from.provider);

            Map<String, Object> paymentForm = {
                "from": transaction.from.providerId,
                "to": receiptWallet.providerId,
                "amount": transaction.amount,
                "transactionId": transaction.id,
            } 

            transaction.from.provider.sendMoney(paymentForm);

            // step 2 (demonstration purposes)
            // technically, the payment call does not respond 
            // immediately. Therefore, the following instruction
            // will be completed on a separate thread

            // upon success

            transaction.proceed(paymentStatus)
        }

        // Send money from merchant account into [transaction.to]
        void cashOut(Transaction transaction) {

            Wallet sendWallet = walletGraph.get(transaction.to.provider):

            Map<String, Object> paymentForm = {
                "from": sendWallet.providerId,
                "to": transaction.to.providerId,
                "amount": transaction.amount,
                "transactionId": transaction.id,
            }

            sendWallet.provider.sendMoney(paymentForm);
        }
    }



.. code-block:: java

    class Transaction {

        Wallet from;
        Wallet to;
        double amount;


        void proceed(bool paymentStatus) {
            if (paymentStatus.isSuccessful()) {
                // moves on to next stage of transaction
                
                Switch.getInstance().cashOut(this);
            }
        }
    }


Problems that can arise during switching
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Some issues that may arise during a transaction include: 

1. Invalid transaction receipient wallet ``(severe)``: That is, the first process of the transaction may have completed successfully and the Merchant wallet credited. However, when proceeding to the second stage to credit the receiving wallet, the provider may respond with the wallet/account being invalid. In such cases, [line of action here]
