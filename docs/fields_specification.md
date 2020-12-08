Apache Avro has `Primitive Types`, `Complex Types` and `Logical Types`, so we need to match these types with python types.

## Primitive Types and python representation

The set of primitive type names is:

* null: no value
* boolean: a binary value
* int: 32-bit signed integer
* long: 64-bit signed integer
* float: single precision (32-bit) IEEE 754 floating-point number
* double: double precision (64-bit) IEEE 754 floating-point number
* bytes: sequence of 8-bit unsigned bytes
* string: unicode character sequence

So, the previous types can be matched to:

| Avro Type | Python Type |
|-----------|-------------|
| string    |     str     |
| long      |     int     |
| boolean   |     bool    |
| double    |     float   |
| null      |     None    |
| bytes     |     bytes   |

## Complex Types

Avro supports six kinds of complex types: enums, arrays, maps, fixed, unions and records.

| Avro Type | Python Type |
|-----------|-------------|
| enums     |   tuple     |
| arrays    |   list      |
| maps      |   dict      |
| fixed     |   types.Fixed|
| unions    |typing.Union |
| records   |Python Class |

* Enums: Use the type name "enum" and support the following attributes:
  1. name: a JSON string providing the name of the enum (required).
  2. namespace: a JSON string that qualifies the name;
  3. aliases: a JSON array of strings, providing alternate names for this enum (optional).
  4. doc: a JSON string providing documentation to the user of this schema (optional).
  5. symbols: a JSON array, listing symbols, as JSON strings (required). All symbols in an enum must be unique; duplicates are prohibited. Every symbol must match the regular expression [A-Za-z_][A-Za-z0-9_]* (the same requirement as for names).

When we want to define a `enum` type we should specify a default value because we need to define the `symbols`
In future version we will have a custom enum type to avoid this

* Arrays: Use the type name "array" and support the following attribute:
  1. name: a JSON string providing the name of the enum (required).
  2. items: the schema of the array's items.

* Maps: Use the type name "map". Map keys are assumed to be string. Support the following attribute:
  1. name: a JSON string providing the name of the enum (required).
  2. values: the schema of the map's values.

* Fixed uses the type name "fixed" and supports two attributes:
  1. name: a string naming this fixed (required).
  2. namespace, a string that qualifies the name;
  3. aliases: a JSON array of strings, providing alternate names for this enum (optional).
  4. size: an integer, specifying the number of bytes per value (required).

* Unions: `Unions` are represented using JSON arrays. For example, `["null", "string"]` declares a schema which may be either a null or string. 
Under the Avro specifications, if a union field as a default, the type of the default must be the first listed type in the array. 
Dataclasses-avroschema will automatically generate the appropriate array if a default is provided. Note that an optional field (typing.Optional[T]) generates the union `[T, null]`, where `T` is the first element in the union. 
`None` will need to be explicitly declared the default to generate the appropriate schema, if the default should be `None/null`. 

* Records: `Records` use the type name `record` and will represent the `Schema`.

### Logical Types

A logical type is an Avro primitive or complex type with extra attributes to represent a derived type. The attribute logicalType must always be present for a logical type, and is a string with the name of one of the logical types listed later in this section. Other attributes may be defined for particular logical types.

A logical type is always serialized using its underlying Avro type so that values are encoded in exactly the same way as the equivalent Avro type that does not have a logicalType attribute. Language implementations may choose to represent logical types with an appropriate native type, although this is not required.

Language implementations must ignore unknown logical types when reading, and should use the underlying Avro type. If a logical type is invalid, for example a decimal with scale greater than its precision, then implementations should ignore the logical type and use the underlying Avro type.

* Date: The date logical type represents a date within the calendar, with no reference to a particular time zone or time of day. A date logical type annotates an Avro int, where the int stores the number of days from the unix epoch, 1 January 1970 (ISO calendar).

* Time (millisecond precision): The time-millis logical type represents a time of day, with no reference to a particular calendar, time zone or date, with a precision of one millisecond. A time-millis logical type annotates an Avro int, where the int stores the number of milliseconds after midnight, 00:00:00.000.

