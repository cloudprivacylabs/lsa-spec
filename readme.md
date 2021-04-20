# Layered Schemas

## Schemas and Overlays

A layer can either be a `Schema` or an `Overlay`. A layer has the
following structure:

```
{
  "@context": "http://layeredschemas.org/v1.0/ls.jsonld",
  "@type": "Schema", or "Overlay",
  "@id": "http://layeredschemas.org/exampleSchema",
  "objectType": "http://example.org/SomeObject",
  "attributes": [
    {
      "@id": "attribute id",
      "@type": "Value",
      .. // other annotations
    },
    ...
  ]
}
```
Or:

```
{
  "@context": "http://layeredschemas.org/v1.0/ls.jsonld",
  "@type": "Schema", or "Overlay",
  "@id": "http://layeredschemas.org/exampleSchema",
  "objectType": "http://example.org/SomeObject",
  "attributeList": [
    {
      "@id": "attribute id",
      "@type": "Value",
      .. // other annotations
    },
    ...
  ]
}
```

### `@context`

The layered schemas context is used to expand the schema. This
context defines the expanded terms for the schema attributes and
schema structure. If the schema includes semantic annotations not
defined in this context, additional contexts can be specified as an
array.

### `@type`

This is a `Schema` object. Valid values are:

 * `Schema` or `http://layeredschemas.org/Schema`
 * `Overlay` or `http://layeredschemas.org/Overlay`
 
 Note that this notation declares the JSON-LD node type as a schema or
 overlay.

### `@id`

The schema identifier. The id is usage specific, it can be a
globally unique identifier, or an identifier meaningful for the
domain schema is used in. This identifier can be used to define
references to the schema, and should be unique for the domain.
  
Note that this notation declares the JSON-LD node id as the schema
identifier.
  
### `objectType` 

`@id: http://layeredschemas.org/Layer/objectType`<br>
`@type: string`

The object type defined by the schema, or the overlay is for. This
object type can be used to reference to this schema from other
schemas.
  
An `objectType` is mandatory for a schema. It is optional for an
overlay. An overlay without an `objectType` can be composed with any
other layer.

### attributes

`@id: http://layeredschemas.org/Object/attributes`<br>
`@container: @id`<br>

The attributes of the object defined by this schema. It is an idmap,
so attributes can be defined as an array of attributes as a JSON
object. That is, both of the following definitions are valid:

```
"attributes": [
  { 
    "@id": "attr1"
  },
  { 
    "@id": "attr2" 
  },
  ...
]
```

```
"attributes": {
  "attr1": {},
  "attr2": {},
  ...
}
```

The order of attributes are not significant, and not necessarily preserved.

### attributeList

`@id: http://layeredschemas.org/Object/attributeList`<br>
`@container: @list`<br>

The attributes of the object defined by this schema. A schema can either have `attributes` or `attributeList`.
`attributeList` is a list container, meaning the ordering of attributes are significant:

```
"attributeList": [
  { 
    "@id": "attr1"
  },
  { 
    "@id": "attr2" 
  },
  ...
]
```

Layered schema support the following types of attributes:

#### Value

`@type: Value` or `@type: http://layeredschemas.org/Value`

A sequence of bytes. Examples are `string`, `int`, `xs:Date`,
etc. The schema may specify the encoding, format, and architecture
or representation specific type.
  
``` 
{ 
  "@id": "attribute1",
  "@type": "Value" 
}
```

#### Object

`@type: Object` or `@type: http://layeredschemas.org/Object`

Key-attribute pairs, similar to a JSON object or an XML element. An
`Object` attribute has `attributes` that list the nested attributes.

```
{
  "@id": "object1",
  "@type": "Object",
  "attributes": {
     "nestedAttr1": {
       "@type": "Value"
     },
     ...
  }
}
```

#### Array

`@type: Array` or `@type: http://layeredschemas.org/Object`

An ordered list of attributes. An `Array` attribute has `items` that
specifies the structure of array items. The following example is a
value array.

```
{
  "@id": "array1",
  "@type": "Array",
  "items": {
    "@id": "arrayElementsID",
    "@type": "Value"
  }
}
```
Below is an object array:
```
{
  "@id": "<attributeId>",
  "@type": "Array",
  "items": {
    "@type": "Object",
    "attributes": {
      ...
    }
  }
}
```

#### Reference

`@type: Reference` or `@type: http://layeredschemas.org/Reference`

