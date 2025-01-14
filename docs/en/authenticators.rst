Authenticators
##############

Authenticators handle convert request data into an authentication
operations. They leverage :doc:`/identifiers` to find a
known :doc:`/identity-object`.

Session
=======

This authenticator will check the session if it contains user data or
credentials. When using any stateful authenticators like ``Form`` listed
below, be sure to load ``Session`` authenticator first so that once
logged in user data is fetched from session itself on subsequent
requests.

Configuration options:

-  **sessionKey**: The session key for the user data, default is
   ``Auth``
-  **identify**: Set this key with a value of bool ``true`` to enable checking
  the session credentials against the identifiers. When ``true``, the configured
  `Identifiers <./Identifers.md>`__ are used to identify the user using data
  stored in the session on each request. Default value is ``false``.
-  **fields**: Allows you to map the ``username`` field to the unique
   identifier in your user storage. Defaults to ``username``. This option is
   used when the ``identify`` option is set to true.

Form
====

Looks up the data in the request body, usually when a form submit
happens via POST / PUT.

Configuration options:

-  **loginUrl**: The login URL, string or array of URLs. Default is
   ``null`` and all pages will be checked.
-  **fields**: Array that maps ``username`` and ``password`` to the
   specified POST data fields.
-  **urlChecker**: The URL checker class or object. Default is
   ``DefaultUrlChecker``.
-  **useRegex**: Whether or not to use regular expressions for URL
   matching. Default is ``false``.
-  **checkFullUrl**: Whether or not to check full URL including the query
  string. Useful when a login form is on a different subdomain. Default is
  ``false``. This option does not work well when preserving unauthenticated
  redirects in the query string.

.. warning::
    If you use the array syntax for the URL, the URL will be
    generated by the CakePHP router. The result **might** differ from what you
    actually have in the request URI depending on your route handling. So
    consider this to be case sensitive!

Token
-----

The token authenticator can authenticate a request based on a token that
comes along with the request in the headers or in the request
parameters.

Configuration options:

-  **queryParam**: Name of the query parameter. Configure it if you want
   to get the token from the query parameters.
-  **header**: Name of the header. Configure it if you want to get the
   token from the header.
-  **tokenPrefix**: The optional token prefix.

An example of getting a token from a header, or query string would be::

    $service->loadAuthenticator('Authentication.Token', [
        'header' => 'Authorization',
        'queryParam' => 'token',
        'tokenPrefix' => 'Token'
    ]);

The above would read the ``token`` GET parameter or the ``Authorization`` header
as long as the token was preceded by ``Token`` and a space.

JWT
===

The JWT authenticator gets the `JWT token <https://jwt.io/>`__ from the
header or query param and either returns the payload directly or passes
it to the identifiers to verify them against another datasource for
example.

-  **header**: The header line to check for the token. The default is
   ``Authorization``.
-  **queryParam**: The query param to check for the token. The default
   is ``token``.
-  **tokenPrefix**: The token prefix. Default is ``bearer``.
-  **algorithms**: An array of hashing algorithms for Firebase JWT.
   Default is an array ``['HS256']``.
-  **returnPayload**: To return or not return the token payload directly
   without going through the identifiers. Default is ``true``.
-  **secretKey**: Default is ``null`` but you’re **required** to pass a
   secret key if you’re not in the context of a CakePHP application that
   provides it through ``Security::salt()``.

If you want to identify the user based on the ``sub`` (subject) of the
token you can use the JwtSubject identifier::

   $service = new AuthenticationService();
   $service->loadIdentifier('Authentication.JwtSubject');
   $service->loadAuthenticator('Authentication.Jwt', [
       'returnPayload' => false
   ]);

HttpBasic
=========

See https://en.wikipedia.org/wiki/Basic_access_authentication

Configuration options:

-  **realm**: Default is ``$_SERVER['SERVER_NAME']`` override it as
   needed.

HttpDigest
==========

See https://en.wikipedia.org/wiki/Digest_access_authentication

Configuration options:

-  **realm**: Default is ``null``
-  **qop**: Default is ``auth``
-  **nonce**: Default is ``uniqid(''),``
-  **opaque**: Default is ``null``

Cookie Authenticator aka "Remember Me"
======================================

The Cookie Authenticator allows you to implement the “remember me”
feature for your login forms.

Just make sure your login form has a field that matches the field name
that is configured in this authenticator.

To encrypt and decrypt your cookie make sure you added the
EncryptedCookieMiddleware to your app *before* the
AuthenticationMiddleware.

