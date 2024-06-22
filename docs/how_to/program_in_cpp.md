
## Set a dev environment



```cpp
#include <cstdio>

int main(){
	printf("Hello, world");
	return 0;
}
```

```gcc
g++ test_cpp.cpp -o test_cpp
```

Cpp has a single entry point called the main(). The entry point execute when the user runs the program.

### The compiler tool chain

The compiler tool chain is series of events that happen sequentially to turn source code into a program.

1) Preprocessor: Performs basic source code manipulation.
2) The compiler: Read a translation unit and generates an object file. Generates intermediate format called object code. '
3) The linker: Generates programs from object files 