A reference to another object. How this reference is resolved is
implementation dependent. It can be a [strong reference](#strong-reference) that selects a particular schema
variant, or a [weak reference](#weak-reference) that will be resolved at run time
based on the existing context.  The reference can be:

  * a hash value or some other DRI of a schema,
  * an IRI specifying an object, or a particular version of that object,
  * an IRI specifying a schema,
  * or some other identifier that can be resolved to a schema.
  
  
When compiled, the resulting schema will have all `Reference`
attributes replaced with the actual referenced schema.
  
```
{
  "@id": "weakref",
  "@type": "Reference",
  "reference": "http://example.org/ExampleObject"
}
```

This is a weak reference to an `ExampleObject` that will be resolved
based on the current processing context.

```
{
  "@id": "strongref",
  "@type": "Reference",
  "reference": "sha256://748736a7fde293...."
}
```
This is a strong reference that directly addresses a schema variant using its hash.

#### Composite

`@type: Composite` or `@type: http://layeredschemas.org/Composite`

A composition of multiple attributes. The result of a composition is
an `Object`, so the elements of a composition are limited to `Value`,
`Object`, and `Reference` types.

```
{
  "@id": "compositeAttr",
  "@type": "Composite",
  "allOf": [
     {
       "@id":"part1",
       "@type": "Reference",
       "reference": "http://example.org/SomeObject"
     },
     {
       "@id": "part2",
       "@type": "Object",
       "attributes": {
          "attr1": {
             "@type": "Value"
          },
          "attr2": {
             "@type": "Value
          }
        }
     },
     {
       "@id": "part3",
       "@type": "Value"
     }
  ]
}
```

When compiled, all `Composite` attributes are replaced with `Object`
attributes containing all the attributes of the composite attribute. In
the above example, the `compositeAttr` will be an `Object` containing
all attributes of `SomeObject`, `attr1`, `attr2`, and `part3`.

#### Polymorphic

`@type: Polymorphic` or `@type: http://layeredschemas.org/Polymorphic`

A polymorphic type that can be one of the types listed in the
attribute definition. This type of attribute requires constraint
annotations to decide the actual type of the object at
run time.
  
```
{
  "@id": "polyAttr",
  "@type": "Polymorphic",
  "oneOf": [
     {
       "@type": "Reference",
       "reference": "sha256://2566efadfe9843..."
     },
     {
       "@type": "Reference",
       "reference": "sha256://874658d0a9e8f.."
     }
  ]
}
```

This attribute can be one of two types defined by strong
references. When data object is ingested using this schema, the
`polyAttr` attribute will be checked if any one of the options
validate the input data. If one of them validates, then the type is
decided and the selected schema will be used to process the
attribute. If none matches, or more than one option matches, an error
will be raised.

### Expanded Model

Layered schema implementations must work with the following expanded
JSON-LD representation. Below is the model for `Schema`. `Overlay`
uses the same model with the corresponding `@type`:

```
[
  {
    "@type": [
      "http://layeredschemas.org/v1.0/Schema"
    ],
    "http://layeredschemas.org/v1.0/Layer/objectType": [
      {
        "@value": "objType"
      }
    ],
    "http://layeredschemas.org/v1.0/Layer/objectVersion": [
      {
        "@value": "objVersion"
      }
    ],
    "http://layeredschemas.org/v1.0/Object/attributes": [
      {
        "@id": "idValue",
        "@type": [
          "http://layeredschemas.org/v1.0/Value"
        ]
      },
      {
        "@id": "idObject",
        "@type": [
          "http://layeredschemas.org/v1.0/Object"
        ],
        "http://layeredschemas.org/v1.0/Object/attributes": []
      },
      {
        "@id": "idReference",
        "@type": [
          "http://layeredschemas.org/v1.0/Reference"
        ],
        "http://layeredschemas.org/v1.0/Reference/reference": [
          {
            "@id": "http://example.org/reference"
          }
        ]
      },
      {
        "@id": "idArray",
        "@type": [
          "http://layeredschemas.org/v1.0/Array"
        ],
        "http://layeredschemas.org/v1.0/Array/items": [
          {}
        ]
      },
      {
        "@id": "idComposite",
        "@type": [
          "http://layeredschemas.org/v1.0/Composite"
        ],
        "http://layeredschemas.org/v1.0/Composite/allOf": [
          {
            "@list": [
              {}
            ]
          }
        ]
      },
      {
        "@id": "idPolymorphic",
        "@type": [
          "Polymorphic"
        ],
        "http://layeredschemas.org/v1.0/Polymorphic/oneOf": [
          {
            "@list": [
              {}
            ]
          }
        ]
      }
    ]
  }
]
```

## Schema Manifest


A `SchemaManifest` object defines how a particular variant of a data
object is composed from its layers.

```
{
  "@context": "https://layeredschemas.org/v1.0/ls.jsonld",
  "@type": "SchemaManifest",
  "@id": "<unique identifier for the schema defined by this manifest>",
  "publishedAt": "<schema publish date>",
  "objectType": "<the object type defined by this schema>",
  "objectVersion": "<the version of the object type>",
  "bundle": "<the bundle containing all valid layers and references>",
  "schema": "<reference to the schema">,
  "overlays": [
     "reference to overlay1",
     "reference to overlay2",
     ...
   ]
}
```

### @type

The type of this JSON-LD object is a
`http://layeredschemas.org/SchemaManifest`.

### @id

The unique ID for this schema variant.

###  publishedAt

`@type: http://schema.org/Date`<br>
`@id: http://layeredschemas.org/v1.0/SchemaManifest/publishedAt`

The schema variant publish date.

### objectType

`@type: @id`<br>
`@id: http://layeredschemas.org/v1.0/Layer/objectType`

The type of the data object defined by this schema. There can be many
schemas defining different variants of the same data object. 

Note that this term is identical to the `objectType` term used in layers.

### objectVersion

`@type: string`<br>
`@id: http://layeredschemas.org/v1.0/Layer/objectVersion`

The version of the data object defined by this schema variant.

Note that this term is identical to the `objectVersion` term used in layers.

### bundle

`@type: @id`<br>
`@id: http://layeredschemas.org/v1.0/SchemaManifest/bundle`

Points to a [schema bundle](#bundle) containing 
[strong references](#strong-reference)
for layers and schemas to resolve all [weak references](#weak-reference).

### schema

`@type: @id`<br>
`@id: http://layeredschemas.org/v1.0/SchemaManifest/schema`


Points to the schema. The schema must be of type `Schema` and declare
the same object type as the schema manifest.

### overlays

`@type: @id`<br>
`@container: @list` <br>
`@id: http://layeredschemas.org/v1.0/SchemaManifest/overlays`

An ordered list of layers composing the schema. Each layer must be of
type `Overlay`, and declare the same `objectType` as the schema manifest.

### Expanded Model

Layered schema implementations must work with the following expanded
JSON-LD representation. 

```[
  {
    "@type": [
      "http://layeredschemas.org/v1.0/SchemaManifest"
    ],
    "http://layeredschemas.org/v1.0/Layer/objectType": [
      {
        "@value": "objType"
      }
    ],
    "http://layeredschemas.org/v1.0/Layer/objectVersion": [
      {
        "@value": "objVersion"
      }
    ],
    "http://layeredschemas.org/v1.0/SchemaManifest/publishedAt": [
      {
        "@value": "20210320T00:00:00Z"
      }
    ],
    "http://layeredschemas.org/v1.0/SchemaManifest/bundle": [
      {
        "@id": "sha256://27634767a887e887f8e87..."
      }
    ],
    "http://layeredschemas.org/v1.0/SchemaManifest/schema": [
      {
        "@id": "sha256://5878a8e8faa9890..."
      }
    ],
    "http://layeredschemas.org/v1.0/SchemaManifest/overlays": [
      {
        "@list": [
          {
            "@id": "sha256://587376a7676f767..."
          },
          {
            "@id": "sha256://ee538866547676f767..."
          }
        ]
      }
    ]
  }
]

```

## Bundle

A schema `Bundle` links a group of schema layers together so the
references to the layers in a schema manifest and references to
objects in schemas can be resolved unambiguously. In other words, a
`Bundle` is a mechanism to convert [weak
references](#weak-reference) into [strong
references](#strong-reference).

```
{
  "@context": "https://layeredschemas.org/v1.0/ls.jsonld",
  "@type": "Bundle",
  "@id": "<unique identifier for the schema bundle>",
  "references": {
     "weak-reference": "strong-reference",
     "weak-reference": [
          "strong-reference",
          "strong-reference",
          ...
       ]
     }
  }
}
```

### @type

The type of this JSON-LD object is a
`http://layeredschemas.org/Bundle`.

### @id

The unique ID for this schema bundle.

### references

`@id: http://layeredschemas.org/Bundle/references`</br>
`@container: @id`

Specifies one or more [strong references](#strong-reference) for each [weak
reference](#weak-reference).

## Slicing


Slicing operation creates new layers from an existing layer by
selecting a subset of the terms. It uses an `accept` operation that
selects the terms that will be included in the output.

```
SliceAttribute(attr,accept)
  newAttribute:= new Attribute
  For each (term, value) in attr
    If accept(term)
      newAttribue[term]=value
  
  If attr is one of Object, Array, Composite, Polymorphic
    For each nestedComponent under attr
      SliceAttribute(nestedComponent,accept)
      
  If newAttribute is not empty, return newAttribute
```

### Example

Consider the following layer:
```
"attributes": {
  "attr1": {
     "@type": "Value",
     "format": "url",
     "privacyClassifications": ["PII"]
  },
  "attr2": {
    "@type": "Object",
    "attributes": {
       "attr3": {
         "@type": "Value",
         "privacyClassifications": ["BIT"]
       }
    }
  }
}
```

Slicing this schema with an `accept` function that only accepts
`attributes`, `items`, `allOf`, `oneOf`, and `reference`:

```
"attributes": {
  "attr1": {
     "@type": "Value",
  },
  "attr2": {
    "@type": "Object",
    "attributes": {
       "attr3": {
         "@type": "Value"
       }
    }
  }
}
```

Slicing this schema with an `accept` function that accepts `format`:
```
"attributes": {
  "attr1": {
     "@type": "Value",
     "format": "url",
  }
}
```

Slicing this schema with an `accept` function that accepts `privacyClassifications`:
```
"attributes": {
  "attr1": {
     "@type": "Value",
     "privacyClassifications": ["PII"]
  },
  "attr2": {
    "@type": "Object",
    "attributes": {
       "attr3": {
         "@type": "Value",
         "privacyClassifications": ["BIT"]
       }
    }
  }
}
```
