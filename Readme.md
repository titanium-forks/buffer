# Buffer

Pure JavaScript is Unicode friendly but not nice to binary data. When dealing with TCP streams or the file system, it's necessary to handle octet streams. Node has several strategies for manipulating, creating, and consuming octet streams.

Raw data is stored in instances of the Buffer class. A Buffer is similar to an array of integers but corresponds to a raw memory allocation outside the heap. A Buffer cannot be resized.

Converting between Buffers and JavaScript string objects requires an explicit encoding method. Here are the different string encodings.

- `ascii` - for 7 bit ASCII data only. This encoding method is very fast, and will strip the high bit if set. Note that this encoding converts a null character ('\0' or '\u0000') into 0x20 (character code of a space). If you want to convert a null character into 0x00, you should use 'utf8'.
- `utf8`  - Multibyte encoded Unicode characters. Many web pages and other document formats use UTF-8.
- `utf16le` - 2 or 4 bytes, little endian encoded Unicode characters. Surrogate pairs (U+10000 to U+10FFFF) are supported.
- `ucs2` - Alias of 'utf16le'.
- `base64` - Base64 string encoding.
- `binary` - A way of encoding raw binary data into strings by using only the first 8 bits of each character. This encoding method is deprecated and should be avoided in favor of Buffer objects where possible. This encoding will be removed in future versions of Node.
- `hex` - Encode each byte as two hexadecimal characters.

The Buffer class is a type for dealing with binary data directly. It can be constructed in a variety of ways.

## Install

    $ tipm install tipm/buffer

## API

### new Buffer(size)
- size Number

Allocates a new buffer of size octets.

### new Buffer(array)
- array Array

Allocates a new buffer using an array of octets.

### new Buffer(str, [encoding])
- str String - string to encode.
- encoding String - encoding to use, Optional.

Allocates a new buffer containing the given str. encoding defaults to `utf8`.

### buf.write(string, [offset], [length], [encoding])
- string String - data to be written to buffer
- offset Number, Optional, Default: 0
- length Number, Optional, Default: `buffer.length - offset`
- encoding String, Optional, Default: `utf8`

Writes string to the buffer at offset using the given encoding. offset defaults to 0, encoding defaults to `utf8`. length is the number of bytes to write. Returns number of octets written. If buffer did not contain enough space to fit the entire string, it will write a partial amount of the string. length defaults to buffer.length - offset. The method will not write partial characters.

```js
buf = new Buffer(256);
len = buf.write('\u00bd + \u00bc = \u00be', 0);
console.log(len + " bytes: " + buf.toString('utf8', 0, len));
```

The number of characters written (which may be different than the number of bytes written) is set in `Buffer._charsWritten` and will be overwritten the next time `buf.write()` is called.

### buf.toString([encoding], [start], [end])
- encoding String, Optional, Default: 'utf8'
- start Number, Optional, Default: 0
- end Number, Optional, Default: buffer.length

Decodes and returns a string from buffer data encoded with encoding (defaults to `utf8`) beginning at start (defaults to `0`) and ending at end (defaults to `buffer.length`).

See `buffer.write()` example, above.

### buf[index]
Get and set the octet at index. The values refer to individual bytes, so the legal range is between `0x00` and `0xFF` hex or `0` and `255`.

Example: copy an ASCII string into a buffer, one byte at a time:

```js
str = "node.js";
buf = new Buffer(str.length);

for (var i = 0; i < str.length ; i++) {
  buf[i] = str.charCodeAt(i);
}

console.log(buf);
```

### Buffer.isBuffer(obj)
- obj Object
- Return: Boolean
- Tests if obj is a Buffer.

### Buffer.byteLength(string, [encoding])
- string String
- encoding String, Optional, Default: `utf8`
- Return: Number

Gives the actual byte length of a string. encoding defaults to `utf8`. This is not the same as `String.prototype.length` since that returns the number of characters in a string.

Example:

```js
str = '\u00bd + \u00bc = \u00be';

console.log(str + ": " + str.length + " characters, " +
  Buffer.byteLength(str, 'utf8') + " bytes");

// ½ + ¼ = ¾: 9 characters, 12 bytes
```

### Buffer.concat(list, [totalLength])
- list Array List of Buffer objects to concat
- totalLength Number Total length of the buffers when concatenated
- Returns a buffer which is the result of concatenating all the buffers in the list together.

If the list has no items, or if the totalLength is `0`, then it returns a zero-length buffer.

If the list has exactly one item, then the first item of the list is returned.

If the list has more than one item, then a new Buffer is created.

If totalLength is not provided, it is read from the buffers in the list. However, this adds an additional loop to the function, so it is faster to provide the length explicitly.

