# Jsonable Behavior

A CakePHP behavior to automatically store nested data as JSON string and return the array on read again.
- Data can be of type array, params or list - or kept in JSON format
- Additional sanitize functionality with "clean", "sort" and "unique

## Configs
- 'fields' => array(), // Fields to convert
- 'input' => 'array', // json, array, param, list (param/list only works with specific fields)
- 'output' => 'array', // json, array, param, list (param/list only works with specific fields)
- 'separator' => '|', // only for param or list
- 'keyValueSeparator' => ':', // only for param
- 'leftBound' => '{', // only for list
- 'rightBound' => '}', // only for list
- 'clean' => true, // only for param or list (autoclean values on insert)
- 'sort' => false, // only for list
- 'unique' => true, // only for list (autoclean values on insert),
- 'map' => array(), // map on a different DB field
- 'encodeParams' // params for json_encode
- 'decodeParams' // params for json_decode

## Usage
Attach it to your models in `initialze()` like so:
```php
$this->addBehavior('Tools.Jsonable', $config);
```
In my first scenario where I used it, I had a geocoder behavior attached to the model which returned an array.
I wanted to save all the returned values, though, for debugging purposes in a field "debug".
By using the following snippet I was able to do exactly that with a single line of config.
The rest is CakePHP automagic :)

```php
$this->addBehavior('Tools.Jsonable',
	array('fields' => array('debug'), 'map' => array('geocoder_result'));
```
I could access the array in the view as any other array since the behavior re-translates it back into an array on find().

Note: The mapping option is useful if you want to rename certain fields.
In my case the geocoder puts its data into the field "geocoder_result".
I might need to access this array later on in the model. So I "jsonable" it in the "debug" field for DB input
and leave the source field untouched for any later usage.
The same goes for the output: It will map the JSON content of "debug" back to the field "geocoder_result" as array, so
I have both types available then.

## Examples

### Params
What if needed something more frontend suitable.
I want to be able to use a textarea field where I can put all kinds of params
which will then also be available as array afterwards (as long as you are not in edit mode, of course).

We can switch to param style here globally for the entity:

```php
$this->addBehavior('Tools.Jsonable',
	array('fields' => 'details', 'input' => 'param', 'output' => 'array'));
```

Only for the add/edit action we need to also make "output" "param" at runtime:
```php
$this->Table->behaviors()->Jsonable->options(
	array('fields' => 'details', 'input' => 'param', 'output' => 'param'));
```

The form contains a "details" textarea field. We can insert:
```php
param1:value1|param2:value2
```

In our views we get our data now as array:
```php
debug($entity->get('details'));
// Prints:
// array('param1' => 'value1', 'param2' => 'value2')));
```


### Enums
we can also simulate an ENUM by using
```php
$this->addBehavior('Tools.Jsonable',
	array('fields' => 'tags', 'sort' => true, 'input' => 'list', 'output' => 'array'));
```
Dont' forget to use `'output' => 'list'` for add/edit actions.

In our textarea we can now type:
```
dog, cat, cat, fish
```

In our views we would result in:
```php
debug($entity->get('tags'));
// Prints:
// array('cat', 'dog', 'fish');
```

Note: The cleanup automation you can additionally turn on/off. There are more things to explore. Dig into the source code for that.

Yes � you could make a new table/relation for this in the first place.
But sometimes it�s just quicker to create such an enumeration field.

Bear in mind: It then cannot be sorted/searched by those values, though.
For a more static solution take a look at my [Static Enums](http://www.dereuromark.de/2010/06/24/static-enums-or-semihardcoded-attributes/).