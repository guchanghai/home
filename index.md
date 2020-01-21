<!-- vscode-markdown-toc -->
* 1. [What is happening](#Whatishappening)
* 2. [Container is restarted by Ethos ELB](#ContainerisrestartedbyEthosELB)
* 3. [Impact to the service](#Impacttotheservice)
* 4. [Common memory usage](#Commonmemoryusage)
* 5. [I was told that uou should NOT worry about memory in JS](#IwastoldYoushouldNOTworryaboutmemoryinJS)
* 6. [Memory of node process](#Memoryofnodeprocess)
* 7. [API to show memory usage](#APItoshowmemoryusage)
* 8. [Relationship between NewRelic, Memory Sanpshot and Real Memory](#RelationshipbetweenNewRelicMemorySanpshotandRealMemory)
* 9. [Memory in JavaScipt](#MemoryinJavaScipt)
* 10. [Garbage collection](#Garbagecollection)
	* 10.1. [Reference-counting garbage collection](#Reference-countinggarbagecollection)
	* 10.2. [Mark-and-sweep algorithm](#Mark-and-sweepalgorithm)
* 11. [Distance](#Distance)
* 12. [Shallow size vs Retained size](#ShallowsizevsRetainedsize)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

# Memory Leak in JavaScript

##  1. <a name='Whatishappening'></a>What is happening

![image](https://git.corp.adobe.com/storage/user/1451/files/6a922900-3b44-11ea-8d98-514a35b95b60)

You can see more "JavaScript heap out of memory" from this [Splunk Query](https://splunk-us.corp.adobe.com/en-US/app/app_UnifiedCheckout/search?q=search%20index%3D%22unifiedcheckout-production%22%20sourcetype%3D%22unified-checkout-*v2-*%22%20%22JavaScript%20heap%20out%20of%20memory%22&display.page.search.mode=fast&dispatch.sample_ratio=1&earliest=1573718400&latest=1574409600&display.page.search.tab=events&display.general.type=events&display.prefs.statistics.count=100&sid=1579534031.773627_2B378380-5A57-4C9D-9527-0AA5F28C5C3C)

##  2. <a name='ContainerisrestartedbyEthosELB'></a>Container is restarted by Ethos ELB

![image](https://git.corp.adobe.com/storage/user/1451/files/5ea76680-3b46-11ea-9488-2989d52ca44d)

##  3. <a name='Impacttotheservice'></a>Impact to the service

![image](https://git.corp.adobe.com/storage/user/1451/files/294f4880-3b47-11ea-971f-d99e944f2bce)

##  4. <a name='Commonmemoryusage'></a>Common memory usage

This is what I learned in colledge

![Common memory usage](https://user-images.githubusercontent.com/2970098/72621557-5658ec80-38f6-11ea-9189-950ce83d217a.png)

When I wrote code in C/C++, I manage memory carefully.

In C, we always remember to `free` the memory that you `malloc`ed.

```c
void test()
{
    char *pData = (char *)malloc(1024);
    strcpy(pData, "Hello world!");
    printf("%s\n", pData);
    free(pData);
}
```

In C++, we need to `delete` the object that you `new`ed

```cpp
void test()
{
    char *pData = new char[1024];
    strcpy(pData, "Hello world!");
    printf("%s\n", pData);
    delete[] pData;
}
```
Also in C++, we should take the advantage of constructor/destuctor to manage the memory easily
```cpp

class CTest
{
    char *pData;

public:
    CTest()
    {
        printf("Memory is allocated in constructor\n");
        char *pData = new char[1024];
        strcpy(pData, "Hello world!");
        printf("%s\n", pData);
    }

    ~CTest()
    {
        printf("Memory is freed in desctructor\n");
        delete[] pData;
        pData = NULL;
    }
};

void testCpp()
{
    CTest *pTest = new CTest();
    delete pTest;
}
```

Output:
```
Memory is allocated in constructor
Hello world!
Memory is freed in desctructor
```

##  5. <a name='IwastoldYoushouldNOTworryaboutmemoryinJS'></a>I was told that you should NOT worry about memory in JS

From MDN, this is what I see about [Memory Management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management)

```
Low-level languages like C, have manual memory management primitives such as malloc() and free().
In contrast, JavaScript automatically allocates memory when objects are created and frees it
when they are not used anymore (garbage collection)...
```

##  6. <a name='Memoryofnodeprocess'></a>Memory of node process

![image](https://git.corp.adobe.com/storage/user/1451/files/cfa24a80-3b55-11ea-9abb-9b289c19fd1f)

##  7. <a name='APItoshowmemoryusage'></a>API to show memory usage

[process.memoryUsage](https://nodejs.org/api/process.html#process_process_memoryusage)

The `process.memoryUsage()` method returns an object describing the memory usage of the Node.js process measured in bytes.

For example, the code:
```javascript
console.log(process.memoryUsage());
```
Will generate:
```json
{
  rss: 56664064
  heapTotal: 10100736
  heapUsed: 6119800
  external: 9126
}
```

Type | Description
--- | ---
rss | Resident Set Size. It is the total memory allocated for the process execution
heapTotal | Total size of the allocated heap
heapUsed | Actual memory used during the execution of our process
external | Memory used by C++ objects bound to JavaScript objects managed by V8

![Rss meory layout](https://user-images.githubusercontent.com/2970098/72691310-46e5c900-3ad9-11ea-9c0c-3203b9a94c81.png)

You can get the similar memory usage from [NewRelic](https://rpm.newrelic.com/accounts/1347546/applications/123232391_h474457217/node_vms)

![New Relice Memory Layout](https://git.corp.adobe.com/storage/user/1451/files/cb276300-3b50-11ea-98af-2ed7c18232ee)

## Run automation locally 

```
"memoryUsage":{"rss":131739648,"heapTotal":40116224,"heapUsed":32697816,"external":1047468},"time":"2019-12-02T07:14:41.067Z",
"memoryUsage":{"rss":132259840,"heapTotal":40116224,"heapUsed":33502592,"external":1052842},"time":"2019-12-02T07:14:41.773Z",
"memoryUsage":{"rss":148410368,"heapTotal":68083712,"heapUsed":49797128,"external":3993900},"time":"2019-12-02T07:14:57.706Z",
"memoryUsage":{"rss":160780288,"heapTotal":74899456,"heapUsed":55844696,"external":19245884},"time":"2019-12-02T07:15:28.910Z"
"memoryUsage":{"rss":168361984,"heapTotal":65986560,"heapUsed":48579832,"external":23102744},"time":"2019-12-02T07:15:34.806Z"
"memoryUsage":{"rss":166612992,"heapTotal":70705152,"heapUsed":50833864,"external":17550980},"time":"2019-12-02T07:15:48.487Z"
"memoryUsage":{"rss":166629376,"heapTotal":70705152,"heapUsed":51287808,"external":17550996},"time":"2019-12-02T07:15:49.217Z"
"memoryUsage":{"rss":176062464,"heapTotal":73326592,"heapUsed":57472288,"external":31662260},"time":"2019-12-02T07:16:21.767Z"
"memoryUsage":{"rss":177221632,"heapTotal":69132288,"heapUsed":49668576,"external":17947792},"time":"2019-12-02T07:16:38.963Z"
"memoryUsage":{"rss":177307648,"heapTotal":69132288,"heapUsed":50104368,"external":17947808},"time":"2019-12-02T07:16:39.805Z"
"memoryUsage":{"rss":183369728,"heapTotal":77520896,"heapUsed":57851200,"external":31839054},"time":"2019-12-02T07:17:01.712Z"
"memoryUsage":{"rss":180834304,"heapTotal":78569472,"heapUsed":50591232,"external":14007378},"time":"2019-12-02T07:17:29.909Z"
"memoryUsage":{"rss":183758848,"heapTotal":71753728,"heapUsed":48792232,"external":17712637},"time":"2019-12-02T07:17:46.426Z"
"memoryUsage":{"rss":184082432,"heapTotal":71753728,"heapUsed":49239944,"external":17712653},"time":"2019-12-02T07:17:47.415Z"
```

And then import the data to excel to create a chart

![Memory usage in local docker](https://git.corp.adobe.com/storage/user/1451/files/3b3fe400-3b65-11ea-8058-95a5178d8bbd)

## Debug Node.js

Get more info from [Node.js debugging](https://nodejs.org/de/docs/guides/debugging-getting-started/)

Flag | Meaning
--- | ---
--inspect | Enable inspector agent Listen on default address and port (127.0.0.1:9229)
--inspect=[host:port] | Enable inspector agent <br> Bind to address or hostname host (default: 127.0.0.1) <br> Listen on port port (default: 9229)
--inspect-brk | Enable inspector agent Listen on default address and port (127.0.0.1:9229) <br> Break before user code starts

Modify your package.json to start the service with debugger enabled
```
cross-env NODE_ENV=production node --inspect-brk=0.0.0.0:9229 --expose-gc build/main.js
```

Attach Chrome Debugger to node.js

Open Chrome browser and input URL of `chrome://inspect/#devices`, then you will see the debugger sessions

![image](https://git.corp.adobe.com/storage/user/1451/files/b6b98980-3b95-11ea-982c-5436171b58da)

After clicking the link, you will attach the Chrome debugger to the node.js process. Then you can take a snapshot of the memory

![image](https://git.corp.adobe.com/storage/user/1451/files/6efb2800-3bbe-11ea-9eba-9ac6b764cc70)

[v8.getHeapStatistics](https://nodejs.org/api/v8.html#v8_v8_getheapstatistics)

Returns an object with the following properties:

```json
{
  total_heap_size: 10100736
  total_heap_size_executable: 3145728
  total_physical_size: 7984200
  total_available_size: 1518306464
  used_heap_size: 5476464
  heap_size_limit: 1526909922
  malloced_memory: 16384
  peak_malloced_memory: 421000
  does_zap_garbage: 0
}
```

##  8. <a name='RelationshipbetweenNewRelicMemorySanpshotandRealMemory'></a>Relationship between NewRelic, Memory Sanpshot and Real Memory

![image](https://git.corp.adobe.com/storage/user/1451/files/13945000-3b54-11ea-83e6-3900f6d393bd)

##  9. <a name='MemoryinJavaScipt'></a>Memory Heap in JavaScipt

More detail about the memory heap

![Memory heap](https://git.corp.adobe.com/storage/user/1451/files/5b53bf80-3bc6-11ea-87a5-f1a597aa1cce)

Item | Description
--- | ---
Constructor | Represents all objects created using this constructor (x`number`: Number of object instances)
Distance | The distance to the root using the shortest simple path of nodes.
Shallow size | Sum of shallow sizes of all objects created by a certain constructor function. <br> The shallow size is the size of memory held by an object itself (generally, arrays and strings have larger shallow sizes)
Retained size | The size of memory that can be freed once an object is deleted <br> (and this its dependents made no longer reachable) is called the retained size.

See more detail from [Object Size](https://developers.google.com/web/tools/chrome-devtools/memory-problems/memory-101#object-sizes)


There are many kinds of JavaScript objects in the heap

```javascript
var numberVar = 100;                // allocates memory for a number

var stringVar = 'node simplified';  // allocates memory for a string

var objectVar = {a: 1};             // allocates memory for an object

var a = [1, null, 'abra'];          // allocates memory for an array

var cacheMap = new Map();	    // allocates memory for a Map
```

[JavaScript object representation](https://developers.google.com/web/tools/chrome-devtools/memory-problems/memory-101#javascript_object_representation)

Type | Defintion | Comment
--- | --- | ---
Primitive | Numbers (e.g., 3.14159..) <br> Booleans (true or false) <br> Strings (e.g., 'Werner Heisenberg') | They cannot reference other values and are always leafs or terminating nodes.
Objects | Defined as an unordered collection of related data, of primitive or reference types, in the form of “key: value” pairs | They can hold reference to the other primitive object or group object

### Memory Graph

The memory graph starts with a `root`, which may be the `window` object of the browser or the `Global` object of a Node.js module. 

![Memory Graph](https://git.corp.adobe.com/storage/user/1451/files/6dd4f580-3bd4-11ea-8156-70aaab79b436)

### Retaining Path

Retaning path is any path from root to a particular object

![Retaining Path](https://git.corp.adobe.com/storage/user/1451/files/3a966480-3bdc-11ea-917d-7ce38d695863)

For the code
```javascript
global.gArray = [];

function basicDataType() {
  const arr = Array(1e6).fill("a");   	// ["a","a","a","a","a"...].length === 1,000,000
  const arrStr = arr.join(""); 		// "aaaaaaaaaaaaaaaaaaa..."
  gArray.push(arrStr);
}

function getBasicObject() {
  const obj = {
    aNumber: 2,
    aStr: Array(1e6)			// "bbbbbbbbbbbbbbbbbbb...".length === 1,000,000
      .fill("b")
      .join("")
  };
  gArray.push(obj);
}

function test() {
  basicDataType();
  getBasicObject();
}

test();
```

![Memory Graph](https://git.corp.adobe.com/storage/user/1451/files/ad550f00-3bdf-11ea-892d-bb81795855e1)

See the memory snapshot

![Object Size](https://git.corp.adobe.com/storage/user/1451/files/b349f080-3bdd-11ea-96bc-ab98893a3993)

Code from NPM [object-sizeof](https://www.npmjs.com/package/object-sizeof) to calcuate JS object size
```javascript
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

function sizeOfObject (object) {
  var bytes = 0
  for (var key in object) {
    bytes += sizeof(key)
    bytes += sizeof(object[key])
  }

  return bytes
}

```
(ECMAScript Language Specification)

# Find the memory leak

## Find the large object

Based on what we have learned, the first thought is 

1. Run the application for a while.
2. Take a memory snapshot.
3. Sort the retained column and find the large objects. 

![image](https://git.corp.adobe.com/storage/user/1451/files/8b787e00-3c19-11ea-8e29-76c1c04b20ac)

Very happy, right? I find the memory leak! But after detail finding you will be very sad, because

* There are too many objects.
* Each object is too small.
* You cannot find where the objects came from.

![image](https://git.corp.adobe.com/storage/user/1451/files/df379700-3c1a-11ea-9d66-d8dbf27d4568)

![image](https://git.corp.adobe.com/storage/user/1451/files/0b9ee380-3c1a-11ea-80e2-2a32e9e0a3f2)

![image](https://git.corp.adobe.com/storage/user/1451/files/ac45d100-3c23-11ea-9046-5288e4bda2c6)

![image](https://git.corp.adobe.com/storage/user/1451/files/c202c880-3c1a-11ea-8b0d-c9a82a410ed0)

### After a long time seaching...

I find one Map is growing fast

![image](https://git.corp.adobe.com/storage/user/1451/files/7baa6b00-3c14-11ea-9a6c-55f494714d3d)

There is an object `axiosCacheMap` is referenced by `global`.

```javascript
    const cacheMap = global.axiosCacheMap || new Map();
    if( !cacheMap.get( svc )){
      cacheMap.set( svc, cache );
      log.info({ msg: 'cache: adding cache to cacheMap', cache: svc, size: cacheMap.size });
      global.axiosCacheMap = cacheMap;
    }
```

You can hover on the object to see what's content of the Map

![image](https://git.corp.adobe.com/storage/user/1451/files/7f8abd00-3c15-11ea-968f-e799c8ce760f)

You can continue to see what's the content for each cache.

![image](https://git.corp.adobe.com/storage/user/1451/files/9e894f00-3c15-11ea-9a5c-d5523001d55a)

You will very happy because you think you find the `memory leak`. But after calculating based on the Splunk query, you will be sad. Because the total cache is just about 100MB.

![image](https://git.corp.adobe.com/storage/user/1451/files/1b1c2d80-3c16-11ea-9e6a-e0433f8dad60)

## Compare memory snapshots

After the first finding, you will find searching one by one is pretty hard. Not just because there are too many objects, also some objects disappeared. 

Every object has an ID: See this object

![image](https://git.corp.adobe.com/storage/user/1451/files/633e4e80-3c1c-11ea-9d63-cb015ae12672)

But ater it disapperared later

![image](https://git.corp.adobe.com/storage/user/1451/files/dba50f80-3c1c-11ea-86e2-a6e9290e4691)

Is there another way to investigate? Are some objects alwary `created` but never `deleted`?

![image](https://git.corp.adobe.com/storage/user/1451/files/df856180-3c1d-11ea-9966-d9563aa3fb14)

Based on this, I found another object `LRUCache`. It is growing fast and never `deleted`.

![image](https://git.corp.adobe.com/storage/user/1451/files/ec568500-3c1e-11ea-9e08-5b7a1282c30d)

Also you can confirme it in both of the memory snapshots 

![image](https://git.corp.adobe.com/storage/user/1451/files/74d42600-3c1d-11ea-8833-de8b7bafd023)

Then when you search `LRUCache` in our code base, you will find there are several reference to `LRUCache`

![image](https://git.corp.adobe.com/storage/user/1451/files/5fabc700-3c1e-11ea-8bac-3e391b2a723f)

Not too hard, you can look what's in the LRUCache

![image](https://git.corp.adobe.com/storage/user/1451/files/b31e1500-3c1e-11ea-9eea-46aef1b2e4c6)

After adding breakpoint, you will find it is `Shrink-Ray library caching SSR response`

![image](https://git.corp.adobe.com/storage/user/1451/files/232c9b00-3c1f-11ea-83b5-1415787dc2dc)

Exciting, right? You find the memory leak, later you were sad again. Because it is just up to 128MB.

According to the library [WIKI](https://www.npmjs.com/package/shrink-ray-current#cachesize):

```
In addition, if a response contains an ETag, shrink-ray-current 
will cache the compressed result for later requests
...
The default cacheSize is 128mb
```

You can confirm in the snapshot

![image](https://git.corp.adobe.com/storage/user/1451/files/b796fd80-3c1f-11ea-8d9f-3d911164c0ce)

See the fix from [PR](https://git.corp.adobe.com/Adobe-Anyware/unified-checkout/pull/1932) to disable the caching in Shrink-Ray 

## Memory usage jump 

Do you remember there is a jump in the chart?

![image](https://git.corp.adobe.com/storage/user/1451/files/91259200-3c20-11ea-9f05-ed28c9dc814c)

After several running to capture the snapshot around that time window, I found another object was growing

![image](https://git.corp.adobe.com/storage/user/1451/files/e5307680-3c20-11ea-9b41-f8a89faeb72c)

Look at the content of the object and who is holding the reference to it

![image](https://git.corp.adobe.com/storage/user/1451/files/32ace380-3c21-11ea-8062-900384039eba)

Then you can find the code soon

```javascript
/**
 * Set data to digitalData layer and send to site catalyst
 */
export function reportEvent( storeContext, name, data ){
  try {
    // Put the event into queue when the reporter is not ready
    if( !isReadyToReportEvent()){
      eventQueue.push({ storeContext, name, data });
      return;
    }
}
```

Then you can confim the memory leak soon after calculation based on Splunk query.

![image](https://git.corp.adobe.com/storage/user/1451/files/caaacd00-3c21-11ea-97ba-f960d97f4445)

See the [PR](https://git.corp.adobe.com/Adobe-Anyware/unified-checkout/pull/1935) to fix.

## Summary

How to find the memory leak?

1. Simulate the PROD using `docker`
2. Use `process.memoryUsage` and automation to get a memory usage picture.
3. Use memory snapshot to investigate the object
	- The large object which has large retained size.
	- Comparing the snapshots to see what's always growing.

We are lucky, because I ran into the failed automation case. Then we can catch the `SERVICE_ERROR` memory leak.

# Other memory leak

Memory leak in global object is the most common case. There are another kind of memory leak: closure.

```javascript
var theThing = null;

var replaceThing = function () {
  var originalThing = theThing;

  // closure to hold a reference to global object
  function unused() {
    if( originalThing ){
      console.log("Hello World!");
    }
  };

  theThing = {
    longStr: new Array(1000000).join('*')
  };
};

setInterval(replaceThing, 1000);
```

But I don't think we worte code like this: Declaring a closure but don't use it at all. Maybe we have a similar case like this. Let's find it in the future.

# Current status

Look at the memory graph firstly.

![image](https://git.corp.adobe.com/storage/user/1451/files/f45ef680-3c17-11ea-8e37-78d15836e5b9)

As the server can run logger than before, then we can find more cache items. For ROSe cache, one host is caching more than 6K items. And the size is `346MB`. So we still need `lru-cache`.

![image](https://git.corp.adobe.com/storage/user/1451/files/cb8a3180-3c16-11ea-8ce5-5442f4dbefea)

# Final

Thanks to QE team to have the awsome automation :)

# Useful links

[Memory Management and Garbage Collection in JavaScript](https://dzone.com/articles/memory-management-and-garbage-collection-in-javasc)

[Memory Terminology](https://developers.google.com/web/tools/chrome-devtools/memory-problems/memory-101#object_sizes)

[NodeJs internals: V8 & garbage collector
](https://medium.com/voodoo-engineering/nodejs-internals-v8-garbage-collector-a6eca82540ec)