### buf.length
Number
The size of the buffer in bytes. Note that this is not necessarily the size of the contents. length refers to the amount of memory allocated for the buffer object. It does not change when the contents of the buffer are changed.

```js
buf = new Buffer(1234);

console.log(buf.length);
buf.write("some string", 0, "ascii");
console.log(buf.length);

// 1234
// 1234
```

### buf.copy(targetBuffer, [targetStart], [sourceStart], [sourceEnd])
- targetBuffer Buffer object - Buffer to copy into
- targetStart Number, Optional, Default: `0`
- sourceStart Number, Optional, Default: `0`
- sourceEnd Number, Optional, Default: `buffer.length`

Does copy between buffers. The source and target regions can be overlapped. targetStart and sourceStart default to `0`. sourceEnd defaults to `buffer.length`.

Example: build two Buffers, then copy buf1 from byte 16 through byte 19 into buf2, starting at the 8th byte in buf2.

```js
buf1 = new Buffer(26);
buf2 = new Buffer(26);

for (var i = 0 ; i < 26 ; i++) {
  buf1[i] = i + 97; // 97 is ASCII a
  buf2[i] = 33; // ASCII !
}

buf1.copy(buf2, 8, 16, 20);
console.log(buf2.toString('ascii', 0, 25));

// !!!!!!!!qrst!!!!!!!!!!!!!
```

### buf.slice([start], [end])
- start Number, Optional, Default: 0
- end Number, Optional, Default: buffer.length

Returns a new buffer which references the same memory as the old, but offset and cropped by the start (defaults to `0`) and end (defaults to `buffer.length`) indexes.

Modifying the new buffer slice will modify memory in the original buffer!

Example: build a Buffer with the ASCII alphabet, take a slice, then modify one byte from the original Buffer.

```js
var buf1 = new Buffer(26);

for (var i = 0 ; i < 26 ; i++) {
  buf1[i] = i + 97; // 97 is ASCII a
}

var buf2 = buf1.slice(0, 3);
console.log(buf2.toString('ascii', 0, buf2.length));
buf1[0] = 33;
console.log(buf2.toString('ascii', 0, buf2.length));

// abc
// !bc
```

### buf.readUInt8(offset, [noAssert])
- offset Number
- noAssert Boolean, Optional, Default: false
- Return: Number

Reads an unsigned 8 bit integer from the buffer at the specified offset.

Set `noAssert` to true to skip validation of offset. This means that offset may be beyond the end of the buffer. Defaults to `false`.

Example:

```js
var buf = new Buffer(4);

buf[0] = 0x3;
buf[1] = 0x4;
buf[2] = 0x23;
buf[3] = 0x42;

for (ii = 0; ii < buf.length; ii++) {
  console.log(buf.readUInt8(ii));
}

// 0x3
// 0x4
// 0x23
// 0x42
```

### buf.readUInt16LE(offset, [noAssert])
### buf.readUInt16BE(offset, [noAssert])
- offset Number
- noAssert Boolean, Optional, Default: false
- Return: Number

Reads an unsigned 16 bit integer from the buffer at the specified offset with specified endian format.

Set noAssert to true to skip validation of offset. This means that offset may be beyond the end of the buffer. Defaults to `false`.

Example:

```js
var buf = new Buffer(4);

buf[0] = 0x3;
buf[1] = 0x4;
buf[2] = 0x23;
buf[3] = 0x42;

console.log(buf.readUInt16BE(0));
console.log(buf.readUInt16LE(0));
console.log(buf.readUInt16BE(1));
console.log(buf.readUInt16LE(1));
console.log(buf.readUInt16BE(2));
console.log(buf.readUInt16LE(2));

// 0x0304
// 0x0403
// 0x0423
// 0x2304
// 0x2342
// 0x4223
```

### buf.readUInt32LE(offset, [noAssert])
### buf.readUInt32BE(offset, [noAssert])
- offset Number
- noAssert Boolean, Optional, Default: false
- Return: Number

Reads an unsigned 32 bit integer from the buffer at the specified offset with specified endian format.

Set noAssert to true to skip validation of offset. This means that offset may be beyond the end of the buffer. Defaults to `false`.

Example:

```js
var buf = new Buffer(4);

buf[0] = 0x3;
buf[1] = 0x4;
buf[2] = 0x23;
buf[3] = 0x42;

console.log(buf.readUInt32BE(0));
console.log(buf.readUInt32LE(0));

// 0x03042342
// 0x42230403
```

### buf.readInt8(offset, [noAssert])
- offset Number
- noAssert Boolean, Optional, Default: false
- Return: Number

Reads a signed 8 bit integer from the buffer at the specified offset.

Set noAssert to true to skip validation of offset. This means that offset may be beyond the end of the buffer. Defaults to `false`.

