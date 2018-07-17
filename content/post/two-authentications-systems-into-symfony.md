---
title: "Two Authentications Systems Into Symfony"
date: 2018-07-17T15:43:08+02:00
draft: false
---

Many times in your application you have a login system based on **JWT Token**. 
At a certain moment you need your application to be able to answer to a rest api request made by another application. So you can give the possibility to login with the JWT Token to it but it's not safe.
So you need to implement an api token authentication for your application but this means that your application has to **manage both authenticators**: JWT and **Api Token**.

![Directory Structure Image](https://alessandrominoccheri.github.io/img/symfony-authentication.jpg)

Well, how can you use both authentication system?

First step is to make the first authenticator work as if it were the only one.

**security.yml**

```
rest_api:
    pattern:   ^/api/
    stateless: true
    guard:
        authenticators:
           - lexik_jwt_authentication.jwt_token_authenticator
```

After that start to implement another authenticator into security.yml file like this:

```
rest_api:
    pattern:   ^/api/
    stateless: true
    guard:
        authenticators:
           - lexik_jwt_authentication.jwt_token_authenticator
           - AppBundle\Security\ApiKeyAuthenticator
        entry_point: lexik_jwt_authentication.jwt_token_authenticator
```

In this configuration you have specified a two different authenticators system and a entry point.
Note that the entry point is mandatory! Entry point highlights the principle authenticator system.

Now you need to add the field apiKey into your user entity, something like this:

```
/**
 * @var string
 *
 * @ORM\Column(name="api_key", type="string", length=255, nullable=false, unique=true)
 */
private $apiKey;
```

After that you need to create your custom ApiKeyAuthenticator, for example:

```
namespace AppBundle\Security;

use AppBundle\Entity\PvpUser;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Exception\AuthenticationException;
use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Core\User\UserProviderInterface;
use Symfony\Component\Security\Guard\AuthenticatorInterface;
use Symfony\Component\Security\Guard\Token\GuardTokenInterface;
use Symfony\Component\Security\Guard\Token\PostAuthenticationGuardToken;

class ApiKeyAuthenticator implements AuthenticatorInterface
{
    private $em;

    public function __construct(EntityManagerInterface $em)
    {
        $this->em = $em;
    }

    public function start(Request $request, AuthenticationException $authException = null)
    {
        return new Response('Auth header required', 401);
    }

    public function supports(Request $request)
    {
        return $request->headers->has('apikey');
    }

    public function getCredentials(Request $request)
    {
        return array(
            'token' => $request->headers->get('apikey'),
        );
    }

    public function getUser($credentials, UserProviderInterface $userProvider)
    {
        $apiKey = $credentials['token'];

        if (null === $apiKey) {
            return;
        }

        $user = $this->em->getRepository(User::class)->findOneByApiKey($apiKey);

        if (null == $user) {
            return null;
        }

        return $userProvider->loadUserByUsername($user->getUsername());
    }

    public function checkCredentials($credentials, UserInterface $user)
    {
        //here you can check password if necessary
        return true;
    }

    public function createAuthenticatedToken(UserInterface $user, $providerKey)
    {
        return new PostAuthenticationGuardToken(
            $user,
            $providerKey,
            $user->getRoles()
        );
    }

    public function onAuthenticationFailure(Request $request, AuthenticationException $exception)
    {
        return new Response('Authentication Failure', 401);
    }

    public function onAuthenticationSuccess(Request $request, TokenInterface $token, $providerKey)
    {
        return null;
    }

    public function supportsRememberMe()
    {
        return false;
    }
}
```

Explanation of functions

**start**

    This is called when an anonymous request accesses a resource that requires authentication. The job of this method is to return some response that "helps" the user start the authentication process.
     It must return a Response object

**supports**
    This function indicates if the authenticator supports the given Request
    It must return a boolean value

**getCredentials**
    This function gets the authentication credentials from the request and returns them into an array or variable, it depends on your system

**getUser**
    Returns a UserInterface object based on the credentials.
    It must return a UserInterface object or null.

**checkCredentials**
    Returns true if the credentials are valid.
    It must return a boolean value

**createAuthenticatedToken**
    Creates an authenticated token for the given user
    It must return a GuardTokenInterface object

**onAuthenticationFailure**
    This function is called when authentication has been executed, but has failed (e.g. wrong apikey).
    It must return a Response object or null

**onAuthenticationSuccess**
    This function is called when authentication has been executed successfully.
    It must return a Response object or null

**supportsRememberMe**
    This function indicates if authentication supports the remember me function
    It must return a boolean value


In this code we check in every request if there is the key apikey into the headers.
If that header exists the authenticator tries to load an user by the apikey stored into the database. Then if the user is found, checks if it is allowed to access the resource requested.

In this way you can use **both authenticator** for all routes.
But if you want to close only some routes you can specify many rules into your **security.yml**
