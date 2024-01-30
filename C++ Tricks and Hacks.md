# C++ Tricks and Hacks

### Pass a C++ string as C char* pointer to legacy API:

While you can use a C++ `std::string` directly, you must resize it to the require buffer size using the string::resize() method.
A more robust and elegant method is to use a C++ `std::vector<char>` to avoid explicitly resizing and potential side-effects when the legacy code writes into the underline structures of the `std::vector`.
You can also mark it as `static` to avoid repeated allocations/deallocations if this runs in a loop:

``` C
  // Assume a legacy API function: str_legacy_api_function(char* buffer, size_t buffer_size)
  size_t buffer_size = 10000;
  static auto buff = std::make_unique<std::vector<char>>(buffer_size);
  auto buff_ptr    = buff.get()->data();
  str_legacy_api_function(buff_ptr, buffer_size);
```

