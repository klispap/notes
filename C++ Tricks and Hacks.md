# C++ Tricks and Hacks

### Pass a C++ string as C char* pointer to legacy API

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



### Update raw for-loops with the more efficient & safe `std::foreach`

While we can easily find ourselves coding with the overly familiar `for(int i=0; i<10; i++) {}` style, C++ offers `std::for_each()` since C++ 11 that allows for loops (pun intended) that minimize the chances to make mistakes in the conditions: 

Raw Loop:
``` cpp
 // Assume msg is a complex payload structure and we want to initialize an array in field hash
 for(unsigned int i = 0; i < sizeof msg.cmplDate; i++) {
        msg.hash[i] = '\0';
    }
```

std::for_each Loop:
``` cpp
   // Assume msg is a complex payload structure and we want to initialize an array in field hash
    std::for_each(std::begin(msg.hash), std::end(msg.hash), [](unsigned int i) {
            i = '\0';
    });
```


## Replace C-style function pointers with template argument

When you want to define a function that can accept a function pointer as an argument, you can replace it with a template argument.
Using a function pointer is an inferior solution, bacause only the function pointer can be passed as an argument, while the templates offer the caller more flexibility because they can take more advanced functors, such as lambdas with some captured state and it typically has worse performance than the template parameter solution.

- A usual function pointer:

  ``` cpp
    void f(void (*callback_fn)());
  ```

- Replace it with a template argument:
  
  ``` cpp
    template<class CallbackFunction>
    void f(CallbackFunction callback_fn);
  ```



## Track memory leaks with Valgrind

One of the major drawbacks of developing an application in C/C++ is that a program can be prune to memory leaks. This behavior cannot be automatically tested during coding, however there are tools that allow checks to detect unsought memory behavior like leaks. [Valgrind](https://valgrind.org/) is a powerful tool that can help you track and debug memory leaks. It uses a CPU emulator to run your application while collecting metrics on memory operations. On exit, the results can be used to detect memory leaks.

- Assumimg you have installed Valgrind and its dependencies, you can execute the following:

  ``` shell
  G_SLICE=always-malloc G_DEBUG=gc-friendly  valgrind -v --tool=memcheck --num-callers=40 --log-file=valgrind.log --leak-check=full --show-leak-kinds=all --track-origins=yes $(which <BINARY>) <ARGS>
  ```
- Once your application is running, use it as always. Once you reached the fault, you exit gracefully or you send a termination signal the report will be generated at `./valgrind.log`

- Open the generated report with your favorite text editor and go to the `HEAP SUMMARY:` section:

  ```
  ==2140709==
  ==2140709==
  ==2140709== HEAP SUMMARY:
  ==2140709==     in use at exit: 12,705,771 bytes in 10,787 blocks
  ==2140709==   total heap usage: 1,275,087 allocs, 1,264,300 frees, 580,792,801 bytes allocated
  ==2140709==
  ==2140709== Searching for pointers to 10,787 not-freed blocks
  ==2140709== Checked 52,368,120 bytes
  ==2140709==
  ...
  ...
  ```

  - The report will show you the traceback that caused each non-freed block to appear. You can then use your favorite debugger to analyse each specific problematic function more.

  **IMPORTANT:** Valgrind has its limitations that include slow speed, issues with CUDA and other accelerators; however, is a powerful tool to get you started!!!!

  


