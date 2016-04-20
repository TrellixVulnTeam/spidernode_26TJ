Representing V8 Values
===
In V8, the `Value` class is an opaque object that can be either 32 or 64 bits, depending on how V8 decides to allocate it.  V8 SMI's are represented as 32-bit values.  The size of the C++ Value object is always 1, since it's an empty class.

In SpiderShim, we have changed the `Value` class to have the same binary layout as `JS::Value`, so that in places that we deal with a `v8::Value`, we can just `reinterpret_cast` to `JS::Value`.  This approach seems to be working for many different value types.  It's possible that eventually we'd move to a more complicated mapping.

Note that there is a C++ class hierarchy that maps to the different types of values too.  For example, a number can be represented as a `v8::Value` or a `v8::Number`.  The picture quickly gets complicated with `v8::Integer`, `v8::Int32`, `v8::Uint32`, etc.  The V8 API provides a set of cast methods like `Integer::Cast()`.  In debug builds, for most basic types these methods check that the correct type of object has been passed in.  For example, `Integer::Cast()` checks the input object type by calling `JS::Value::isNumber()`.  More specific cast functions such as `Uint32::Cast()` may also check that the passed object falls into the supported value range (for example between `0` and `UINT32_MAX`.)  In non-debug builds, these functions happily `static_cast` the input to the requested type.