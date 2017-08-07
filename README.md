# Azure SDK for PHP

## POC

https://packagist.org/packages/microsoft/windowsazure#dev-arm2

## Plan

1. Make a branch from the
   [azure-sdk-for-php](https://github.com/Azure/azure-sdk-for-php) master
   branch, for example
   [windowsazure](https://github.com/Azure/azure-sdk-for-php/tree/windowsazure), the same as a the
   current name of the
   [composer package](https://packagist.org/packages/microsoft/windowsazure).
1. Create a `readme.md` in the master branch which points to the old branch
   and the old composer package.
1. Publish a new empty package (version 0.0.0). What's the new name should
   be? **microsoft/azure**, **microsoft/azure-sdk**? For example,
   [microsoft/azure-storage](https://packagist.org/packages/microsoft/azure-storage).
1. Create the **Azure/azure-runtime-for-php** repository for common RunTime
   code.

## Requirements

1. Support for different PHP versions (see
   [PHP: Supported Versions](http://php.net/supported-versions.php)):
   - PHP 5.6
   - PHP 7.0
   - PHP 7.1
   - Hack
1. CI (Travis-CI, Appveyor?)
1. Profile support.
1. Descriptive code generation (instead of imperative).
1. An authentication. See the email thread
   `FW: Azure SDK common auth file support`.

## Wish List

1. One common core/run-time library for `Azure SDK for PHP` and
   `Azure Storage for PHP`.
1. Documentation generation for docs.microsoft.com.
1. Telemetry in the Core library
1. JSON-RPC support in the Core library.
1. API levels
   - sync/async
   - raw/not-raw
   - strongly/duck typed
   - fluent?
1. An AutoRest PHP code generator as a PHP tool (published on Composer).
1. PHP conventions http://www.php-fig.org/bylaws/psr-naming-conventions/

### Data Types

- Q. Why avoid arrays?
- A. Because they passed by value by default.

### API Level Example

```php
$s = $client.sync();
$a = $client.async();
$a = $client.syncRaw();
$result = $s.someFunction(...);
$asyncClient = $s.client.async();
```

By default

- sync
- not raw
- strongly typed

### For Developers

#### On Windows

1. Download **PHP 5.6 x86 Non Thread Safe** from
 [here](http://windows.php.net/download#php-5.6-nts-VC11-x86).
1. Unpack it to **c:\\php-5.6\\** folder.
1. Copy `php.ini-development` to `php.ini`.
1. Uncomment `extension_dir = "ext"`
1. Uncomment extensions:
   - `openssl`
   - `mbstring`
1. Run [install_composer.cmd](install_composer.cmd).

#### Run-Time Library Interface

```php
namespace Microsoft/Rest;

interface OperationInterface
{
    /**
     * @param  array $parameters
     * @return mixed
     */
    function call(array $parameters);
}

interface ClientInterface
{
    /**
     * @param  string             $operationId
     * @return OperationInterface
     */
    function createOperation($operationId);
}

interface RunTimeInterface
{
    /**
     * @param  array           $swaggerObjectData
     * @param  array           $sharedParameterMap
     * @return ClientInterface
     */
    function createClient(array $swaggerObjectData, array $sharedParameterMap);
}

final class AzureStatic
{
    /**
     * @param  string $clientId
     * @param  string $tenantId
     * @param  string $clientSecret
     * @return RunTimeInterface
     */
    function create($clientId, $tenantId, $clientSecret);

    private function __constructor() {}
}
```

#### An Example of Run-Time Interface Usage

```php
class RedisClient
{
    /**
     * @param string $subscriptionId
     */
    function __constructor(RunTimeInterface $runTime, $subscriptionId)
    {
        $this->_client = $runTime->createFromData(self::_SWAGGER_OBJECT_DATA, $subscriptionId);
        $this->_create_operation = $this->_client->createOperation('create');
    }

    /**
     * @param string $name
     */
    function create($name)
    {
        return $this->_create_operation->call(['name' => $name]);
    }

    /**
     * @var \Microsoft\Rest\OperationInterface
     */
    private $_create_operation;

    const _SWAGGER_OBJECT_DATA = [
        ...
    ];
}
```

#### Data Types

|type   |format           |properties          |PHP Type Information|PHP Run-Time Type|
|-------|-----------------|--------------------|--------------------|-----------------|
|boolean|                 |                    |BooleanType         |boolean          |
|string |                 |                    |StringType          |string           |
|string |byte             |                    |Base64Type          |string           |
|string |binary           |                    |BinaryType          |string           |
|string |date             |                    |DateType            |string           |
|string |date-time        |                    |DateTimeType        |string           |
|string |password         |                    |PasswordType        |string           |
|string |duration         |                    |DurationType        |string           |
|string |uuid             |                    |UuidType            |string           |
|string |date-time-rfc1123|                    |DateTimeRfc1123Type |string           |
|string |                 |enum                |EnumType            |string           |
|integer|int32            |                    |Int32Type           |integer          |
|integer|int64            |                    |Int64Type           |string           |
|number |float            |                    |FloatType           |float            |
|number |double           |                    |DoubleType          |float            |
|number |decimal          |                    |DecimalType         |string           |
|array  |                 |items               |ArrayType           |iterable         |
|object |                 |                    |ObjectType          |mixed            |
|object |                 |additionalProperties|MapType             |iterable         |
|       |                 |properties          |ClassType           |ArrayAccess      |

## API Levels

- dictionaries - PHP 5.6
- static types - PHP 7.1
  ```php
  class X
  {
      public $a;

      function setA(string $a) : this
      {
          return $this;
      }

      function unsetA() : this
      {
          unset($this->a);
          return $this;
      }

      // optional
      function getA() : string
      {
          return $this->a;
      }

      // optional
      function isASet() : bool
      {
          return isset($this->a);
      }

      // optional
      function getAOrNull() : string
      {
          return isset($this->a) ? $this->a : null;
      }
  }
  ```