* Timestamp (millisecond precision): The timestamp-millis logical type represents an instant on the global timeline, independent of a particular time zone or calendar, with a precision of one millisecond. A timestamp-millis logical type annotates an Avro long, where the long stores the number of milliseconds from the unix epoch, 1 January 1970 00:00:00.000 UTC.

* UUID: Represents a uuid as a string

* Decimal: Represents a decimal.Decimal as bytes

| Avro Type | Logical Type |Python Type |
|-----------|--------------|-------------|
| int       |  date        | datetime.date
| int       |  time-millis | datetime.time     |
| long      |  timestamp-millis | datetime.datetime |
| string    |  uuid        | uuid.uuid4 |
| string    |  uuid        | uuid.UUID |
| bytes     | decimal      | decimal.Decimal |

### Avro Field and Python Types Summary

Python Type | Avro Type   | Logical Type |
|-----------|-------------|--------------|
| str       | string      | do not apply |
| long      | int         | do not apply |
| bool      | boolean     | do not apply |
| double    | float       | do not apply |
| None      | null        | do not apply |
| bytes     | bytes       | do not apply |
| typing.List      | array       | do not apply |
| typing.Tuple     | array       | do not apply |
| typing.Sequence      | array       | do not apply |
| typing.MutableSequence      | array       | do not apply |
| typing.Dict      | map         | do not apply |
| typing.Mapping      | map         | do not apply |
| typing.MutableMapping      | map         | do not apply |
| types.Fixed      | fixed         | do not apply |
| types.Enum      | enum         | do not apply |
| typing.Union| union     | do not apply |
| typing.Optional| union (with `null`)    | do not apply |
| Python class | record  | do not apply |
| datetime.date | int     |  date        |
| datetime.time | int     |  time-millis |
| datetime.datetime| long  |  timestamp-millis |
| uuid.uuid4  | string    |  uuid        |
| decimal.Decimal | bytes | decimal      |

## Adding Custom Field-level Attributes

You may want to add field-level attributes which are not automatically populated according to the typing semantics
listed above. For example, you might want a `"doc"` attribute or even a custom attribute (which Avro supports as long
as it doesn't conflict with any field names in the core Avro specification). An example of a custom attribute is a flag
for whether a field contains sensitive data. e.g. `"sensitivty"`.

When your Python class is serialised to Avro, each field will contain a number of attributes. Some of these of are common
to all fields such as `"name"` and others are specific to the datatype (e.g. `array` will have the `items` attribute).
In order to add custom fields, you can use the `field` descriptor of the built-in `dataclasses` package and provide a
`dict` of key-value pairs to the `metadata` parameter as in `dataclasses.field(metadata={'doc': 'foo'})`.

### Examples

Adding a `doc` attribute to fields.

```python
from dataclasses import dataclass, field
from dataclasses_avroschema import AvroModel, types

@dataclass
class User(AvroModel):
    "An User"
    name: str = field(metadata={'doc': 'bar'})
    age: int = field(metadata={'doc': 'foo'})

User.avro_schema()

{
    "type": "record",
    "name": "User",
    "doc": "An User",
    "fields": [
        {"name": "name", "type": "string", "doc": "bar"},
        {"name": "age", "type": "long", "doc": "foo"}
    ]
}
```

Adding an additional `sensitivity` attribute to fields.

```python
from dataclasses import dataclass, field
from dataclasses_avroschema import AvroModel, types

@dataclass
class User(AvroModel):
    "An User"
    name: str = field(metadata={'doc': 'bar', 'sensitivity': 'HIGH'})
    age: int = field(metadata={'doc': 'foo', 'sensitivity': 'MEDIUM'})

User.avro_schema()

{
    "type": "record",
    "name": "User",
    "doc": "An User",
    "fields": [
        {"name": "name", "type": "string", "doc": "bar", "sensitivity": "HIGH"},
        {"name": "age", "type": "long", "doc": "foo", "sensitivity": "MEDIUM"}
    ]
}
```
