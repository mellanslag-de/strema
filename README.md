<h1 align="center">
  <code>strema</code>
</h1>

<p align="center">
  Experimental schema builder using TypeScript templates.
</p>


## Usage

Use `compileSchema` to create a schema.

```tsx
import { compileSchema } from "strema";

const schema = compileSchema(`{
  message: string;
  size: number <positive, int>;
  tags: string[];
  author: {
    name: string;
    email: string <email>;
    age: number <int, min(18)>;
  };
}`);
```

To get the type from the `schema`, use `ExtractSchemaType`.

```tsx
import { ExtractSchemaType } from "strema";

type Body = ExtractSchemaType<typeof schema>;
```

To validate and parse incoming data, use the `parseSync` method on `schema`.

```tsx
// Throws an error if req.body does not conform to the schema
const body = schema.parseSync(req.body);
```


## Overview

- [Field types](#Field_types)
  - [Primitives](#Primitives)
    - [String](#String)
      - [String rules](#String_rules)
    - [Number](#Number)
      - [Number rules](#Number_rules)
    - [Boolean](#Boolean)
  - [Objects](#Objects)
  - [Arrays](#Arrays)
  - [Records](#Records)
- [Rules](#Rules)
- [Optional fields](#Optional_fields)


## Field types

This library supports four field types.

- [Primitives](#Primitives)
- [Objects](#Objects)
- [Arrays](#Arrays)
- [Records](#Records)


### Primitives

There are three primitive types.

- [String](#String)
- [Number](#Number)
- [Boolean](#Boolean)


#### String

<table>
<tr>
<th>Schema</th>
<th>TypeScript type</th>
</tr>
<tr>
<td>

```tsx
const schema = compileSchema(`{
  email: string;
}`);
```
</td>
<td>

```tsx
{ email: string }
```
</td>
</tr>
</table>

##### String rules

 - `min(n)` sets a minimum length for the string.
 - `max(n)` sets a maximum length for the string.
 - `length(n)` equivalent to `min(n), max(n)`.
 - `email` the string value must be an email address.
 - `uuid` the string value must be a uuid.

A default value can be provided inside of double quotes:

```tsx
const schema = compileSchema(`{
  category: string <min(3)> = "other";
}`);
```


#### Number

<table>
<tr>
<th>Schema</th>
<th>TypeScript type</th>
</tr>
<tr>
<td>

```tsx
const schema = compileSchema(`{
  width: number;
}`);
```
</td>
<td>

```tsx
{ width: number }
```
</td>
</tr>
</table>

##### Number rules

 - `min(n)` sets a minimum value for the number.
 - `max(n)` sets a maximum value for the number.
 - `int` the value must be an integer.
 - `positive` equivalent to `min(0)`.

A default value can be provided:

```tsx
const schema = compileSchema(`{
  delayMs: number <positive> = 0;
}`);
```


#### Boolean

 ```tsx
const schema = compileSchema(`{
  include: boolean;
}`);
```

Booleans do not support any rules.

A default value of either `true` or `false` can be provided:

<table>
<tr>
<th>Schema</th>
<th>TypeScript type</th>
</tr>
<tr>
<td>

```tsx
const schema = compileSchema(`{
  include: boolean;
}`);
```
</td>
<td>

```tsx
{ include: boolean }
```
</td>
</tr>
</table>


## Rules

Primitive types support rules to perform basic validation. Rules are specified inside of `<>` after the type name and before `;` with multiple rules separated by `,`. If the rule takes an argument, provide it inside of `()` after the rule name.

```tsx
const schema = compileSchema(`{
  age: number <positive, int>;
  email: string <email>;
  password: string <min(8)>;
}`);
```

The available rules can be found here:

- [String rules](#String_rules)
- [Number rules](#Number_rules)
- Booleans do not support rules

Rules may also be applied to arrays (and multidimensional arrays). In those cases, specify the rules after all `[]`

```tsx
const schema = compileSchema(`{
  tags: string[] <min(1)>;
  coords: number[][] <int>;
}`);
```

Rules can not be applied directly to arrays or objects.


## Optional fields

By default, all fields are required. To mark a field as optional, add a `?` after the field name:

<table>
<tr>
<th>Schema</th>
<th>TypeScript type</th>
</tr>
<tr>
<td>

```tsx
const schema = compileSchema(`{
  description?: string;
}`);
```
</td>
<td>

```tsx
{ description: string | null }
```
</td>
</tr>
</table>


### Optional primitive fields

When a primitive field is optional, `null` and `undefined` values are not rejected.

```tsx
const schema = compileSchema(`{
  description?: string;
}`);

const output = schema.parseSync({ description: undefined });

console.log(output);
//=> { description: null }
```


### Optional array fields

Optional arrays behave in the same way as primitives.

```tsx
const schema = compileSchema(`{
  tags?: string[];
}`);

const output = schema.parseSync({ tags: undefined });

console.log(output);
//=> { tags: null }
```


### Optional object fields

Object fields behave the same as primitives and arrays, with the exception that object fields with no required fields accept `null` and `undefined`.

```tsx
const schema = compileSchema(`{
  options: { notify?: boolean; delay?: number };
}`);

const output = schema.parseSync({ options: undefined });

console.log(output.options);
//=> { notify: null, delay: null }
```

However, if the object is optional, it resolves to `null` when `null` or `undefined` are provided.

```tsx
const schema = compileSchema(`{
  options?: { notify?: boolean; delay?: number };
}`);

const output = schema.parseSync({ options: undefined });

console.log(output.options);
//=> null
```


### Optional record fields

Records fields are always optional. Using the `?:` optional notation throws an error.

```tsx
const schema = compileSchema(`{
  record?: Record<string, string>;
}`);
// Throws: Type 'record' cannot be optional
```