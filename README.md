## Easy XML Builder

Simple and lightweight library to make generating XML a breeze.

### Installation

Install via [Composer](https://getcomposer.org/).

```
$ composer require pdeans/xml-builder
```

### Usage

The XML builder library extends PHP's [XMLWriter](http://us3.php.net/manual/en/book.xmlwriter.php) extension. All [XMLWriter](http://us3.php.net/manual/en/example.xmlwriter-oop.php) object oriented API properties and methods are available for each XML builder instance.

First, instantiate a new XML builder class object:

```php
use pdeans\Builders\XmlBuilder;

$bulder = new XmlBuilder;
```

The `create` method is used to generate an xml tag. The `create` method takes the name of the root element as the first argument, and an associative array consisting of the data to build the root attribute elements and/or child elements as the second argument.

Here is a simple example:

```php
$xml = $builder->create('Category_Add', [
    '@tags' => [
        'Code' => 'Tools',
        'Name' => $builder->cdata('Class Tools and Skill Kits'),
    ],
]);
```

This will produce the following xml:

```xml
<Category_Add>
    <Code>Tools</Code>
    <Name><![CDATA[Class Tools and Skill Kits]]></Name>
</Category_Add>
```

#### Parent/Child Elements

Notice how the array key-values function under the `@tags` array from the above example. The keys represent the xml element names, and the values represent the xml element values. Child tags can also be nested following this pattern with the parent element represented by the array key, and the array value consisting of an array of the child elements as key-value pairs. This pattern can be repeated as needed to nest subsequent child elements.

#### Element Value Helpers

The `cdata` helper method can be used to wrap an element value in a `<![CDATA[]]>` tag, while the `decimal` helper method can be used to format a decimal number into a standard decimal format, rounding to 2 decimals by default and stripping out commas. The `decimal` helper method accepts an optional second parameter to set the precision.

```php
// Output: <![CDATA[Class Tools and Skill Kits]]>
echo $builder->cdata('Class Tools and Skill Kits');

// Output: 49.00
echo $builder->decimal(49.0000000);

// Output: 49.001
echo $builder->decimal(49.0005, 3);
```

#### Reserved Keys

The `@tags` key represents one of 3 reserved keys (each containing shortcut key counterparts) that the xml builder uses to parse and generate the xml. The reserved keys are as follows:

**@attributes Key**  
_Shortcut: **@a**_

The `@attributes` key is used to create xml element attributes. The `@a` key is also supported as a shortcut for the `@attributes` key.

Examples:

```php
$xml = $builder->create('CategoryProduct_Assign', [
    '@attributes' => [
        'category_code' => 'Food',
        'product_code'  => 'ale-gallon',
    ],
]);

$xml = $builder->create('CategoryProduct_Assign', [
    '@a' => [
        'category_code' => 'Food',
        'product_code'  => 'ale-gallon',
    ],
]);
```

XML Produced:

```xml
<CategoryProduct_Assign category_code="Food" product_code="ale-gallon"/>
```

**@tags Key**  
_Shortcut: **@t**_

The `@tags` key accepts an associative array of data to build the root element's children. The `@t` key is also supported as a shortcut for the `@tags` key.

Examples:

```php
$xml = $builder->create('ProductAttribute_Add', [
    '@a' => [
        'product_code' => 'chest',
    ],
    '@tags' => [
        'Code'   => 'lock',
        'Type'   => 'select',
        'Prompt' => $builder->cdata('Lock'),
    ],
]);

$xml = $builder->create('ProductAttribute_Add', [
    '@a' => [
        'product_code' => 'chest',
    ],
    '@t' => [
        'Code'   => 'lock',
        'Type'   => 'select',
        'Prompt' => $builder->cdata('Lock'),
    ],
]);
```

XML Produced:

```xml
<ProductAttribute_Add product_code="chest">
    <Code>lock</Code>
    <Type>select</Type>
    <Prompt><![CDATA[Lock]]></Prompt>
</ProductAttribute_Add>
```

**@value Key**  
_Shortcut: **@v**_

The `@value` key explicitly sets an xml element value. Generally, this is only required on xml elements that require both attributes and a value to be set. The `@v` key is also supported as a shortcut for the `@value` key.

Examples:

```php
$xml = $builder->create('Module', [
    '@attributes' => [
        'code' => 'customfields',
        'feature' => 'fields_prod',
    ],
    '@tags' => [
        'ProductField_Value' => [
            '@attributes' => [
                'product' => 'chest',
                'field' => 'armor_type',
            ],
            '@value' => 'wood',
        ],
    ],
]);

$xml = $builder->create('Module', [
    '@a' => [
        'code' => 'customfields',
        'feature' => 'fields_prod',
    ],
    '@t' => [
        'ProductField_Value' => [
            '@a' => [
                'product' => 'chest',
                'field' => 'armor_type',
            ],
            '@v' => 'wood',
        ],
    ],
]);
```

XML Produced:

```xml
<Module code="customfields" feature="fields_prod">
    <ProductField_Value product="chest" field="armor_type">wood</ProductField_Value>
</Module>
```

Note that the `@tags` key is used on the first level only of the associative array of tag data, as it represents the child tag data, while the other two reserved keys can be used on any sub-level throughout the associative array.

#### Repeated Tags

Sometimes repeated tags are used in xml, which does not play nice with associative array key-value pairs. To circumvent this, the element name is still passed as the array key, however, the array value consists of a sequential array of arrays with the tag data.

```php
$xml = $builder->create('Order_Add', [
    '@t' => [
        'Charges' => [
            'Charge' => [
                [
                    'Type' => 'SHIPPING',
                    'Description' => 'Shipping: UPS Ground',
                    'Amount' => 5.95
                ],
                [
                    'Type' => 'TAX',
                    'Description' => 'Sales Tax',
                    'Amount' => 2.15
                ],
            ],
        ],
    ],
]);
```

XML Produced:

```xml
<Order_Add>
    <Charges>
        <Charge>
            <Type>SHIPPING</Type>
            <Description>Shipping: UPS Ground</Description>
            <Amount>5.95</Amount>
        </Charge>
        <Charge>
            <Type>TAX</Type>
            <Description>Sales Tax</Description>
            <Amount>2.15</Amount>
        </Charge>
    </Charges>
</Order_Add>
```

#### Self-closing Tags

To generate a self-closing element without attributes, pass a value of *null* as the array value.

```php
$xml = $builder->create('Order_Add', [
    '@t' => [
        'TriggerFulfillmentModules' => null,
    ],
]);
```

XML Produced:

```xml
<Order_Add>
    <TriggerFulfillmentModules />
</Order_Add>
```