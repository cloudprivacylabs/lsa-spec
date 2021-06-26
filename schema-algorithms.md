# Layered Schemas - Algorithms

## Composition 

Composition operation combines layers to create a new variant of a
schema, or to create a new overlay that is a combination of several
overlays. When composing layers, an overlay can be added into a schema
or another overlay. A schema cannot be composed with another schema.

When composing schema layers, all layers must agree on `@type`. That
means, the `@type`s of layers excluding the attribute types must have
nonempty intersections. A layer that has only attribute types can be
composed with any other layer.

The following are valid compositions:

 * `Schema(object) + Overlay_1 + Overlay_2(object)`
The result is a schema for `object`.
 * `Overlay_1(object) + Overlay_2(Object)`
The result is an overlay for `object`.
 * `Schema(object) + Overlay_1 + Overlay_2(object)`
The result is a schema for object. The first overlay can be composed with the schema because it is not declared for a particular object type. 
   
### Composing Terms
    
A key part of the algorithm is composing the values of terms included
in the attribute. It should be possible for an implementation to
define terms that specify term-specific composition methods. The term
composition methods are as follows:
 
  * Set composition: This is the default term composition method. The
    set composition of two terms is the set union of their values. For
    example:

| Value 1 | Value 2 | Composition |
| ------- | ------- | ----------- |
| A       | [A, B]  | [A, B]      |
| A       | B       | [A, B]      |
| A       | [B, C]  | [A, B, C]   |

  * List composition: The list composition of two terms is the
    concatenation of the values of the second term to the first. For
    example:
    
| Value 1 | Value 2 | Composition |
| ------- | ------- | ----------- |
| A       | [A, B]  | [A, A, B]   |
| A       | B       | [A, B]      |
| A       | [B, C]  | [A, B, C]   |

  * Override composition: The value of the second term overrides the
    first. For example:
    
| Value 1 | Value 2 | Composition |
| ------- | ------- | ----------- |
| A       | [A, B]  | [A, B]      |
| A       | B       | [B]         |
| A       | [B, C]  | [B, C]      |

   * No composition: The value of the first term remains. For example:
   
| Value 1 | Value 2 | Composition |
| ------- | ------- | ----------- |
| A       | [A, B]  | [A]         |
| A       | B       | [A]         |
| A       | [B, C]  | [A]         |
      

### Algorithm

This algorithm composes the `source` layer into `target` layer. The
result is the `target` layer. The algorithm recursively processes the
source attributes, find the matching target attribute and composes the
two.

For a given `source` attribute `sourceAttr`, the `path(sourceAttr)`
refers to the sequence of attribute id's from `sourceAttr` to the
layer root. For example:
```
{
  "@id": "a",
  "@type": "Object",
  "attributes": {
     "b": {
       "@type":"Object",
       "attributes": {
         "c": {}
       }
     }
  }
}
```
Above, `path(a) = a`, `path(b) = `a.b`, and `path(c) = a.b.c`.

In the below algorithm, an overlay node `o` matches the base layer
node `b` if `path(o)` is a suffix of `path(b)`. 


```
ComposeNode(target,source)
  ComposeTerms(target,source)
  For each source attribute node s
    Find target node t such that path(t) has path(s) as a suffix
    ComposeNode(t,s)

  Add all source non-attributes nodes emanating from s into t
```

This algorithm allows defining overlays that contains only the leaf
nodes without the intermediate steps. For example:
```
{
  "@type": "Schema",
  "layer": {
    "@type": "Object",
    "attributes": {
      "obj": {
        "@type": "Object",
        "attributes": {
           "nestedAttr": {
              "@type": "Value"
           }
        }
    }
  }
}

{
  "@type": "Overlay",
  "layer": {
    "@type": "Object",
    "attributes": {
      "nestedAttr": {
        "@type":"Value",
        "descr": "description"
      }
    }
  }
}
```

The `nestedAttr` in the overlay has path `nestedAttr`, which matches
`obj.nestedAttr`, so the composition becomes:

```
{
  "@type": "Schema",
  "layer": {
    "@type": "Object",
    "attributes": {
      "obj": {
        "@type": "Object",
        "attributes": {
           "nestedAttr": {
              "@type": "Value",
              "descr": "description"
           }
        }
      }
    }
  }
}
```

## Slicing

Slicing operation creates new layers from an existing layer by
selecting a subset of the terms. It uses an `accept` operation that
selects the terms that will be included in the output.

### Algorithm

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
