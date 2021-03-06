# libparse

### Description

**libparse** is a python library for generation file parsers. 

It is not intended for performance parsing, but for generating analysis tools. 

### Basic Structures

**libparse** is based on two basic structures to enable parsing of corrupted or tampered files: **BinData** and **ByteStream**

#### BinData
The **BinData** structure is used to represent any actual data in the file. It has the property of always being a valid structure, even if the input data provided in it's initialization is invalid.

#### ByteStream
The **ByteStream** structure is the representation of a file stream, but in memory. It allows for reading bytes and seeking to a given offset. The diffencence between this structure and a traditional stream is that a seek will always be successful, even for invalid offsets, and a read will also always be successful. In the situation where the current offset is invalid, or the amount of bytes read is bigger than the stream can provide, the ByteStream will be marked as exhausted, An exhausted ByteStream always represents that the input file is corrupted.

### Parse Structures
**libparse** is based on three parse structures for automatic parsing: **Entry**, **EntryList** and **EntryTable**.

####Entry
The **Entry** structure is the basic representation of an object in the file. It is a dynamic generated class, initialized with the `Entry.create(name,attr_name)` API. 

######Example:
```python
BasicEntry = Entry.create('BasicEntry',[
        ['field1', 2, BinData],
        ['field2', 4, BinData],
])
```

In this example we are creating an Entry named `BasicEntry` with two fields. Each field has a `name`, `size`, and `type` property. 

The `name` property can be any valid string, and can be later accessed with direct reference using that name (E.g. `BasicEntry.field1`)

The `type` field can be a **BinData** subclass or any of the parse structures (**Entry**,**EntryList** and **EntryTable**). In the first case, the parser will read `size` bytes from the ByteStream.  In the second case, the parser will recursively parse the provided parse structure.

The `size` property can be either a constant, a name field or a custom function in the format `size_func(self,bytestream)`.

######Example:
```python
def get_str_size(bstream):
    start = bstream.offset
    length = 0

    r_byte = bstream.read(1)
    while len(r_byte) == 1 and r_byte[0] != 0x0:
        length+=1
        r_byte = bstream.read(1)

    bstream.offset = start

    return length

BasicTest = Entry.create('BasicTest',[
        ['field1', 2, BinData],
        ['field2', 4, BinData],
        ])

ClassFieldSizeTest = Entry.create('ClassFieldSizeTest',[
        ['field1', 4, BinData],
        ['field2', 'field1', BinData],
        ])

DinamicSizeTest = Entry.create('DinamicSizeTest',[
        ['attr1', get_str_size, BinData],
	])
```

#### EntryList
An **EntryList** structure represents a sequential list of **Entry** objects. It is used when you have consecutive structures of the same type in the target file. 

######Example:
```python
list_size = 10
list_offset = 0x1000

basic_list = EntryList(bytestream, BasicEntry, list_size, list_offset)
```

In the example above the `basic_list` object will try to parse 10 entries of the type `BasicEntry` starting at offset `0x1000` of the ByteStream.

####EntryTable
An **EntryTable** structure represents a group of structures of the same type, but not necessarily sequential. It parses objects based on an **EntryList** and requires one offset per entry.

######Example:
```python
basic_table = EntryTable(bytestream, BasicEntry, basic_list,´field1´)
```

In the example above the `basic_table` object will iterate through the `basic_list` object and use the `field1` value of each item in the list as offset to parse a `BasicEntry` entry.

### Print Structure
**libparse** provides a **Printer** class to allow simple visualization of the parsed data. This class is able to print all the three arse structures. 

######Example:
```python
p = Printer()

print p.parse(basic_list)
```

**Printer** can be extended to provide detailed printing. The **BinData** objects are printed by using a call `__str__(self)`, so by  creating a child class that overwrites that method it is possible to change the parser output.

the `bindata` module provides three convenience classes for printing: **BinInt**, **BinHex** and **BinStr**.

###Full Example
For a full detailled example of **libparse** usage, check **dexlib** at https://github.com/rchiossi/dexlib
