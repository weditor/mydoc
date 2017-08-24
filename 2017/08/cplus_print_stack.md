
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

## 技术细节

### 1. gcc函数注入选项instrument-functions

g++编译时增加-finstrument-functions , 就会在项目中每个函数执行入口增加调用__cyg_profile_func_enter , 函数退出时执行__cyg_profile_func_exit, 但是增加了 __attribute__((__no_instrument_function__)) 的函数不会被注入这两个函数， 函数原型：

```cpp
void __cyg_profile_func_enter(void *this_func, void *call_site)
void __cyg_profile_func_exit(void *this_func, void *call_site)
```

### 2. backtrace

头文件:
```cpp
#include <execinfo.h>
```

此头文件包含三个函数：
```cpp
// 将最多__size层堆栈指针存放到__array数组。
int backtrace(void **__array, int __size);

// 将上一个借口返回的array转换为文件名和函数名的字符串形式。
// 返回一个字符串数组指针，使用完后记住释放那个指针。
char **backtrace_symbols(void *__const *__array, int __size);

// 和backtrace_symbols一样，但是结果会立即写入一个文件;
void backtrace_symbols_fd(void* __const *array, int __size, int __fd);
```

### gcc的rdynamic选项
经过上述两步，可以打印出行号, 但是函数名仍然看不到，因为最终二进制文件并没有保留符号，可以添加编译选项-rdynamic, 

### demangle

C++会对函数名进行重新命名，增加前缀, 增加参数类型后缀，如add(int, int)被替换成_Zaddii(int, int), 可以手动使用c++filt命令将名称还原。
函数中调用了abi::__cxa_demangle将名称直接打印还原。
需要包含头文件 #include <cxxabi.h>
