---
title: "Better performances, high security, deprecated functions: here's  all the news about PHP 7.2"
date: 2018-03-20T15:09:20+01:00
draft: false
---

**PHP 7.2** has many features about security, implementations and deprecated functions.
In addition, it introduces a performance increase (already in version 7 it was done a great work): PHP 7.2 is 20% faster than version 7.0 and 10% faster than version 7.1.

Here's all the **highlights** offered by this new version.

## Type hinting argument

Since PHP 5 it was possibile to specify the argument type passed to a function.

Example:

```php
class SimpleClass {
    public $foo = 'bar';
}

$simpleClass = new SimpleClass;

function test(SimpleClass $simpleClass){
    return $simpleClass->foo;
}

echo test($simpleClass);
```

Since PHP 7.1 if it was passed a non SimpleClass type value to the function it generated a 500 error.

With PHP 7.2 the **type hinting argument** can be declared as object, which means that it's a generic object.

Example:

```php
class SimpleClass {
    public $foo = 'bar';
}

class FirstClass extends SimpleClass {
    public $foo = 'baz';
}
class SecondClass extends SimpleClass{
    public $foo = 'foobar';
}

$firstClass = new FirstClass;
$secondClass = new SecondClass;

function test(object $arg)
{
    return $arg->var;
}

echo test($firstClass);

echo test($secondClass);

```

In this example the code runs declaring the argument object type and then a generic object so it doesn't generate a 500 error.
Furthermore, this features permits to have a type "variance" implemented in the subclass for example.

Object becomes a key value from version 7.2, so be sure to not use it for class names, interfaces or traits.


## Type hinting return value

Since version 7.2 it is possible to declare **type hint return value** as object.

Example:

```php
function test($arg) :object
{
    return new Foo($arg);
}
```


## Abstract methods and re-writable interfaces

PHP 7.2 allows to **overwrite abstract methods and interfaces** which is to omit the declaration type of a parameter in an abstract method extended by the parent class.

Example:

```php
abstract class foo
{
    abstract function baz(string $yourVar);
}

abstract class bar extends foo
{
    abstract function baz($yourVar) : int;
}
```

The same functionality also applies to interfaces, for example:

```php
interface foo
{
    public function test(string $yourVar);
}

class bar implements foo
{
    public function test($yourVar)
    {
        //your code
    }
}
```

This new feature allows to update parent classes without updating all its subclasses.

Probably this is a marginal improvement  (it will never be used so frequently).


## Comma into syntax at the end of use statement's lists 

Comma at the end of the last array element it's valid in the PHP syntax, and sometimes it's encouraged to add easily new elements and avoid analysis errors by missing comma.

Since PHP 7.2 version it is possible to use commas even for the "use" inside classes. 

Example: 

```php
use Foo\Bar\{
    Foo,
    Bar,
    Baz,
};
```

## Argon2 in password's hash

Argon2 is a powerful hashing algorithm that was selected as the 2015 Password Hashing Competition winner, and PHP 7.2 introduces as a secure substitute for the Bcrypt algorithm.  

The new PHP version introduces the PASSWORD_ARGON2I constant, which now can be used in password_ * function, following this syntax: 

```php
password_hash('secretpassword', PASSWORD_ARGON2I);
```

Despite Bcrypt, which requires only one factor, Argon2 considers three different factors:

* A memory cost that defines Kib number to use during hashing (standard values are 1 << 10 or 1024 KiB or 1 MiB);
* A time cost that defines the number of interactions of the hashing algorithm (standard value is 2); 
* A parallelism factor, that sets the parallel thread number which will be used during hashing (standard value is 2);

Three new costants define standard cost's factors:

    1. PASSWORD_ARGON2_DEFAULT_MEMORY_COST

    2. PASSWORD_ARGON2_DEFAULT_TIME_COST

    3. PASSWORD_ARGON2_DEFAULT_THREADS

Example:

```php
$options = ['memory_cost' => 1<<11, 'time_cost' => 4, 'threads' => 2];
password_hash('secretpassword', PASSWORD_ARGON2I, $options);
```

## Libsodium as part of the PHP Core

Since version 7.2, PHP includes the **Sodium library** in its core. Libsodium is a cross-platform library and cross-languages for encrypting, decrypting, signatures, password hashing and much more.
The library was previously available by PECL.
PHP 7.2 is the first programming language that adds modern encrypting in its libraries.


## Count scalar values

Did you know that it is possible to count scalar values? In reality this is insignificant , as the function count() returns only 1 for scalars.

Example:

```php
var_dump(count(null)); // int(0)
var_dump(count(0)); // int(1)
var_dump(count(4)); // int(1)
var_dump(count('4')); // int(1)
```

Since PHP 7.2, it will show a message:

```
Warning: count(): The parameter must be an array or an object that implements Countable into / in / 4aIl2 on line 3.
```

## New features

PHP 7.2 introduces some new features which can be very useful: ftp_append(), hash_hmac_algos(), imagesetclip(), imagegetclip(), imageopenpolygon(), imageresolution(), imagecreatefrombmp(), imagebmp().


## Deprecated functions

The **assert()** function verifies data assertion and takes appropriate actions if the result is FALSE.
Using assert() with a string argument it's now deprecated because it opens a RCE vulnerability.

The **__autoload** function has been substituted by spl_autoload_register in PHP 5.1.
Now it's generated a deprecation message during compilation when it's revealed.

The **$php_errormsg** variable is created in the local scope when a non-fatal error is thrown. Since PHP 7.2 error_get_last and error_clear_last should be used instead.

**create_function()** allows to create a function with a generated function name, an argument list and the code body passed as arguments. Because of security problems and inadequate performances, it has been signed as deprecated (using enclosure is otherwise suggested).

**each()** is used to iterate on an array similar to foreach() that is 10 times faster.

**$errcontext** is an array that contains local existing variables at the error generation moment.

**gmp_random()**  is considered to be dependent on a platform and it will be deprecated. It's otherwise suggested to use gmp_random_bits () and gmp_random_rage ().

For **mbstring.func_overload**, the ini setting inserted into a default value which differs from zero has been signed as deprecated.

**parse_str()** analyzes a query string in an array if a second argument is passed or if the local symbol table isn't used. Because the dynamical variables setting of the function isn't encouraged for security reasons, the use of parse_str() without a second argument will generate a deprecation advice.

**(unset)cast** is an expression that returns always null and is considered not useful.

For further information contact me or follow  me on:

[Twitter](https://twitter.com/minompi)
[GitHub](https://github.com/AlessandroMinoccheri)

