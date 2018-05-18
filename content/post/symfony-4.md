---
title: "Symfony 4 Installation and new features"
date: 2018-05-18T14:42:37+02:00
draft: false
---

**Symfony 4** is the newest version of the one of the most famous PHP frameworks.
It has been released on the 30th of November 2017 and it's having a great success!
One of the biggest difference with the older versions is the presence of very small dependencies installed when you init your Symfony 4 project.
Symfony 4 is so tiny that [Silex](https://silex.symfony.com/) project has been dismissed.


## How to Install Symfony 4

In order to create a new Symfony 4 application you need to have **PHP 7.1** or higher and Composer installed.
With **Composer** installed you can create the project launching this command into your console:

```
composer create-project symfony/website-skeleton my-project
```

This is the command for traditional web applications that downloads a base skeleton of a Symfony project with a ready to start default configuration.
If you are building microservices, console applications or APIs, consider using the much simpler skeleton project.

```
composer create-project symfony/skeleton my-project
```

In this version there aren't bundles installed so it's smaller.


## Directory Structure

Symfony 4 has changed a bit its directory structure according to other frameworks and community requests, so now the new directory structure is:

![Directory Structure Image](https://alessandrominoccheri.github.io/img/symfony_folder_strcuture.jpg)

**app/**
The application configuration, templates and translations, where you can find the AppKernel file, the main entry point of the application configuration.

**bin/**
Executable files (e.g. bin/console).

**src/**
The project's PHP code, where you have controllers, views and entities directories.
In this folder there isn't a main bundle because Symfony 4's idea is to have a single project instead of more packages in a project.

**tests/**
Automatic tests (e.g. Unit tests with PHPUnit or Behat or others).

**var/**
Generated files (cache, logs, etc.).

**vendor/**
External libraries installed by composer.

**public/**
The web root directory where are stored public and static files like images, stylesheets and JavaScript files, this folder is the old "web" folder.


## Environment Variables

When designing your system, you may wish to have different variables that can be not the same as in other environments.
In previous Symfony versions you put that variables into a file called **parameters.yml** and in parameters.yml.yourenv. Now that file doesn't exist (you can obviously change your code to use it but it's a little bit complex) and you need to write those variables in a file called .env in the project root.
Obviously you can have different .env files but it depends on how many environments you have into your application.
Here is a simple **.env** file:

```
DB_USER=root
DB_PASS=pass
```

You can also use variables inside it like this:

```
DB_USER=root
DB_PASS=${DB_USER}pass
```

The component that reads those files is the Dotenv component that parses .env files to make environment variables stored in them accessible via **getenv()**, **$_ENV** or **$_SERVER**.
If you don't already have it in your project you can launch this command to form your CLI to use it:

```
composer require symfony/dotenv
```

To get the value of an environment variable you can use this syntax inside your code:

```php
$dbUser = getenv('DB_USER');
```

Remember that Symfony Dotenv never overwrites existing environment variables.


## Autowiring

**Autowiring** is one of the most powerful feature in Symfony 4 (you can use it since Symfony 3.3 version).
It allows you to manage services in the container with minimal configuration, without specifying, for example, all the arguments that have to be passed to your service (now also controllers are services!).
So if you have a service class where you need to pass into __construct method some other services you only need to enable autowiring and Symfony will do it for you!

Example:

```php
namespace App\Service;
use App\Util\Bar;
use Psr\Log\LoggerInterface;
class Foo
{
    private $logger;
    public function __construct(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }
}
```

In the older method you needed to do this into your **services.yml**

```
app.foo:
    class:     AppBundle\Services\Foo
    arguments: ['@logger']
```

In this new version you can do like this:

```
app.foo:
    class: AppBundle\Services\Foo
    public: true
```

So you need to specify only public as true. Imagine having plenty arguments to declare for many services, in this way you will gain spare time!
Basically, services aren't public so you need to render it true into services.yml if you want also in this way:

```
services:
    _defaults:
        autowire: true
        autoconfigure: true
        public: true
```

This is the default configuration of your services and as you can see it's under _defaults.
If you want to have different configuration for your service, for example a private service, you can write this:

```
services:
    _defaults:
        autowire: true
        autoconfigure: true
        public: true

    App\Services\YourService:
        class: App\Services\YourService
        public: false
```

So the Autowire option indicates to Symfony to automatically inject dependencies in your services.
Autoconfigure option indicates to Symfony to automatically register your services as commands, event subscribers, etc.

Now also controllers are services so it's easy to make unit tests on it instead of using integration tests that are slower.
So now you can also inject services inside your controller like this:

```php
public function index(LoggerInterface $logger) 
{
    $logger->info('This is a injected service!');
    //code
}
```

In this action you inject **LoggerInterface** directly and you don't need to instantiate it inside the action!

## Flex and Recipes

Symfony Flex is a Composer plugin and it's called when you run require, update, and remove composer commands.
When you run those flex search commands inside **Symfony Flex**, the server will take it, install it and configure it for you if the package is on there.
The power of Flex is that you don't have to watch all the libraries' readme files to configure your package since there is a recipe that does it for you.
You don't need to add your new bundle into AppKernel.php because flex already did it when you install it.
If your package doesn't exist, it fallbacks on Composer standard behaviour.

Flex keeps track of the recipes it installed in a **symfony.lock** file, which must be committed to your code repository.
Recipes are stored in two different github repositories:

* [Recipe](https://github.com/symfony/recipes), is a curated list of recipes for high quality and maintained packages. Symfony Flex looks into this repository by default.

* [Recipes-Contrib](https://github.com/symfony/recipes-contrib), contains all the recipes created by the community. All of them are guaranteed to work, but their associated packages could be unmaintained. Symfony Flex ignores these recipes by default, but you can execute this command to start using them in your project:

```
composer config extra.symfony.allow-contrib true
```

Inside this site [Symfony](https://symfony.sh/) you can view the recipes complete list where you can find how to install them into your project.
Sometimes there is an alias so you can use it like this one to be faster and to remember the command easier:

```
composer require logger
```

A recipe contains a **manifest.json** file where the package configuration is stored. When Flex installs this package it gets the manifest.json file and applies default configuration to your project, so you can use the new library immediately without writing any configuration.


## The Messenger Component

The Messenger component helps applications send and receive messages to/from other applications or via message queues.
In order to install the component, you need to launch from your console:

```
composer require symfony/messenger
```

![Messenger Component](https://alessandrominoccheri.github.io/img/symfony_messenger_component.jpg)

### How does it work?

Sender serializes and sends a message to something. For example this something can be a message broker or a third party API.
The bus dispatches the message. The behaviour of the bus is in its ordered middleware stack. The component comes with a set of middleware that you can use.
When using the message bus with **Symfony's FrameworkBundle**, the following middleware are configured for you:

**LoggingMiddleware** (logs the processing of your messages)

**SendMessageMiddleware** (enables asynchronous processing)

**HandleMessageMiddleware** (calls the registered handle)

Once the message is dispatched to the bus it will be handled by a "message handler".
A message handler is a PHP callable (i.e. a function or an instance of a class) that will do the required processing for your message.
In order to send and receive messages, you will have to configure the adapter. The adapter will be responsible for communicating with your message broker or 3rd parties.
Then the receiver Receiver deserializes and forwards the message to the handler(s). This can be a message queue puller or an API endpoint.

For further information contact me or follow  me on:

[Twitter](https://twitter.com/minompi)
[GitHub](https://github.com/AlessandroMinoccheri)