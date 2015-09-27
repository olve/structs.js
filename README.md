struct.js
=========

A Javascript data type for creating and manipulating bytearrays

Usage
-----

```javascript
var bottle = new Struct();

var label = bottle.push(new Struct());
var content = bottle.push("B", [119, 0x41, 116, 101, 114]); //push array of unsigned 8bit integers
label.push("c", "WATER"); //push a string, strings are pushed as arrays of uint8 charcodes.

//structs have some getter-methods defined:
var bytes = bottle.array; //[87, 65, 84, 69, 82,  119, 0x41, 116, 101, 114]
var text = bottle.string; //WATERwAter
var length = bottle.byteLength; //10
var buffer = bottle.buffer; //ArrayBuffer {}

//set the byte at offset 1 of the bottle's content to be a char of value "a"
content.set("c", 1, "a"); 
console.log(bottle.string); //WATERwater

label.set("L", 1, 0); //write an unsigned long with value 0 at offset 1 of the label
console.log(bottle.array); //[87, 0, 0, 0, 0,  119, 97, 116, 101, 114]

//create a blob from our struct.
var blob = new Blob([bottle.buffer], {type: "plain/text"});
```

Format characters
-----------------

struct.js uses format characters like the [Python struct module](https://docs.python.org/2/library/struct.html#format-characters) to define types of values.
```javascript
var struct = new Struct();
struct.push("B", 3);	//push an unsigned byte with value 3
struct.push("l", 300)	//push a signed long with value 300
struct.push("c", "a")	//push the charcode of "a"
``` 

character | size | type
----------|------|------
    c     |   1  | char (string)
    b     |   1  | signed char
    B     |   1  | unsigned char
    ?     |   1  | bool
    h     |   2  | signed short
    H     |   2  | unsigned short
    l     |   4  | signed long
    L     |   4  | unsigned long
    f     |   4  | float
    d     |   8  | double float

Types are defined in Struct.prototype.TYPES

Reading and writing at offsets
-------------------------------
When one pushes a new member that is not a struct, a getter and setter for reading and writing specific offsets of the member's bytearray are bound to the member object.

```javascript
var struct = new Struct();
var someBytes = struct.push("B", [65, 66, 67, 68]);
someBytes.get("B", 0)			// read as unsigned chars from offset 0: returns 65
someBytes.get("H", 0)			// read as unsigned short from offset 0: returns 16706
someBytes.get("c", 1)			// read as char from offset 1: returns "B"
someBytes.set("H", 0, 5)		// write unsigned short of value 5 at offset 0
console.log(someBytes.array)	// [0, 5, 67, 68]
```

The getter and setter work by applying the prototype-methods of the native DataView type.
so someBytes.get("H", 0) will apply DataView.prototype.getUint16(0) to a DataView created from the member's bytearray.

the getters and setters are defined in Struct.prototype.createMember, and you can see which DataView.prototype-method is applied by looking up the format-character of your type in Struct.prototype.TYPES.

Struct.prototype.get and Struct.prototype.set do the same thing as a member's getters and setters, however the changes are applied to the total array of the Struct
