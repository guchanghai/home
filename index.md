# Memory Leak in JavaScript

## Common memory usage

![Common memory usage](https://user-images.githubusercontent.com/2970098/72621557-5658ec80-38f6-11ea-9189-950ce83d217a.png)

## API to show memory usage

[process.memoryUsage](https://nodejs.org/api/process.html#process_process_memoryusage)

The `process.memoryUsage()` method returns an object describing the memory usage of the Node.js process measured in bytes.

For example, the code:
```
console.log(process.memoryUsage());
```
Will generate:
```
{
  rss: 4935680,
  heapTotal: 1826816,
  heapUsed: 650472,
  external: 49879
}
```

Type | Description
--- | ---
rss | Resident Set Size. It is the total memory allocated for the process execution
heapTotal | Total size of the allocated heap
heapUsed | Actual memory used during the execution of our process
external | Memory used by “C++ objects bound to JavaScript objects managed by V8

## Memory in JavaScipt

```
var numberVar = 100;                // allocates memory for a number

var stringVar = 'node simplified';  // allocates memory for a string

var objectVar = {a: 1};             // allocates memory for an object

var a = [1, null, 'abra'];          // allocates memory for an array

function f(a) {                     // allocates memory for a function
  return a + 2;
} 
```

Primitive value is member of one of the types Undefined, Null, Boolean, Number, or String as defined in Clause 8

Code from NPM [object-sizeof](https://www.npmjs.com/package/object-sizeof) to calcuate JS object size
```
const ECMA_SIZES = {
  STRING: 2,      // Each String value is represented by 16-bit unsigned integer
  BOOLEAN: 4,
  NUMBER: 8       // Number uses the double-precision 64-bit format IEEE 754 values
}

function sizeof (object) {
  var objectType = typeof (object)
  switch (objectType) {
    case 'string':
    return object.length * ECMA_SIZES.STRING
    case 'boolean':
      return ECMA_SIZES.BOOLEAN
    case 'number':
      return ECMA_SIZES.NUMBER
    case 'object':
      if (Array.isArray(object)) {
        return object.map(sizeof).reduce(function (acc, curr) {
          return acc + curr
        }, 0)
      } else {
        return sizeOfObject(object)
      }
    default:
      return 0
  }
}
```
(ECMAScript Language Specification)

## Garbage collection

### Reference-counting garbage collection

### Mark-and-sweep algorithm

# Knowledge Base

## Distance

## Shallow size vs Retained size

# Find the memory leak

# Other memory leak

Memory leak in global object is the most common case. There are another kind of memory leak: closure.

# Useful links

[Memory Management and Garbage Collection in JavaScript](https://dzone.com/articles/memory-management-and-garbage-collection-in-javasc)

[Memory Terminology](https://developers.google.com/web/tools/chrome-devtools/memory-problems/memory-101#object_sizes)

Tools to analyze the memory leak.

