# Evaluator Guide

The audience for this guide is for developers who implement a full evaluator or extend an existing one to support additional cases. The purpose of this guide is to standardize on the common types and predicates supported by most databases or query services.

## Types
Types are necessary for clients to understand how to represent values or present an interface to support entering a value. The following primitive types are defined.

#### `string`
A standard string encoded as UTF-8.

#### `bytes`
Byte array of arbitrary bytes. Processes that do not support byte arrays natively will receive data as base64-encoded strings.

#### `number`
Any real number.

#### `boolean`
A boolean, `true` or `false`.

#### `term`
A term refers to a value that is interpreted at runtime.

#### `union`
A union type allows for a value of one of the types defined. For example, a union type `number | string` allows for number or string values. It is responsibility of the evaluator to interpret this correctly. The common use is to use strings or some other type as a proxy for an alternate expression.

## Logicals
A _logical_ is special type of predicate that combines two or more predicates using a logical operator such as `AND` and `OR`.

#### `and([<pred>...])`
Apply a logical AND of the predicates.

#### `or([<pred>...])`
Apply a logical OR of the predicates.

## Predicates
Logically, predicates are functions that take zero or more parameters and return true or false. They are The `any` type is only part of the definition, but the concrete operator returned by an evaluator can specify an explicit type of the parameter. If not defined, the type of the `term` will be used.

#### `equals(<term>, <any>)`
Exact match of a single value.

**Examples**
- `equals(gender, 'Male')`

#### `match(<term>, <string>)`
Case-insensitive match of string.

**Examples**
- `match(gender, 'male')`

#### `search(<term>, <string>)`
Case-insensitive match of a substring.

**Examples**
- `substr(diagnosis, 'cold')`

#### `regexp(<term>, <string>)`
Match using a regular expression.

**Examples**
- `regexp(note, '(\bEVA\b|(?i)enlarged vestibular aqueduct)')` - EVA (for enlarged vestibular aqueduct) or case insensitive phrase.

#### `range(<term>, gt=<number>, lt=<number>, gte=<number>, lte=<number>)`
Match on the range.

**Examples**
- `range(age, gt=20)` - unbounded upper bound
- `range(age, lte=80)` - unbounded lower bound, inclusive

#### `oneof(<term>, [<any>...])`
Match if at least one of the values is satisfied.

**Examples**
- `oneof(diagnosis, ['271.91', '392.18'])`

#### `allof(<term>, [<any>...])`
Match only if all of the values are satisfied.

**Examples**
- `allof(diagnosis, ['271.91', '392.18'])`

#### `isnull(<term>)`
Match if no value is associated with the term. This is a unary operator and does not take any parameters.

**Examples**
- `isnull(gender)` - Gender is unknown.
- `-isnull(location) - Gender is known, the value does not matter.
