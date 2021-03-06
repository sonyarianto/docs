---
layout: default
description: How to enforce attributes and data types for documents using JSON Schema on collection level.
title: Schema Validation for Documents
redirect_from:
  - /3.7/document-schema-validation.html # 3.7 -> 3.7
---
Schema Validation
=================

<small>Introduced in: v3.7.0</small>

While ArangoDB is schema-less, it allows to enforce certain document structures
on collection level. The desired structure can be described in the popular
[JSON Schema](https://json-schema.org/){:target="_blank"} format (draft-4,
without remote schema support). The level of validation and a custom error
message can be configured.

Also see [v3.7 Known Issues](release-notes-known-issues37.html#other)

Enable validation for a collection
----------------------------------

Schema validation can be managed via the JavaScript API, typically
using _arangosh_, as well as via the HTTP interface.

To enable schema validation either pass the `validation` property on collection
creation or update the properties of an existing collection. It expects an
object with the following attributes: `rule`, `level` and `message`.

- The `rule` attribute must contain the JSON Schema description.
- `level` controls when the validation will be applied.
- `message` sets the message that will be used when validation fails.

```js
var validation =
{ rule: { properties: { nums: { type: "array"
                              , items: { type: "number", maximum: 6 }
                              }
                      }
        , additionalProperties: { type: "string" }
        , required: ["nums"]
        }
, level: "moderate"
, message: "The document does not contain an array of numbers in attribute 'nums' or one of the numbers is bigger than 6."
}

// Create new collection with validation
db.create("new_collection", { "validation": validation })

// Update validation of existing collection
db.existing_collection.properties({ "validation": validation })
```

JSON Schema Rule
----------------

The `rule` must be a valid JSON Schema object as outlined in the
[specification](https://json-schema.org/specification.html){:target="_blank"}.
See [Understanding JSON Schema](https://json-schema.org/understanding-json-schema/reference/object.html){:target="_blank"}
for a user guide on how to write JSON Schema descriptions.

{% hint 'security' %}
Remote schemas are not supported for security reasons.
{% endhint %}

Levels
------

The level controls when the validation is triggered:

- `none`: The rule is inactive and validation thus turned off.
- `new`: Only newly inserted documents are validated.
- `moderate`: New and modified documents must pass validation, except for
  modified documents where the OLD value did not pass validation already.
  This level is useful if you have documents which do not match your target
  structure, but you want to stop the insertion of more invalid documents
  and prohibit that valid documents are changed to invalid documents.
- `strict`: All new and modified document must strictly pass validation.
  No exceptions are made (default).