Works as buffer.readUInt8, except buffer contents are treated as two's complement signed values.

### buf.readInt16LE(offset, [noAssert])
### buf.readInt16BE(offset, [noAssert])
- offset Number
- noAssert Boolean, Optional, Default: false
- Return: Number

Reads a signed 16 bit integer from the buffer at the specified offset with specified endian format.

Set noAssert to true to skip validation of offset. This means that offset may be beyond the end of the buffer. Defaults to `false`.

Works as `buffer.readUInt16*`, except buffer contents are treated as two's complement signed values.

### buf.readInt32LE(offset, [noAssert])
### buf.readInt32BE(offset, [noAssert])
- offset Number
- noAssert Boolean, Optional, Default: false
- Return: Number

Reads a signed 32 bit integer from the buffer at the specified offset with specified endian format.

Set noAssert to true to skip validation of offset. This means that offset may be beyond the end of the buffer. Defaults to `false`.

Works as `buffer.readUInt32*`, except buffer contents are treated as two's complement signed values.

### buf.readFloatLE(offset, [noAssert])#
### buf.readFloatBE(offset, [noAssert])#
- offset Number
- noAssert Boolean, Optional, Default: false
- Return: Number

Reads a 32 bit float from the buffer at the specified offset with specified endian format.

Set noAssert to true to skip validation of offset. This means that offset may be beyond the end of the buffer. Defaults to `false`.

Example:

```js
var buf = new Buffer(4);

buf[0] = 0x00;
buf[1] = 0x00;
buf[2] = 0x80;
buf[3] = 0x3f;

console.log(buf.readFloatLE(0));

// 0x01
```

### buf.readDoubleLE(offset, [noAssert])
### buf.readDoubleBE(offset, [noAssert])
- offset Number
- noAssert Boolean, Optional, Default: false
- Return: Number

Reads a 64 bit double from the buffer at the specified offset with specified endian format.

Set noAssert to true to skip validation of offset. This means that offset may be beyond the end of the buffer. Defaults to false.

Example:

```js
var buf = new Buffer(8);

buf[0] = 0x55;
buf[1] = 0x55;
buf[2] = 0x55;
buf[3] = 0x55;
buf[4] = 0x55;
buf[5] = 0x55;
buf[6] = 0xd5;
buf[7] = 0x3f;

console.log(buf.readDoubleLE(0));

// 0.3333333333333333
```

### buf.writeUInt8(value, offset, [noAssert])
- value Number
- offset Number
- noAssert Boolean, Optional, Default: false

Writes value to the buffer at the specified offset. Note, value must be a valid unsigned 8 bit integer.

Set noAssert to true to skip validation of value and offset. This means that value may be too large for the specific function and offset may be beyond the end of the buffer leading to the values being silently dropped. This should not be used unless you are certain of correctness. Defaults to `false`.

Example:

```js
var buf = new Buffer(4);
buf.writeUInt8(0x3, 0);
buf.writeUInt8(0x4, 1);
buf.writeUInt8(0x23, 2);
buf.writeUInt8(0x42, 3);

console.log(buf);

// <Buffer 03 04 23 42>
```

### buf.writeUInt16LE(value, offset, [noAssert])
### buf.writeUInt16BE(value, offset, [noAssert])
- value Number
- offset Number
- noAssert Boolean, Optional, Default: false

Writes value to the buffer at the specified offset with specified endian format. Note, value must be a valid unsigned 16 bit integer.

Set noAssert to true to skip validation of value and offset. This means that value may be too large for the specific function and offset may be beyond the end of the buffer leading to the values being silently dropped. This should not be used unless you are certain of correctness. Defaults to `false`.

Example:

```js
var buf = new Buffer(4);
buf.writeUInt16BE(0xdead, 0);
buf.writeUInt16BE(0xbeef, 2);

console.log(buf);

buf.writeUInt16LE(0xdead, 0);
buf.writeUInt16LE(0xbeef, 2);

console.log(buf);

// <Buffer de ad be ef>
// <Buffer ad de ef be>
```

### buf.writeUInt32LE(value, offset, [noAssert])
### buf.writeUInt32BE(value, offset, [noAssert])
- value Number
- offset Number
- noAssert Boolean, Optional, Default: false

Writes value to the buffer at the specified offset with specified endian format. Note, value must be a valid unsigned 32 bit integer.

Set noAssert to true to skip validation of value and offset. This means that value may be too large for the specific function and offset may be beyond the end of the buffer leading to the values being silently dropped. This should not be used unless you are certain of correctness. Defaults to `false`.

Example:

```js
var buf = new Buffer(4);
buf.writeUInt32BE(0xfeedface, 0);

console.log(buf);

buf.writeUInt32LE(0xfeedface, 0);

console.log(buf);

// <Buffer fe ed fa ce>
// <Buffer ce fa ed fe>
```

