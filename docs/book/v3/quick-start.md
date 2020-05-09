# laminas-hydrator

Hydration is the act of populating an object from a set of data.

laminas-hydrator is a simple component to provide mechanisms both for hydrating
objects, as well as extracting data sets from them.

The component consists of interfaces, and several implementations for common use cases.

## Base Interfaces

### ExtractionInterface

```php
namespace Laminas\Hydrator;

interface ExtractionInterface
{
    /**
     * Extract values from an object
     *
     * @return mixed[]
     */
    public function extract(object $object) : array;
}
```

### HydrationInterface

```php
namespace Laminas\Hydrator;

interface HydrationInterface
{
    /**
     * Hydrate $object with the provided $data.
     *
     * @param mixed[] $data
     * @return object The implementation should return an object of any type.
     *     By purposely omitting the return type from the signature,
     *     implementations may choose to specify a more specific type.
     */
    public function hydrate(array $data, object $object);
}
```

### HydratorInterface

```php
namespace Laminas\Hydrator;

interface HydratorInterface extends
    ExtractionInterface,
    HydrationInterface
{
}
```

## Usage

Usage involves instantiating the hydrator, and then passing information to it.

```php
use Laminas\Hydrator;
$hydrator = new Hydrator\ArraySerializable();

// To hydrate an object from values in an array:
$object = $hydrator->hydrate($data, new ArrayObject());

// or, going the other way and extracting the values from an object as an array:
$data = $hydrator->extract($object);
```

## Available Implementations

### Laminas\\Hydrator\\ArraySerializable

Follows the definition of `ArrayObject`. Objects must implement either the `exchangeArray()` or
`populate()` methods to support hydration, and the `getArrayCopy()` method to support extraction.

### Laminas\\Hydrator\\ClassMethods

Any data key matching a setter method will be called in order to hydrate; any method matching a
getter method will be called for extraction, according to the following rules:

- `is*()`, `has*()`, and `get*()` methods will be extracted, and the method
  prefix will be removed from the property name.
- `set*()` methods will be used when hydrating properties.

### Laminas\\Hydrator\\DelegatingHydrator

Composes a hydrator locator, and will delegate `hydrate()` and `extract()` calls
to the appropriate one based upon the class name of the object being operated
on.

```php
// Instantiate each hydrator you wish to delegate to
$albumHydrator = new Laminas\Hydrator\ClassMethods();
$artistHydrator = new Laminas\Hydrator\ClassMethods();

// Map the entity class name to the hydrator using the HydratorPluginManager.
// In this case we have two entity classes, "Album" and "Artist".
$hydrators = new Laminas\Hydrator\HydratorPluginManager;
$hydrators->setService('Album', $albumHydrator);
$hydrators->setService('Artist', $artistHydrator);

// Create the DelegatingHydrator and tell it to use our configured hydrator locator
$delegating = new Laminas\Hydrator\DelegatingHydrator($hydrators);

// Now we can use $delegating to hydrate or extract any supported object
$array  = $delegating->extract(new Artist());
$artist = $delegating->hydrate($data, new Artist());
```

### Laminas\\Hydrator\\ObjectProperty

Any data key matching a publicly accessible property will be hydrated; any public properties
will be used for extraction.

### Laminas\\Hydrator\\Reflection

Similar to the `ObjectProperty` hydrator, but uses [PHP's reflection API](http://php.net/manual/en/intro.reflection.php)
to hydrate or extract properties of any visibility. Any data key matching an
existing property will be hydrated; any existing properties will be used for
extraction.
