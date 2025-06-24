C++ Pitfalls
==

## Memory

### std::out_ptr
1. Don't use `std::out_ptr` and the held smart pointer in the same expression.

    **ISSUE**

    The following code will produce different output depending on the compiler:

    ```cpp
    #include <memory>
    #include <print>

    int f(int** raw) {
        *raw = new int;
        return 0;
    }

    int main() {
        std::unique_ptr<int> p;
        if (f(std::out_ptr(p)) || !p) {
            std::println("p is empty");
        } else {
            std::println("p is not-empty");
        }
    }
    ```
    
    |Compiler|Compile Command|Result|Conforming Standard|
    |--|--|--|--|
    |gcc 15.0.1|g++ -std=c++23|p is not-empty|No|
    |clang 20.1.2|clang++ -std=c++23 -stdlib=libc++|p is empty|Yes|
    |msvc 19.44.35211|cl /std:c++latest|p is empty|Yes|

    According to the C++ standard, [`std::out_ptr`](https://en.cppreference.com/w/cpp/memory/out_ptr_t/out_ptr) creates a temporary object
    of type [`std::out_ptr_t`](https://en.cppreference.com/w/cpp/memory/out_ptr_t.html). When this temporary object is destroyed, it resets
    the adapted smart pointer with the result. In the above code, when `!p` is evaluated, the temporary object has not yet been destroyed,
    so the value of `p` remains empty. Therefore, the correct output should be `p is empty`.

    In the GCC standard library implementation [libstdc++](https://github.com/gcc-mirror/gcc/blob/6deab186535a5aa9f930e2db637089865d0bc4ff/libstdc%2B%2B-v3/include/bits/out_ptr.h#L121),
    it returns the address of the internal smart pointer object directly, so `p` will be set earlier. This behavior does not conform to the standard.

    This is technically a bug in libstdc++, but because it has existed for a long time and some existing code may rely on it. GCC may choose
    not to fix it to avoid introducing a breaking change.

    **ADVICE**

    Don't use `std::out_ptr` and the held smart pointer in the same expression. Depending on the compiler, the smart pointer may or may not be
    set before the expression is evaluated.
