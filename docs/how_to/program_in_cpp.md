
## Compile code online

+ https://wandbox.org/
+ https://www.godbolt.org


```cpp
#include <cstdio>

int main(){
	printf("Hello, world");
	return 0;
}
```

```gcc
# First compile
g++ test_cpp.cpp -o test_cpp
```

Cpp has a single entry point called the main(). The entry point execute when the user runs the program.

### The compiler tool chain

The compiler tool chain is series of events that happen sequentially to turn source code into a program.

1) Preprocessor: Performs basic source code manipulation.
2) The compiler: Read a translation unit and generates an object file. Generates intermediate format called object code. '
3) The linker: Generates programs from object files 

### Cpp type system

C++ is object oriented language. Think as a object with specific characteristic that define it. Like a dog or a light switch.

A dog has 4 legs, barks and has furry tail. In this example the collection of states (4 legs, barks and has furry tail) define the  __type__ called dog 

C++ is a strongly typed language, meaning each object has a predefined data type. 

For example int is a type that can store whole numbers

### Declaring variables


```cpp
#include <cstdio>
#include <iostream>

int main(){

int number = 45/3;

printf("Returning division:\n");

std::cout << number;

return 0;

}
```

### Conditional statements

```cpp
int main() {
int x = 0;

if (x > 0)
	printf('x is Positive')

else if (x < 0)
	printf('x is negative')

else
; // Equality
; // Inequality
100 > x; // Greater than
123 >= x; // Greater than or equal to
-10 < x; // Less than
-99 <= x; // Less than or equal to
}
```