### buf.writeInt8(value, offset, [noAssert])
- value Number
- offset Number
- noAssert Boolean, Optional, Default: false

Writes value to the buffer at the specified offset. Note, value must be a valid signed 8 bit integer.

Set noAssert to true to skip validation of value and offset. This means that value may be too large for the specific function and offset may be beyond the end of the buffer leading to the values being silently dropped. This should not be used unless you are certain of correctness. Defaults to `false`.

Works as `buffer.writeUInt8`, except value is written out as a two's complement signed integer into buffer.

### buf.writeInt16LE(value, offset, [noAssert])
### buf.writeInt16BE(value, offset, [noAssert])
- value Number
- offset Number
- noAssert Boolean, Optional, Default: false

Writes value to the buffer at the specified offset with specified endian format. Note, value must be a valid signed 16 bit integer.

Set noAssert to true to skip validation of value and offset. This means that value may be too large for the specific function and offset may be beyond the end of the buffer leading to the values being silently dropped. This should not be used unless you are certain of correctness. Defaults to `false`.

Works as `buffer.writeUInt16*`, except value is written out as a two's complement signed integer into buffer.

### buf.writeInt32LE(value, offset, [noAssert])
### buf.writeInt32BE(value, offset, [noAssert])
- value Number
- offset Number
- noAssert Boolean, Optional, Default: false

Writes value to the buffer at the specified offset with specified endian format. Note, value must be a valid signed 32 bit integer.

Set noAssert to true to skip validation of value and offset. This means that value may be too large for the specific function and offset may be beyond the end of the buffer leading to the values being silently dropped. This should not be used unless you are certain of correctness. Defaults to `false`.

Works as `buffer.writeUInt32*`, except value is written out as a two's complement signed integer into buffer.

### buf.writeFloatLE(value, offset, [noAssert])
### buf.writeFloatBE(value, offset, [noAssert])
- value Number
- offset Number
- noAssert Boolean, Optional, Default: false

Writes value to the buffer at the specified offset with specified endian format. Note, value must be a valid 32 bit float.

Set noAssert to true to skip validation of value and offset. This means that value may be too large for the specific function and offset may be beyond the end of the buffer leading to the values being silently dropped. This should not be used unless you are certain of correctness. Defaults to `false`.

Example:

```js
var buf = new Buffer(4);
buf.writeFloatBE(0xcafebabe, 0);

console.log(buf);

buf.writeFloatLE(0xcafebabe, 0);

console.log(buf);

// <Buffer 4f 4a fe bb>
// <Buffer bb fe 4a 4f>
```

### buf.writeDoubleLE(value, offset, [noAssert])
### buf.writeDoubleBE(value, offset, [noAssert])
- value Number
- offset Number
- noAssert Boolean, Optional, Default: false

Writes value to the buffer at the specified offset with specified endian format. Note, value must be a valid 64 bit double.

Set noAssert to true to skip validation of value and offset. This means that value may be too large for the specific function and offset may be beyond the end of the buffer leading to the values being silently dropped. This should not be used unless you are certain of correctness. Defaults to `false`.

Example:

```js
var buf = new Buffer(8);
buf.writeDoubleBE(0xdeadbeefcafebabe, 0);

console.log(buf);

buf.writeDoubleLE(0xdeadbeefcafebabe, 0);

console.log(buf);

// <Buffer 43 eb d5 b7 dd f9 5f d7>
// <Buffer d7 5f f9 dd b7 d5 eb 43>
```

### buf.fill(value, [offset], [end])
- value
- offset Number, Optional
- end Number, Optional

Fills the buffer with the specified value. If the offset (defaults to `0`) and end (defaults to `buffer.length`) are not given it will fill the entire buffer.

```js
var b = new Buffer(50);
b.fill("h");
```

### toArray ([start, end])

### readUInt16 (offset, [isBigEndian, noAssert])

### readInt16 (offset, [isBigEndian, noAssert])

### readUInt32 (offset, [isBigEndian, noAssert])

### readInt32 (offset, [isBigEndian, noAssert])

### readFloat (offset, [isBigEndian, noAssert])

### readDouble (offset, [isBigEndian, noAssert])

### writeUInt16 (value, offset, [isBigEndian, noAssert])

### writeInt16 (value, offset, [isBigEndian, noAssert])

### writeUInt32 (value, offset, [isBigEndian, noAssert])

### writeInt32 (value, offset, [isBigEndian, noAssert])

### writeFloat (value, offset, [isBigEndian, noAssert])

### writeDouble (value, offset, [isBigEndian, noAssert])

## License

(The MIT License)

Copyright (c) 2013 Christian Sullivan &lt;cs@euforic.co&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.