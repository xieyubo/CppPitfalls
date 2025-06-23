C++ Pitfalls
==

## Memory

### std::out_ptr
1. Don't use `std::out_ptr` and the holded smart pointer in ths same expression.

    **ISSUE**

    The following code will output different on different compiler:

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
    |clang 20.1.2|clang++ -std=c++23 -stdlib=libc++ c.cpp|p is empty|Yes|
    |msvc 19.44.35211|cl /std:c++latest|p is empty|Yes|

    In c++ standard, [`std::out_ptr`](https://en.cppreference.com/w/cpp/memory/out_ptr_t/out_ptr) will create a temporary object
    which has type [`std::out_ptr_t`](https://en.cppreference.com/w/cpp/memory/out_ptr_t.html). When this temporary object is destroyed,
    the adapted smart pointer object will be rested with the result. In the above code, when `!p` is evaluated, the temporary object
    hasn't been destoryed, so the value of `p` is still empty. The output of the above code should be `p is empty`.

    In gcc standard library implementation [libstdc++](https://github.com/gcc-mirror/gcc/blob/6deab186535a5aa9f930e2db637089865d0bc4ff/libstdc%2B%2B-v3/include/bits/out_ptr.h#L121),
    it returns the address of the internal smart pointer object directly, so `p` will be set eariler. This behavior isn't conforming the standard.

    This is a bug of libstdc++ actually, but because this issue has existed for a long time and some existing codes might relay on this issue already.
    gcc may not fix it to avoid introducing a breaking change.

    **ADVICE**

    Don't use `std::out_ptr` and the holded smart pointer in the same expression. Depends on different compiler, the smart pointer
    may be set or not be set before the expression evaluated.
