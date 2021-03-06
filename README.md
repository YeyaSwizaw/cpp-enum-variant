# cpp-enum-variant
A simple variant data type similar to boost::variant, 
designed to be easy to use, intuitive, and not to be a big pile of macros.

## Including
Using this library is as simple as including ```enum.hpp``` in your project. 
Include ```optional.hpp``` for a simple ```Optional<T>``` implementation using it, 
or ```tree.hpp``` for a proof-of-concept BST implementation.

It requires a decent C++14 compiler - I have tested it in VS2015, gcc, and clang.

## Using
Create a type like so:

```c++
using Test = venum::Enum
  ::Variant<int>
  ::Variant<char>;
```

or the less verbose

```c++
using Test = venum::EnumT<int, char>;
```

The type is made using ```std::aligned_storage``` and a single ```int``` tag, 
and so is as big as the biggest variant + an ```int```.

To construct an instance, simply call the constructor with whatever arguments. 
The variant you want should be inferred correctly from the arguments:

```c++
Test int_test(5); // produces an int variant
Test char_test('a'); // produces a char variant
```
  
If you absolutely must specify the variant you want, use ```construct<T>```:

```c++
auto test = Test::construct<int>('a'); // produces an int variant
```
  
If, in any of these cases, a valid construtor can not be found, the program will fail to compile.

To get the data out of the object, you use ```match```, 
which takes a function for each possible variant (in order) and applies the correct one:

```c++
std::string s = test.match(
  [](int i) { std::ostringstream str; str << "int: " << i; return str.str(); },
  [](char c) { std::ostringstream str; str << "char: " << c; return str.str(); }
);
```

You can also use ```apply```, which takes a single polymorphic function:

```c++
test.apply([](auto& value) { std::cout << value << std::endl; });
```

## Error Handling
The above is fine if you are using simple types that can never throw, 
however a throwing copy/move introduces some extra complexity that needs to be handled.

If a copy (or move) assignment throws, the variant gets set to an invalid state.
The variant itself is still usable, or assignable to, however the object inside is inaccessible.
Any attempt to use the object inside an invalid variant will throw an exception stating what caused the invalid state:

```c++
nasty_variant = NastyType(7); // This copy throws, invalidating the variant.
try {
  nasty_variant.apply([](auto& a) { std::cout << a << std::endl; }); // This call will throw.
} catch(venum::InvalidVariantError& e) {
  std::cout << e.what(); // This will explain what invalidated the variant.
}
```

In addition, ```match``` can take an optional final callback for when the variant is invalid,
which takes an object of this exception type:

```c++
nasty_variant.match(
  // other cases
  [](const venum::InvalidVariantError& e) { /* handle error case */ }
);
```

Finally, there is a ```valid``` function and boolean conversion for convenience:

```c++
if(nasty_variant) {
  // it's valid
} else {
  // it's not valid
}
```
