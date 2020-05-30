---
title: Objects
---

# Objects

## Object Literal

##### Syntax
```imba
let object = {a: 'foo', b: 42, c: {}}
console.log(object.a) # => 'foo'
```

##### Destructuring
```imba
let a = 'foo'
let b = 42
let c = {}
let object = {a,b,c}
console.log(object) # => {a: 'foo', b: 42 c: {}}
```


##### Dynamic strings
```imba
"imba version {imba.version}"
```

Double quoted strings can

##### Template strings
```imba
`hello world`
`imba version {imba.version}`
```
> In JavaScript `${}` is used for interpolation. Imba uses `{}`. If you want interpolated strings with literal curly-braces, remember to escape them with `\`