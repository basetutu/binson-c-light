[<img src="http://quickcheck-ci.com/p/assaabloy-ppi/binson-c-light.png" alt="Build Status" width="120px">](http://quickcheck-ci.com/p/assaabloy-ppi/binson-c-light)

# binson-c-light

A light-weight C implementation of the Binson serialization format

> Binson is like JSON, but faster, binary and even simpler. See [binson.org](http://binson.org/)

Features
---------

* Written in ANSI C (C99 standard)
* Portable MCU-friendly code.
* Tiny binary size (~9K object file, clang -Os -target arm-none-eabi)
* Tested at:
  * x86_64 Linux (gcc, clang)
  * ~~ARM Cortex-M3 (with QEMU)~~
  * ~~ARM Cortex-M4 baremetal (nRF52832 SoC)~~
* Compatible with: 
  * [binson-java](https://github.com/franslundberg/binson-java)
  * [binson-java-light](https://github.com/franslundberg/binson-java-light)
* Has no 3rd party dependencies. (libc only)
* No dynamic memory allocation (in-place parsing)
* ~~Partial destination buffer allow to stream objects even larger than RAM available~~


What's new in v2
---------------

* v1 bugs fixed
* No recursion
* Simpler state machine
* Parser callback support (ready to be used as low-level API for "binson-c" project)
* Unified support for OBJECT and ARRAY blocks (is true for both API and the code)
* Both OBJECT and ARRAY top blocks now are supported automatically with zero care
* Positioning by index supported for both OBJECT and ARRAY blocks  ( see binson_parser_at() )
* Raw binson encoded block extraction and output supported now (see #8)
* Removed type checks in getters (it's up to user app now)
* Built-in binson-to-string convertion function
* Optimization frendly single-function traversal code.
* Less lines, smaller binary size
* Better unit test coverage.
* Well-commended parsing algorithm code
* Pyfuzz fuzz testing tool for random binson tree and corresponding writer/parser source code automated generation, build and run loop.
* Quickcheck models for property based testing. Integration with Quviq Quickcheck Continuous Integration server 
* Code fixed to be compatible with Clang (std=c99)
* Pass AFL fuzzing tests. (includes custom dict for binson format).
* Pass libFuzzer tests. (includes custom dict for binson format).

Status of v2
---------

Looks stable with good unit test coverage, need more eyes for review, API is still a subject of discussion.

API changes in v2:
---------------

* No implicit block level entering: for top-level nor for OBJECT items (e.g. "a":{})
* User level API returns bool (succes/failure), it's up to app to check later "error_flags" if needed.
* All parsing functions are just shortcuts to binson_parser_advance(), controlled with wide range of parameters.
* *_ensure() functions are designed to additionaly check item type (see #8)


TODO in v2:
-----------

* More code review & cleanup
* Add more unit test cases (after code coverage checks)
* Standartized API doc comments in the header/source files (doxygen)
* Try to make quick integration of this project to act as low-level API of binson-c, to proof APIs are valid and flexible enough.

TODO in v3:
-----------
* Streaming mode (parsing partial binson data while reading next chunk of input)


Writer API usage
---------

```
 uint8_t        buf[64];
 int            cnt;
 binson_writer  w;
 
 binson_writer_init( &w, buf, sizeof(buf) );
 
 // {"cid":123}
 binson_write_object_begin( &w ); 
 binson_write_name( &w, "cid" );
 binson_write_integer( &w, 123 );
 binson_write_object_end( &w ); 
 
 cnt = binson_writer_get_counter( &w );
 
 for (int i=0; i<cnt; i++)
   printf( "0x%02x ", buf[i] ); 
```
Will print: 

`
0x40 0x14 0x03 0x63 0x69 0x64 0x10 0x7b 0x41
`

Parser API usage
---------


```
  // { "a":123, "bcd":"Hello world!" }
  const uint8_t src[] = "\x40\x14\x01\x61\x10\x7b\x14\x03\x62\x63\x64\x14\x0c\x48\x65\x6c\x6c\x6f\x20\x77\x6f\x72\x6c\x64\x21\x41";
  uint8_t       buf[64];
  binson_parser p;
 
  binson_parser_init( &p, src, sizeof(src) );
 
  binson_parser_go_into( &p );  
  binson_parser_field( &p, "a" );
  printf("a: %d\n", (int)binson_parser_get_integer( &p ));
    
  binson_parser_field( &p, "bcd" );
  binson_parser_get_string_copy( &p, buf ); 
  printf("bcd: %s\n", buf);
```
Will print:

```
a: 123
bcd: Hello world!
```


Build
---------

`
$ ./build.sh
`

Resulting binary files will be placed under ./build


