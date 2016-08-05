# Evaluator Guide

The audience for this guide is for developers who implement a full evaluator or extend an existing one to support additional cases. The purpose of this guide is to standardize on the common types and predicates supported by most databases or query services. This is to provide structure and general semantics for types and predicates.

## Types
Types are necessary for clients to understand how to represent values or present an interface to support entering a value. The following primitive types are defined.

### Concrete

#### `string`
A standard string encoded as UTF-8.

#### `bytes`
Array of arbitrary bytes. Formats that do not support byte arrays natively will receive data as base64-encoded strings.

#### `number`
Any real number.

#### `boolean`
A boolean, `true` or `false`.

#### `date`
A date, in the ISO 8601 format.

### Interpreted

#### `any`
The `any` type implies the predicate can be applied to any type. At runtime however, the type of the `term` will be used used by default.

#### `term`
A term refers to a value that is interpreted at runtime.

#### `union`
A union type allows for a value of one of the types defined. For example, a union type `number | string` allows for number or string values. It is responsibility of the evaluator to interpret this correctly. The common use is to use strings or some other type as a proxy for an alternate expression.

## Predicates
Predicates are functions that take zero or more parameters and return true or false. They are composed together to yield an expression that is evaluated.

### Operators

#### `equals(<term>, value=<any>)`
Exact match of a single value.

Examples
- `equals(gender, 'Male')`

#### `match(<term>, value=<string>)`
Case-insensitive match of string.

Examples
- `match(gender, 'male')`

#### `search(<term>, value=<string>)`
Case-insensitive match of a substring.

Examples
- `substr(diagnosis, 'cold')`

#### `regexp(<term>, value=<string>)`
Match using a regular expression.

Examples
- `regexp(note, '(\bEVA\b|(?i)enlarged vestibular aqueduct)')` - EVA (for enlarged vestibular aqueduct) or case insensitive phrase.

#### `range(<term>, gt=<any>, lt=<any>, gte=<any>, lte=<any>)`
Match on the range.

Examples
- `range(age, gt=20)` - unbounded upper bound
- `range(dob, lte='1975-01-01')` - unbounded lower bound, inclusive

#### `oneof(<term>, value=[<any>...])`
Match if at least one of the values is satisfied. This is semantically equivalent to defining `equals` predicates and wrapping them in `or`.

This operator generally applies to data having a one-to-many structure such as a document field holding an array of values, a reverse foreign key relationship in a relational data model, or any graph structure. However since only one case needs to match, structures that only support one value can be used here as well.

Examples
- `oneof(diagnosis, ['271.91', '392.18'])`

#### `allof(<term>, value=[<any>...])`
Match only if all of the values are satisfied. This is semantically equivalent to defining `equals` predicates and wrapping them in `and`.

See `oneof` above for an explanation of this operator type.

Examples
- `allof(diagnosis, ['271.91', '392.18'])`

#### `isnull(<term>)`
Match if no value is associated with the term. This is a unary operator and does not take any parameters.

Examples
- `isnull(gender)` - Gender is unknown.

## Logicals
A _logical_ is special type of predicate that combines two or more predicates using a logical operator such as `AND` and `OR`.

#### `and([<pred>...])`
Apply a logical AND between predicates.

Examples

```
and(
  oneof(diagnosis, ['392.17', '392.18']),
  allof(diagnosis, ['271.91', '271.92']),
)
```

#### `or([<pred>...])`
Apply a logical OR betwen predicates.

Examples

```
or(
  search(race, 'latino'),
  search(race, 'hispanic'),
)
```

#### `not(<pred>)`
Negate a predicate.

Examples

Gender is known, but it doesn't matter what it is.

```
not(isnull(gender))
```

