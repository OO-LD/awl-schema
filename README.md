# Abstract Workflow Language Schema

Application of [OO-LD](https://github.com/OO-LD/awl-schema) on [Abstract Syntax Trees](https://en.wikipedia.org/wiki/Abstract_syntax_tree) in order share workflow descriptions and map them to RDF.

For illustration we use the AST generated by the python std-lib [ast module](https://docs.python.org/3/library/ast.html). However, the concept should work with any AST (examples see https://astexplorer.net/) although it is recommeded to stick to core language patterns with broad language support.

## Objectives
- Language-agnostic JSON serialization
- Support for any pattern that is supported by programming languages (Conditions, Loops, Function calls)
- Option to restrict supported features via JSON-SCHEMA (e.g. don't allow class declarations, allow only a whitelist of function calls, restrict function call parameters)
- Option to map to RDF via JSON-LD context in order to allow complex analytics and queries via SPARQL or SHACL (e.g. find all workflows that make use of certain functions in a specific order)

## Playground
A playground to work with AWL can be found here: https://oo-ld.github.io/playground-awl/ (Work in progress)

## Schema

### Python-AST
Note: The context makes use of type-scoped contexts, see https://www.w3.org/TR/json-ld11/#scoped-contexts

Note: The schema section is empty allowing any AST. However, workflow domains should define a restricted set of e.g. function calls that represent available and safe options.

```yaml
'@context':
  - awl: https://oo-ld.github.io/awl-schema/
    ex: https://example.org/
    '@base': https://oo-ld.github.io/awl-schema/
    _type: '@type'
    id: '@id'
    body: awl:HasPart
    Name:
      '@id': awl:Variable
      '@context':
        '@base': https://example.org/
    targets: awl:HasTarget
    value:
      '@id': awl:HasValue
      '@context':
        value: '@value'
    If:
      '@id': awl:If
      '@context':
        body: awl:IfTrue
    orelse: awl:IfFalse
    test: awl:HasCondition
    comparators: awl:HasRightHandComparator
    ops: awl:HasOperator
    left: awl:HasLeftHandComparator
    func:
      '@id': awl:HasFunctionCall
      '@context':
        '@base': https://example.org/
        Name: awl:Function
    args: awl:HasArgument
    _keywords: awl:HasKeywordArgument
    arg: awl:HasKey
title: AWL
type: object
```

## Examples

### If-Else-Block

```python
if a == 1:
    b = 1
else:
    b = 'test'
```
> Python Code


```yaml
_type: Module
body:
  - _type: If
    body:
      - _type: Assign
        targets:
          - _type: Name
            id: b
        value:
          _type: Constant
          value: 1
    orelse:
      - _type: Assign
        targets:
          - _type: Name
            id: b
        value:
          _type: Constant
          value: test
    test:
      _type: Compare
      comparators:
        - _type: Constant
          value: 1
      left:
        _type: Name
        id: a
      ops:
        - _type: Eq
type_ignores: []
```
> AST


```turtle
<https://example.org/run> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://oo-ld.github.io/awl-schema/Function> .
<https://example.org/x> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://oo-ld.github.io/awl-schema/Variable> .
_:b0 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://oo-ld.github.io/awl-schema/Module> .
_:b0 <https://oo-ld.github.io/awl-schema/HasPart> _:b1 .
_:b0 <https://oo-ld.github.io/awl-schema/HasPart> _:b3 .
_:b1 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://oo-ld.github.io/awl-schema/Assign> .
_:b1 <https://oo-ld.github.io/awl-schema/HasTarget> <https://example.org/x> .
_:b1 <https://oo-ld.github.io/awl-schema/HasValue> _:b2 .
_:b2 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://oo-ld.github.io/awl-schema/Call> .
_:b2 <https://oo-ld.github.io/awl-schema/HasFunctionCall> <https://example.org/run> .
_:b3 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://oo-ld.github.io/awl-schema/Expr> .
_:b3 <https://oo-ld.github.io/awl-schema/HasValue> _:b4 .
_:b4 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://oo-ld.github.io/awl-schema/Call> .
_:b4 <https://oo-ld.github.io/awl-schema/HasArgument> <https://example.org/x> .
_:b4 <https://oo-ld.github.io/awl-schema/HasFunctionCall> <https://example.org/run> .
```
> RDF


### Funtion call

```python
x = run(a=1))
run(x)
```
> Python Code


```yaml
_type: Module
body:
  - _type: Assign
    body: []
    value:
      _type: Call
      args: []
      func:
        _type: Name
        id: run
      keywords:
        - _type: keyword
          arg: a
          value:
            _type: Constant
            value: 1
    targets:
      - _type: Name
        id: x
  - _type: Expr
    body: []
    value:
      _type: Call
      args:
        - _type: Name
          id: x
      func:
        _type: Name
        id: run
      keywords: []
type_ignores: []
```
> AST


```turtle
<https://example.org/run> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://oo-ld.github.io/awl-schema/Function> .
<https://example.org/x> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://oo-ld.github.io/awl-schema/Variable> .
_:b0 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://oo-ld.github.io/awl-schema/Module> .
_:b0 <https://oo-ld.github.io/awl-schema/HasPart> _:b1 .
_:b0 <https://oo-ld.github.io/awl-schema/HasPart> _:b3 .
_:b1 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://oo-ld.github.io/awl-schema/Assign> .
_:b1 <https://oo-ld.github.io/awl-schema/HasTarget> <https://example.org/x> .
_:b1 <https://oo-ld.github.io/awl-schema/HasValue> _:b2 .
_:b2 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://oo-ld.github.io/awl-schema/Call> .
_:b2 <https://oo-ld.github.io/awl-schema/HasFunctionCall> <https://example.org/run> .
_:b3 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://oo-ld.github.io/awl-schema/Expr> .
_:b3 <https://oo-ld.github.io/awl-schema/HasValue> _:b4 .
_:b4 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <https://oo-ld.github.io/awl-schema/Call> .
_:b4 <https://oo-ld.github.io/awl-schema/HasArgument> <https://example.org/x> .
_:b4 <https://oo-ld.github.io/awl-schema/HasFunctionCall> <https://example.org/run> .
```
> RDF