Configuration options:

-  **rememberMeField**: Default is ``remember_me``
-  **cookie**: Array of cookie options:

   -  **name**: Cookie name, default is ``CookieAuth``
   -  **expire**: Expiration, default is ``null``
   -  **path**: Path, default is ``/``
   -  **domain**: Domain, default is an empty string \`\`
   -  **secure**: Bool, default is ``false``
   -  **httpOnly**: Bool, default is ``false``
   -  **value**: Value, default is an empty string \`\`

-  **fields**: Array that maps ``username`` and ``password`` to the
   specified identity fields.
-  **urlChecker**: The URL checker class or object. Default is
   ``DefaultUrlChecker``.
-  **loginUrl**: The login URL, string or array of URLs. Default is
   ``null`` and all pages will be checked.
-  **passwordHasher**: Password hasher to use for token hashing. Default
   is ``DefaultPasswordHasher::class``.

OAuth
=====

There are currently no plans to implement an OAuth authenticator. The
main reason for this is that OAuth 2.0 is not an authentication
protocol.

Read more about this topic
`here <https://oauth.net/articles/authentication/>`__.

We will maybe add an OpenID Connect authenticator in the future.

Events
======

There is only one event that is fired by authentication:
``Authentication.afterIdentify``.

If you don’t know what events are and how to use them `check the
documentation <https://book.cakephp.org/3.0/en/core-libraries/events.html>`__.

The ``Authentication.afterIdentify`` event is fired by the
``AuthenticationComponent`` after an identity was successfully
identified.

The event contains the following data:

-  **provider**: An object that implements
   ``\Authentication\Authenticator\AuthenticatorInterface``
-  **identity**: An object that implements ``\ArrayAccess``
-  **service**: An object that implements
   ``\Authentication\AuthenticationServiceInterface``

The subject of the event will be the current controller instance the
AuthenticationComponent is attached to.

But the event is only fired if the authenticator that was used to
identify the identity is *not* persistent and *not* stateless. The
reason for this is that the event would be fired every time because the
session authenticator or token for example would trigger it every time
for every request.

From the included authenticators only the FormAuthenticator will cause
the event to be fired. After that the session authenticator will provide
the identity.

URL Checkers
============

Some authenticators like ``Form`` or ``Cookie`` should be executed only
on certain pages like ``/login`` page. This can be achieved using URL
Checkers.

By default a ``DefaultUrlChecker`` is used, which uses string URLs for
comparison with support for regex check.

Configuration options:

-  **useRegex**: Whether or not to use regular expressions for URL
   matching. Default is ``false``.
-  **checkFullUrl**: Whether or not to check full URL. Useful when a
   login form is on a different subdomain. Default is ``false``.

A custom URL checker can be implemented for example if a support for
framework specific URLs is needed. In this case the
``Authentication\UrlChecker\UrlCheckerInterface`` should
be implemented.

For more details about URL Checkers :doc:`see this documentation
page </url-checkers>`.

Getting the Successful Authenticator or Identifier
==================================================

After a user has been authenticated you may want to inspect or interact with the
Authenticator that successfully authenticated the user::

    // In a controller action
    $service = $this->request->getAttribute('authentication');

    // Will be null on authentication failure, or an authenticator.
    $authenticator = $service->getAuthenticationProvider();

You can also get the identifier that identified the user as well::

    // In a controller action
    $service = $this->request->getAttribute('authentication');

    // Will be null on authentication failure, or an identifier.
    $identifier = $service->getIdentificationProvider();


Using Stateless Authenticators with stateful Authenticators
===========================================================

When using ``HttpBasic`` or ``HttpDigest`` with other authenticators, you should
remember that these authenticators will halt the request when authentication
credentials are missing or invalid. This is necessary as these authenticators
must send specific challenge headers in the response. If you want to combine
``HttpBasic`` or ``HttpDigest`` with other authenticators, you may want to
configure these authenticators as the *last* authenticators::

    use Authentication\AuthenticationService;

    // Instantiate the service
    $service = new AuthenticationService();

    // Load identifiers
    $service->loadIdentifier('Authentication.Password', [
        'fields' => [
            'username' => 'email',
            'password' => 'password'
        ]
    ]);

    // Load the authenticators leaving Basic as the last one.
    $service->loadAuthenticator('Authentication.Session');
    $service->loadAuthenticator('Authentication.Form');
    $service->loadAuthenticator('Authentication.HttpBasic');
