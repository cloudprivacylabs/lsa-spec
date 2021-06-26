# Layered Schemas

## Introduction

A schema describes the shape of data. An *instance* of a schema is a
data object that conforms to a schema. In general, a schema includes
structural constraints about its instances, not semantic
information. A structurally valid data object can be semantically
inconsistent. Layered schemas define data objects using open-ended
semantic information. This additional information can be used to
validate data objects structurally and allows writing programs that
deal with semantics without hardwiring meaning into algorithms. For
example, a program that removes personally identifiable information
can operate on many types of data by selecting data elements based on
the privacy attributes assigned by the schema, instead of hardcoding
certain fields.

A layered schema has a schema base and layers (overlays) that modify
the information of the base. These layers can be used to add or modify
constraints, semantic tags, processing directives, and other metadata
based on the use case, locale, or implementation. 

A layered schema defines data objects as a labeled property graph. The
same schema can be used to validate a structured object (such as a
JSON or an XML document) and translate it into a labeled propery
graph. The translation process itself can be written as a program that
reads the semantic information embedded into the schema, so it is not
necessarily linked to a particular data format. 

[Syntax and Data Model](syntax-and-model.md)

[Algorithms](schema-algorithms.md)


