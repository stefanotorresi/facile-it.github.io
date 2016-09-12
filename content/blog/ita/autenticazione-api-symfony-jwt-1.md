---
authors: ["algatux"]
comments: true
date: "2016-04-14"
draft: true
image: "/images/cover.jpg"
menu: ""
share: true
categories: [Italiano, JWT, Symfony]
title: "Autenticazione API con Symfony e JWT [1/2]"

languageCode: "en-En"
type: "post"

---
As promised in the previous post [Json Web Token](http://engineering.facile.it/json-web-joken/), with this article I want to explain an example of JWT authentication on Symfony 3.0.
In this first part I will focus on the firewall setup and services necessary to realize a JWT login and token generation with a stateless implementation, without the use of pre-packaged bundles.
All you will see is the result of experiments done during my "journey" to discover the JSON Web Tokens and Symfony framework; and for this reason I preferred not to use third-party bundle, but instead realize a "personal solution".

## Dipendenze
The package to install, `lcobucci/jwt` is listed among the trusted JWT libraries and is currently the most complete as support for claims, signing and verification methods. Essentially it allows JWT Tokens generation and validation.

Add the dependency with composer: 
```
composer require lcobucci/jwt
```

## Design
This solution involves the creation of two additional firewalls to the Symfony application: 

- The first one with the purpose of authenticate users, and give them a token 
- The second acts as protection for our API routes assuring that a valid token in passed in. 

Both firewalls will be stateless, so they will not maintain user information over the request lifecycle.

Lets add this lines into `security.yml`:
``` yaml
## WS API
ws_login:
  pattern: ^/ws/login$
  stateless: true
  anonymous: true
  simple_form:
    provider: api_user_provider
    authenticator: ws.security_authentication.json_login_authenticator
    check_path: ws_login_check
    username_parameter: username
    password_parameter: password
    success_handler:  ws.security_authentication.success_handler
    failure_handler:  ws.security_authentication.failure_handler
    require_previous_session: false
  logout: false
  context: ws_api

ws_api:
  pattern: ^/ws
  stateless: true
  simple_preauth:
    authenticator: ws.security.api_authenticator
  provider: api_user_provider
```

`ws_login` its our login route, in this case i choose to use a `simple_form` which differs from `form_login` for ability to specify a custom authentication service that we will see shortly. Moreover, the UserProvider is also personalized, with the purpose to avoid session persistence.
The special feature of this setup is the use of ` success handler` and `failure_handler` that simple__form is designed to call in case of success or failure of the login request.

`ws__api` is instead the firewall that protects our routes, here, we will see another custom authentication service, and it will also make use of the custom provider` api_user_provider`.

## User Provider
This user provider just looks for the username by performing a query on the database through the UserRepository and ensures that authentication remains stateless.
``` php
<?php
declare(strict_types = 1);
namespace WsBundle\Security;

use AppBundle\Entity\User;
use AppBundle\Repository\UserRepository;
use Symfony\Component\Security\Core\Exception\UnsupportedUserException;
use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Core\User\UserProviderInterface;
/**
 * Class ApiUserProvider
 * @package AppBundle\Security
 */
class ApiUserProvider implements UserProviderInterface
{
    /** @var UserRepository */
    private $userRepository;
    /**
     * ApiUserProvider constructor.
     * @param UserRepository $userRepository
     */
    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }
    /**
     * @param string $username
     * @return User|null
     */
    public function loadUserByUsername($username)
    {
        return $this->userRepository->findByUsername($username);
    }
    /**
     * @param UserInterface $user
     * @return UserInterface|void
     */
    public function refreshUser(UserInterface $user)
    {
        // this is used for storing authentication in the session
        // but in this example, the token is sent in each request,
        // so authentication can be stateless. Throwing this exception
        // is proper to make things stateless
        throw new UnsupportedUserException();
    }
    /**
     * @param string $class
     * @return bool
     */
    public function supportsClass($class)
    {
        return User::class === $class;
    }
}
```

## Login Authenticator
The request forwarded to the authenticator must contain a JSON payload with user's username and password. The service will then take care of extract the payload, verify it and proceed with the autentication using the just seen provider; checking the authority of credentials. 
``` php
<?php
declare(strict_types=1);
namespace WsBundle\Security\Authentication;

use AppBundle\Repository\UserRepository;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Authentication\Token\UsernamePasswordToken;
use Symfony\Component\Security\Core\Encoder\UserPasswordEncoderInterface;
use Symfony\Component\Security\Core\Exception\CustomUserMessageAuthenticationException;
use Symfony\Component\Security\Core\Exception\UsernameNotFoundException;
use Symfony\Component\Security\Core\User\UserProviderInterface;
use Symfony\Component\Security\Http\Authentication\SimpleFormAuthenticatorInterface;
use WsBundle\Exceptions\Request\InvalidLoginRequestPayload;
use WsBundle\Models\LoginRequestPayload;
use WsBundle\Security\Jwt\TokenGenerator;

/**
 * Class JsonLoginAuthenticator
 * @package WsBundle\Security
 */
class JsonLoginAuthenticator implements SimpleFormAuthenticatorInterface
{

    const LOGIN_REQUEST_MALFORMED = 'Login request is not in the correct format or malformed.';

    /** @var UserRepository */
    private $userRepository;

    /** @var TokenGenerator  */
    private $tokenGenerator;

    /** @var UserPasswordEncoderInterface  */
    private $encoder;

    /**
     * ApiUserProvider constructor.
     * @param UserRepository $userRepository
     * @param TokenGenerator $tokenGenerator
     * @param UserPasswordEncoderInterface $encoder
     */
    public function __construct(
        UserRepository $userRepository,
        TokenGenerator $tokenGenerator,
        UserPasswordEncoderInterface $encoder
    )
    {
        $this->userRepository = $userRepository;
        $this->tokenGenerator = $tokenGenerator;
        $this->encoder = $encoder;
    }

    /**
     * @param TokenInterface $token
     * @param UserProviderInterface $userProvider
     * @param $providerKey
     * @return UsernamePasswordToken
     */
    public function authenticateToken(TokenInterface $token, UserProviderInterface $userProvider, $providerKey)
    {
        try {
            $user = $userProvider->loadUserByUsername($token->getUsername());
        } catch (UsernameNotFoundException $e) {
            throw new CustomUserMessageAuthenticationException('Invalid username or password');
        }

        $passwordValid = $this->encoder->isPasswordValid($user, $token->getCredentials());

        if ($passwordValid) {
            return new UsernamePasswordToken(
                $user,
                $user->getPassword(),
                $providerKey,
                $user->getRoles()
            );
        }

        throw new CustomUserMessageAuthenticationException('Invalid username or password');
    }

    /**
     * @param TokenInterface $token
     * @param $providerKey
     * @return bool
     */
    public function supportsToken(TokenInterface $token, $providerKey)
    {
        return $token instanceof UsernamePasswordToken && $token->getProviderKey() === $providerKey;
    }

    /**
     * @param Request $request
     * @param $username
     * @param $password
     * @param $providerKey
     * @return UsernamePasswordToken
     */
    public function createToken(Request $request, $username, $password, $providerKey)
    {
        try{
            $payload = $this->getRequestBody($request);
        } catch (InvalidLoginRequestPayload $e){
            throw new CustomUserMessageAuthenticationException($e->getMessage());
        }

        return new UsernamePasswordToken($payload->getUsername(), $payload->getPassword(), $providerKey);
    }

    /**
     * @param Request $request
     * @return LoginRequestPayload
     * @throws InvalidLoginRequestPayload
     */
    private function getRequestBody(Request $request): LoginRequestPayload
    {
        if ($request->getContentType() === 'json') {
            $payload = json_decode($request->getContent(), true);

            return new LoginRequestPayload(
                $this->extractParameterFromRawPayload($payload, 'username'),
                $this->extractParameterFromRawPayload($payload, 'password')
            );
        }

        throw new InvalidLoginRequestPayload(self::LOGIN_REQUEST_MALFORMED . ' Required Json');

    }

    /**
     * @param array $payload
     * @param string $parameter
     * @return string
     * @throws InvalidLoginRequestPayload
     */
    private function extractParameterFromRawPayload(array $payload, string $parameter): string
    {
        if (! isset($payload[$parameter])) {
            throw new InvalidLoginRequestPayload(self::LOGIN_REQUEST_MALFORMED . sprintf(' You must specify %s to login', $parameter));
        }

        return $payload[$parameter];
    }

}
```

## Form Handlers
In the case where the authenticator gives his agree, the `SuccessHandler` will be called. His role is to generate a JWT Token for the logging user using a `TokenGenerator` service, and to send it back into a Json Response.
``` php
<?php
declare(strict_types=1);
namespace WsBundle\Security\Authentication\Handlers;

use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Http\Authentication\AuthenticationSuccessHandlerInterface;
use WsBundle\Security\Jwt\TokenGenerator;

/**
 * Class SuccessHandler
 * @package WsBundle\Security\Authentication
 */
class SuccessHandler implements AuthenticationSuccessHandlerInterface
{

    /** @var TokenGenerator  */
    private $tokenGenerator;

    /**
     * SuccessHandler constructor.
     * @param TokenGenerator $tokenGenerator
     */
    public function __construct(TokenGenerator $tokenGenerator)
    {
        $this->tokenGenerator = $tokenGenerator;
    }

    /**
     * This is called when an interactive authentication attempt succeeds. This
     * is called by authentication listeners inheriting from
     * AbstractAuthenticationListener.
     *
     * @param Request $request
     * @param TokenInterface $token
     *
     * @return JsonResponse
     */
    public function onAuthenticationSuccess(Request $request, TokenInterface $token): JsonResponse
    {
        $user = $token->getUser();
        $jwtToken = $this->tokenGenerator->createToken($user);

        return new JsonResponse(
            [
                'token' => $jwtToken->__toString()
            ],
            201
        );
    }

}
```

Otherwise a `FailureHandler` will be called, sending back to the user a Response with details about the authentication failure.
``` php
<?php
declare(strict_types=1);
namespace WsBundle\Security\Authentication\Handlers;

use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Security\Core\Exception\AuthenticationException;
use Symfony\Component\Security\Http\Authentication\AuthenticationFailureHandlerInterface;

/**
 * Class FailureHandler
 * @package WsBundle\Security\Authentication
 */
class FailureHandler implements AuthenticationFailureHandlerInterface
{

    /**
     * This is called when an interactive authentication attempt fails. This is
     * called by authentication listeners inheriting from
     * AbstractAuthenticationListener.
     *
     * @param Request $request
     * @param AuthenticationException $exception
     *
     * @return JsonResponse
     */
    public function onAuthenticationFailure(Request $request, AuthenticationException $exception): JsonResponse
    {
        return new JsonResponse([
            'failure' => 'authentication error',
            'error' => $exception->getMessage(),
        ], 401);
    }
}
```

## Token Generator
Eccoci arrivati al servizio adibito alla creazione dei Token. In questa classe vengono injettati:
 
- il `Builder` del token, un signer di nostra scelta tra quelli disponibili
- la secret key 
- un' array di configurazione dei claims.

In questo esemio vengono impostati i claim standard `iss`, `aud`, `exp` e `nbf`; mentre viene settato un claim privato `data` contenente i dati dell'utente.
``` php
<?php
declare(strict_types=1);
namespace WsBundle\Security\Jwt;

use AppBundle\Entity\User;
use Carbon\Carbon;
use Lcobucci\JWT\Builder;
use Lcobucci\JWT\Signer;
use Lcobucci\JWT\Token;

/**
 * Class TokenGenerator
 * @package WsBundle\Security\Jwt
 */
class TokenGenerator implements AuthTokenFactoryInterface
{

    /** @var Builder  */
    private $tokenBuilder;

    /** @var Signer  */
    private $signer;

    /** @var string  */
    private $secret;

    /** @var array  */
    private $config;

    /**
     * TokenGenerator constructor.
     * @param Builder $tokenBuilder
     * @param Signer $signer
     * @param string $secret
     * @param array $config
     */
    public function __construct(Builder $tokenBuilder, Signer $signer, string $secret, array $config)
    {
        $this->tokenBuilder = $tokenBuilder;
        $this->signer = $signer;
        $this->secret = $secret;
        $this->config = $config;
    }

    /**
     * @param User $user
     * @return Token
     */
    public function createToken(User $user): Token
    {
        $issued = Carbon::now();
        $expire = Carbon::now()->addSeconds((int) $this->config['expire']);
        $notBefore = Carbon::now()->addSeconds((int) $this->config['notbefore']);

        return $this->tokenBuilder
            ->setIssuer($this->config['issuer'])
            ->setAudience($this->config['audience'])
            ->setId($this->config['appid'], true)
            ->setIssuedAt($issued->getTimestamp())
            ->setNotBefore($notBefore->getTimestamp())
            ->setExpiration($expire->getTimestamp())
            ->set('data', [
                "uid" => $user->getId(),
                "uidentifier" => $user->getUsername(),
            ])
            ->sign($this->signer, $this->secret)
            ->getToken();
    }

}
```

### Registrare i servizi

``` haml
# src/WsBundle/Resources/config/security.yml
services:

  # Authentication
  ws.security.api_user_provider:
    class: WsBundle\Security\ApiUserProvider
    arguments:
      - "@app.user.repository"

  ws.security_authentication.json_login_authenticator:
    class: WsBundle\Security\Authentication\JsonLoginAuthenticator
    arguments:
      - "@app.user.repository"
      - "@ws.security_jwt.generator"
      - "@security.password_encoder"

  ws.security_jwt.generator:
    class: WsBundle\Security\Jwt\TokenGenerator
    arguments:
      - "@ws.security.lcobucci.jwt.builder"
      - "@ws.security.lcobucci.jwt.hmac_signer"
      - "%secret%"
      - "%jwt_validation%"

  ws.security_authentication.success_handler:
      class: WsBundle\Security\Authentication\Handlers\SuccessHandler
      arguments:
        - "@ws.security_jwt.generator"

  ws.security_authentication.failure_handler:
      class: WsBundle\Security\Authentication\Handlers\FailureHandler

  ## LIBRARY SERVICES
  ws.security.lcobucci.jwt.builder:
    class: Lcobucci\JWT\Builder

  ws.security.lcobucci.jwt.hmac_signer:
    class: Lcobucci\JWT\Signer\Hmac\Sha256
```

## Configurazione
``` yaml
# app/config/parameters.yml
...

jwt_validation:
    issuer: ~
    audience: ~
    appid: ~
    expire: 300
    notbefore: 1
```
