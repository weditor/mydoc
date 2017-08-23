
# C++ 调试

以下代码会自动打印每一层函数调用流程，以缩进方式展示。
```cpp
#include <iostream>
#include <stdio.h>
#include <execinfo.h>
#include <stdlib.h>
#include <string.h>
#include <cxxabi.h>

using namespace std;

extern "C" {
    static __thread int indent=0;
    static __thread char func_name[1024];
    const char* __attribute__((__no_instrument_function__)) demangle(const char* symbol)
    {
        size_t size;
        int status;
        char* demangled;
        //first, try to demangle a c++ name
        if (1 == sscanf(symbol, "%*[^(]%*[^_]%127[^)+]", func_name)) {
            if (NULL != (demangled = abi::__cxa_demangle(func_name, NULL, &size, &status))) {
              strncpy(func_name, demangled, sizeof(func_name)-1);
              free(demangled);
              return func_name;
            }
        }
        //if that didn't work, try to get a regular c symbol
        if (1 == sscanf(symbol, "%127s", func_name)) {
            return func_name;
        }
    
        //if all else fails, just return the symbol
        return symbol;
    }

    void __attribute__((__no_instrument_function__)) __cyg_profile_func_enter(void *this_func, void *call_site)
    {
        if (call_site == (void *)__cyg_profile_func_enter) {
            return;
        }
        int size = 2;
        void * array[2];
        int stack_num = backtrace(array, size);
        char ** stacktrace = backtrace_symbols(array, stack_num);
        if (stack_num >= 2) {
            ++indent;
            for (int i = 0; i < indent; ++i){
                printf("    ");
            }
            printf("enter->%s\n", demangle(stacktrace[1]));
        }
        free(stacktrace);
    }

    void __attribute__((__no_instrument_function__)) __cyg_profile_func_exit(void *this_func, void *call_site)
    {
        if (call_site == (void *)__cyg_profile_func_exit) {
            return;
        }
        int size = 2;
        void * array[2];
        int stack_num = backtrace(array, size);
        char ** stacktrace = backtrace_symbols(array, stack_num);
        if (stack_num >= 2) {
            for (int i = 0; i < indent; ++i){
                printf("    ");
            }
            printf("exit <-%s\n", demangle(stacktrace[1]));
            --indent;
        }
        free(stacktrace);
    }


    void __attribute__((__no_instrument_function__)) print_stacktrace()
    {
        int size = 16;
        void * array[16];
        int stack_num = backtrace(array, size);
        char ** stacktrace = backtrace_symbols(array, stack_num);
        for (int i = 0; i < stack_num; ++i)
        {
            printf("%s\n", stacktrace[i]);
        }
        free(stacktrace);
    }
}

int add(int a, int b) {
    return a+b;
}

int add3(int a, int b, int c) {
    int d = add(a,b);
    return add(d, c);
}


int main() {
    cout<< add3(1, 2, 3)<<endl;
    cout<<"Hello world"<<endl;
    return 0;
}

```

make命令如下
```shell
g++ -o main -finstrument-functions main.cpp -O0 -rdynamic
```

执行后输出如下
```shell
    enter->./main()
        enter->./main()
        exit <-./main()
    exit <-./main()
    enter->./main(main+0x1a)
        enter->add3(int, int, int)
            enter->add(int, int)
            exit <-add(int, int)
            enter->add(int, int)
            exit <-add(int, int)
        exit <-add3(int, int, int)
6
Hello world
    exit <-./main(main+0x79)
```

