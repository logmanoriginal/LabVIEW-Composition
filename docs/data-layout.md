This document describes the type and data strings produced by the `Variant To Flattened String` function.

# Type String

The type string defines the type and structure of the data type. It is an array of words (u16) that consists of these fundamental fields:

| Field | Description |
| ----- | ----------- |
| `length` | The total number of bytes (including the length field) for the current data type in the type string. The minimum length for any type is `4` (length + type descriptor). |
| `type descriptor` | One of the supported [type descriptors](https://labviewwiki.org/wiki/Type_descriptor). The high-order byte is reserved by LabVIEW and should be ignored.  |
| `type info` | (optional) Additional type information depending on the data type. |
| `label` | (optional) The label of the element as a Pascal string. |


`label` is padded to a multiple of two bytes.

> [!INFO]
> `length` is represented as `[nn]` in the tables below.

> [!INFO]
> A Pascal string consists of a 1-Byte length field followed by the string. The string content is therefore limited to 255 bytes (256 bytes total).

# Data String

The data string contains the value of the defined data type. Its composition entirely depends on the type descriptor.

# Numeric Types

## Integers

| Data Type | Type String | Data String |
| --------- | ----------- | ----------- |
| I8        | `[nn] xx01` | `00` |
| I16       | `[nn] xx02` | `0000` |
| I32       | `[nn] xx03` | `0000 0000` |
| I64       | `[nn] xx04` | `0000 0000 0000 0000` |
| U8        | `[nn] xx05` | `00` |
| U16       | `[nn] xx06` | `0000` |
| U32       | `[nn] xx07` | `0000 0000` |
| U64       | `[nn] xx08` | `0000 0000 0000 0000` |

## Floating-Point Numbers

| Data Type | Type String | Data String |
| --------- | ----------- | ----------- |
| SGL       | `[nn] xx09` | `0000 0000` |
| DBL       | `[nn] xx0A` | `0000 0000 0000 0000` |
| EXT       | `[nn] xx0B` | `0000 0000 0000 0000 0000 0000 0000 0000` |

## Complex Floating-Point Numbers

| Data Type | Type String | Data String |
| --------- | ----------- | ----------- |
| CSG       | `[nn] xx0C` | `0000 0000 0000 0000` |
| CDB       | `[nn] xx0D` | `0000 0000 0000 0000 0000 0000 0000 0000` |
| CXT       | `[nn] xx0E` | `0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000` |

## Enumerations

Enumerations have additional information in their type string, which is the number of values `[i]` and the individual string values as Pascal strings `[pstr]`. The block of string values is padded as a whole to a multiple of two bytes, not each string individually.

The data string contains the enum value.

| Data Type | Type String | Data String |
| --------- | ----------- | ----------- |
| U8        | `[nn] xx15 [i][pstr]...` | `00` |
| U16       | `[nn] xx16 [i][pstr]...` | `0000` |
| U32       | `[nn] xx17 [i][pstr]...` | `0000 0000` |

## Physical Numbers

Physical numbers have additional units and exponents on top of a floating-point or complex floating-point number. The type string indicates the number of units/exponents `[i]` followed by the individual units/exponents as two words `[unit, exponent]`.

**Example (excluding type prefix)**

In the following example, the physical number is defined as `m/s`.

```
00 02 00 03 00 01 00 02 FF FF
└─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘
  │     │     │     │     └─ exponent: -1
  │     │     │     └─────── unit: s
  │     │     └───────────── exponent: 1
  │     └─────────────────── unit: m
  └───────────────────────── number of units/exponents: 2
```

The data string carries the numerical value.

| Data Type | Type String | Data String |
| --------- | ----------- | ----------- |
| SGL       | `[nn] xx19 [i][unit, exponent]...` | `0000 0000` |
| DBL       | `[nn] xx1A [i][unit, exponent]...` | `0000 0000 0000 0000` |
| EXT       | `[nn] xx1B [i][unit, exponent]...` | `0000 0000 0000 0000 0000 0000 0000 0000` |
| CSG       | `[nn] xx1C [i][unit, exponent]...` | `0000 0000 0000 0000` |
| CDB       | `[nn] xx1D [i][unit, exponent]...` | `0000 0000 0000 0000 0000 0000 0000 0000` |
| CXT       | `[nn] xx1E [i][unit, exponent]...` | `0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000` |

### Physical Units

| Unit | Code |
| ---- | ---- |
| `rad` | 0 |
| `sr` | 1 |
| `s` | 2 |
| `m` | 3 |
| `kg` | 4 |
| `A` | 5 |
| `K` | 6 |
| `mol` | 7 |
| `cd` | 8 |

# Boolean

There exist two types of Boolean values in LabVIEW: 8-Bit and 16-Bit. The 16-Bit variant exists for historic reasons and is no longer in use. A False value is represented as `00` and True as `01`.

| Data Type | Type String | Data String |
| --------- | ----------- | ----------- |
| Bool (Legacy) | `[nn] xx20` | `0000` |
| Bool | `[nn] xx21` | `00` |

# String / Picture

For Strings and Pictures, the length information in the type info is always `FFFF FFFF` (variable size). The length of the actual string/picture is embedded in the data string as a 32-Bit length field.

| Data Type | Type String | Data String |
| --------- | ----------- | ----------- |
| String | `[nn] xx30 FFFF FFFF` | `[length][content]` |
| Picture | `[nn] xx33 FFFF FFFF` | `[length][content]` |

# Path

The type string of a path is exactly the same as strings and pictures.

Path data strings contain a literal 4-Byte prefix `PTH0` that serves as an identifier. It is followed by a 32-Bit length field (covers everything after the length field), a 16-Bit path type (see table of path types below), a 16-Bit component count, and then one Pascal string per component.

| Data Type | Type String | Data String |
| --------- | ----------- | ----------- |
| Path | `[nn] xx32 FFFF FFFF` | `PTH0 [length] [path-type] [component-count] [component-pstr]...` |

## Path Types

| Path Type | Code |
| --------- | ---- |
| Absolute | `0000` |
| Relative | `0001` |
| Not-A-Path | `0002` |
| UNC | `0003` |

# Array

The type string for arrays begins with the number of dimensions `[dims]`, followed by `FFFF FFFF` for every dimension to indicate variable dimension sizes (the actual size is in the data string). After the size information comes the type information of the embedded data type - which is a recursive type string. The data string begins with the size of the dimensions followed by the data as one contiguous block.

**Example**

The following type string is of a 1-dimensional U8 array.

```
00 0E xx 40 00 01 FF FF FF FF 00 04 xx 05
└─┬─┘ └─┬─┘ └─┬─┘ └────┬────┘ └─┬─┘ └─┬─┘
  │     │     │        │        │     └── element descriptor: xx05 = U8
  │     │     │        │        └──────── element string length: 4
  │     │     │        └───────────────── size of first dimension: -1 (variable - see data string)
  │     │     └────────────────────────── number of dimensions: 1
  │     └──────────────────────────────── type descriptor: xx40 = Array
  └────────────────────────────────────── type string length: 14
```

The corresponding data string is as follows.

```
00 00 00 02 00 01
└────┬────┘ └─┬─┘
     │        └── data: [0, 1]
     └─────────── size of first dimension: 2
```

| Data Type | Type String | Data String |
| --------- | ----------- | ----------- |
| Array | `[nn] xx40 [dims] [FFFF FFFF]... [type-info]` | `[dim-size]... [content]` |

# Cluster

Clusters have simple type strings that define the number of elements followed by the element type strings. The data string contains the corresponding data of every element in the same order as in the type string.

| Data Type | Type String | Data String |
| --------- | ----------- | ----------- |
| Cluster | `[nn] xx50 [elms] [type-info]...` | `[content]` |

# Waveform / Digital Waveform / Digital Data / Dynamic Data / Time Stamp

The type string of a Waveform consists of a waveform-specific preamble followed by a Cluster type string. The preamble contains only one element, which is the subtype code of the Waveform. This subtype code is specific to Waveforms and must not be confused with type descriptors. Find below a table of subtype codes.

Digital Waveform, Digital Data, Dynamic Data, and Time Stamp are subtypes of Waveform with specific subtype codes. The only difference is that they do not include a cluster type info but type definitions specific to their domain. By assuming generic types after the Waveform preamble, all types can be handled.

| Data Type | Type String | Data String |
| --------- | ----------- | ----------- |
| Waveform | `[nn] xx54 [subtype] [nested-type-info]` | `[content]` |

## Waveform Subtype Codes

| Type | Code |
| ---- | ---- |
| I16 | `0002` |
| DBL | `0003` |
| SGL | `0005` |
| Time Stamp | `0006` |
| Digital Data | `0007` |
| Digital Waveform | `0008` |
| Dynamic Data | `0009` |
| EXT | `000A` |
| U8 | `000B` |
| U16 | `000C` |
| U32 | `000D` |
| I8 | `000E` |
| I32 | `000F` |
| CSG | `0010` |
| CDB | `0011` |
| CXT | `0012` |
| I64 | `0013` |
| U64 | `0014` |

Subtype codes `0001` and `0004` are unknown.

# Variant

A variant has its own type and data strings. The type string only indicates that this is in fact a variant. The data string, however, is much more intricate. It begins with the version number of LabVIEW used to serialize the embedded type: `[version]`. The first word represents the major version (e.g., `1700` for LabVIEW 2017). The second word represents the minor version (e.g., `8000` for SP1). This is followed by an array of boxed type descriptors `[number-of-boxed-type-descriptors] [boxed-type-descriptors]...`.

> [!IMPORTANT]
> Each boxed type descriptor is either a referenced type descriptor or the type descriptor of the actual embedded type. For example, a cluster with three contained elements (numeric, bool, string) is represented as [numeric, bool, string, cluster] where cluster references the other type descriptors by reference instead of embedding them as a regular cluster would. This is especially efficient in situations where the same type is used multiple times. For example, a cluster with two elements of the same kind (string, numeric, string) is represented as [string, numeric, cluster] where cluster references string twice. The order of type descriptors in the data string is very important because the reference is resolved by index of appearance. For example, the cluster would reference [0, 1, 0].

After the list of type descriptors come two words: the first one appears to always be `0001` and might represent the number of primary type descriptors. But this is just speculation. The second one represents the number of referenced type descriptors - this is how many type descriptors are defined as reference points for the actual type.

What follows is the data string of the elements of the embedded type.

After the data string of the embedded type comes a 32-Bit number that indicates how many attributes are attached to the variant. A value of zero indicates no attributes. This is followed by the attributes, where each attribute has a key stored as a string with a 32-Bit length prefix and a nested variant for the value (this can be recursive).

| Data Type | Type String | Data String |
| --------- | ----------- | ----------- |
| Variant | `[nn] xx53` | `[version] [number-of-boxed-type-descriptors] [boxed-type-descriptors]... [number-of-primary-type-descriptors (assumption)] [number-of-referenced-type-descriptors] [content] [number-of-attributes] [[attribute-key][attribute-value]]...` |

# Map Collection
A Map Collection is effectively an sequence of key-value-pairs. Its type string always defines two type descriptors, one for the key and one for the value.

The data string defines the number of key-value-pairs followed by the actual key-value-pair values.

| Data Type | Type String | Data String |
| --------- | ----------- | ----------- |
| Map Collection | `[nn] xx74 [number-of-type-descriptors] [key-type-descriptor] [value-type-descriptor]` | `[number-of-elements] [[key-data] [value-data]]...` |

# Set Collection
A Set Collection is very similar to a one-dimensional array. Its type string always defines only one dimension.

The data string is a regular array.

> [!IMPORTANT]
> Set Collections are ordered and must not contain duplicate items. For manually constructed types, the caller is responsible for ensuring that this is the case. The behavior of invalid Set Collections is unknown.

| Data Type | Type String | Data String |
| --------- | ----------- | ----------- |
| Set Collection | `[nn] xx73 [number-of-dimensions] [element-type-string]` | `[number-of-elements] [element-data]...` |
