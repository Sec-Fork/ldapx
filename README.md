# ldapx

[![Go Report Card](https://goreportcard.com/badge/github.com/Macmod/ldapx)](https://goreportcard.com/report/github.com/Macmod/ldapx)
[![GoDoc](https://godoc.org/github.com/Macmod/ldapx?status.svg)](https://godoc.org/github.com/Macmod/ldapx)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Flexible LDAP proxy that can be used to inspect, transform or encrypt to LDAPS all LDAP packets generated by other tools on the fly.

## Installation

```bash
go get github.com/Macmod/ldapx
```

## Usage

```bash
ldapx -target LDAPSERVER:389 [-f MIDDLEWARECHAIN] [-a MIDDLEWARECHAIN] [-b MIDDLEWARECHAIN] [-listen LOCALADDR:PORT]
```

Where:
* `-f` will apply Filter middlewares to all search requests
* `-a` will apply AttrList middlewares to all search requests
* `-b` will apply BaseDN middlewares to all search requests

`-debug` can also be provided to make it show verbose logs.

Each middleware is specified by a single-letter key (detailed below), and can be specified multiple times.
For each type of middleware, the middlewares in the chain will be applied *in the order that they are specified* in the command.

## Examples
(TODO)

## Library Usage 
(TODO)

## Middlewares
The library provides several middlewares for LDAP filter transformation:

| Type | Key | Name | Purpose | Description | Input | Output | Details |
|------|-----|------|---------|-------------|--------|--------|---------|
| Filter | <kbd>S</kbd> | Spacing | Obfuscation | Adds random spaces between characters | `(cn=john)` | `( c n = j o h n )` | Max spaces configurable |
| Filter | <kbd>T</kbd> | Timestamp | Obfuscation | Adds random chars to timestamp values | `(time=20230812Z)` | `(time=20230812abcZ)` | Prepend/append configurable |
| Filter | <kbd>B</kbd> | AddBool | Obfuscation | Adds random boolean conditions | `(cn=john)` | `(&(cn=john)(\|(a=1)(a=2)))` | Max depth configurable |
| Filter | <kbd>D</kbd> | DblNegBool | Obfuscation | Adds double negations | `(cn=john)` | `(!(!(cn=john)))` | Max depth configurable |
| Filter | <kbd>M</kbd> | DeMorganBool | Obfuscation | Applies De Morgan's laws | `(!(\|(a=1)(b=2)))` | `(&(!(a=1))(!(b=2)))` | Probability based |
| Filter | <kbd>O</kbd> | OIDAttribute | Obfuscation | Converts attrs to OIDs | `(cn=john)` | `(2.5.4.3=john)` | Uses standard LDAP OIDs |
| Filter | <kbd>C</kbd> | Case | Obfuscation | Randomizes character case | `(cn=John)` | `(cN=jOhN)` | Probability based |
| Filter | <kbd>X</kbd> | HexValue | Obfuscation | Hex encodes characters | `(cn=john)` | `(cn=\6a\6f\68\6e)` | Probability based |
| Filter | <kbd>R</kbd> | ReorderBool | Obfuscation | Reorders boolean conditions | `(&(a=1)(b=2))` | `(&(b=2)(a=1))` | Random reordering |
| AttrList | <kbd>C</kbd> | Case | Obfuscation | Randomizes attribute case | `cn,sn` | `cN,Sn` | Probability based |
| AttrList | <kbd>O</kbd> | OIDAttribute | Obfuscation | Converts to OID form | `cn,sn` | `2.5.4.3,2.5.4.4` | Uses standard LDAP OIDs |
| AttrList | <kbd>G</kbd> | GarbageNonExisting | Obfuscation | Adds fake attributes | `cn,sn` | `cn,sn,x-123` | Garbage is chosen randomly from an alphabet |
| AttrList | <kbd>g</kbd> | GarbageExisting | Obfuscation | Adds real attributes | `cn` | `cn,sn,mail` | Garbage is chosen from real attributes |
| AttrList | <kbd>S</kbd> | Spacing | Obfuscation | Adds random spaces in the attributes | `cn,name` | `c n,n  Am e` | Max spaces configurable |
| AttrList | <kbd>D</kbd> | Duplicate | Obfuscation | Duplicates attributes | `cn` | `cn,cn,cn` | Max duplicates configurable |
| AttrList | <kbd>W</kbd> | AddWildcard | Obfuscation | Adds a wildcard attribute to the list | `cn,name` | `cn,name,*` |  |
| AttrList | <kbd>w</kbd> | ReplaceWithWildcard | Obfuscation | Replaces the list with a wildcard | `cn,sn` | `*` | Replaces all attributes |
| AttrList | <kbd>E</kbd> | ReplaceWithEmpty | Obfuscation | Empties the attributes list | `cn,sn` | | |
| AttrList | <kbd>R</kbd> | ReorderList | Obfuscation | Randomly reorders attrs | `cn,sn,uid` | `uid,cn,sn` | Random permutation |
| BaseDN | <kbd>C</kbd> | Case | Obfuscation | Randomizes DN case | `CN=lol,DC=draco,DC=local` | `cN=lOl,dC=dRaCo,Dc=loCaL` | Probability based |
| BaseDN | <kbd>O</kbd> | OIDAttribute | Obfuscation | Converts DN attrs to OIDs | `cn=Admin` | `2.5.4.3=Admin` | Uses standard LDAP OIDs |
| BaseDN | <kbd>Z</kbd> | OIDPrependZeros | Obfuscation | Prepends zeros to OID components | `2.5.4.3=admin` | `002.0005.04.03=admin` | Only applies if there are OID components (for instance, by applying O before) |
| BaseDN | <kbd>S</kbd> | Spacing | Obfuscation | Adds random spaces in the BaseDN | `DC=draco` | `DC=draco     ` | Min/max spaces/probEnd configurable |
| BaseDN | <kbd>Q</kbd> | DoubleQuotes | Obfuscation | Adds quotes to values | `cn=Admin` | `cn="Admin"` |  |
| BaseDN | <kbd>X</kbd> | HexValue | Obfuscation | Hex encodes characters in the values | `cn=john` | `cn=\6a\6fmin` | Probability based | 

### Implementation status
* Filter - `Spacing` and `HexValue` not working properly
* AttrList - `Case` and `Spacing` not working properly
* BaseDN - Six methods working (spaces only work in beginning and end / hex only works in the values)

## Contributing

Contributions are also welcome by [opening an issue](https://github.com/Macmod/ldapx/issues/new) or by [submitting a pull request](https://github.com/Macmod/ldapx/pulls).

## License
MIT License

Copyright (c) 2023 Artur Henrique Marzano Gonzaga

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.