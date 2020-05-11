Third Party Applications
^^^^^^^^^^^^^^^^^^^^^^^^^^

Third party apps or clients need scoped access/permissions on a user. This scope is defined when the client/app is registered on m4e. For example, a client ``UN Bank`` registered on m4e will need a number of permission on m4e users they need to manage. Therefore, credentials *(on the server)* for ``UN Bank`` will include

.. code-block:: javascript

    {
        client_name: 'UN Bank',
        client_id: '8b2a2b58-f3db-4d81-a95d-ec46e678f071',
        client_secret: '*****************',
        scope: [
            'CREATE_WALLET',
            'READ_WALLET', // non-private wallets
            'READ_TRANSACTIONS',
        ],
        description: ...,
    }


Requesting permissions on a user requires the user to login into m4e while specifying the client (to grant permissions to). Currently using phone number for login, the endpoint to request for permissions will be in form:

.. code-block:: javascript

    endpoint = '/login?client_id=8b2a2b58-f3db-4d81-a95d-ec46e678f071&phone=0247812093&verification=866129'


*Normally, the user will have to explicitly confirm with a dialog to grant those permissions, after login. This can be easily refactored later*

m4e then returns to this request with a ``token`` which will be used by the client (UN Bank, in this example) to verify the user.

.. code-block:: javascript

    {
        token: 'eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzI1NiJ9.eyJzdWIiOiIxNmM5...',
        grant: [
            'CREATE_WALLET',
            'READ_WALLET',
            'READ_TRANSACTIONS'
        ] // this may not be relevant at the moment, since users can't decline/accept selected access
    }


Verifying access
------------------

The client can then retrieve an ``access_token`` (and possibly a ``refresh_token``) and other credentials from m4e using a combination of the ``token``, ``client_id`` and ``client_secret``.

.. code-block:: javascript

    endpoint = '/verify?client_id=&client_secret=&token=eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzI1NiJ9.eyJzdWIiOiIxNmM5...'


m4e responds with

.. code-block:: javascript

    {
        user: {
            id: '59e670a5-bd97-41be-a87b-187014362dad',
            phone: '0247812093',
            ...
        },
        access_token: 'JleHAiOjE1ODkxNTUyODgsImlhdCI6MTU4OTA2ODg4OH0.LILWyNUoFvh0FHoukyeudY3lUvLT...',
        refresh_token: '...',
        expires: ...,
    }

The ``access_token`` will then be used to make requests (specified in the scope) on behalf of this user.

*Decide: use basic auth (client id, client secret) and then specify access token in url*
