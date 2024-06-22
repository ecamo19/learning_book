
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
#include <cstdio>

int main() {

int x = 2;

if (x > 0) printf("x is positive");

else if (x < 0) printf("x is negative");

else printf("x is 0");
}
```

### Functions

```cpp
#include <iostream>
#include <cstdio>

int step_function(int x){

// Create object for storing the result

int result = 0;

if (x < 0){

	result = -1;
	}

else if (x > 0){

	result = 1;
	}

else

	result = 0;

return result;

}

  
// Call the function
int main() {

int value1 = step_function(100); // value1 is 1

printf("value1\n");

std::cout << value1;

printf("\n");

  

int value2 = step_function(0); // value2 is 0

printf("value2\n");

std::cout << value2;

printf("\n");

  

int value3 = step_function(-10); // value3 is -1

printf("value3\n");

std::cout << value3;

printf("\n");

  
  

}
```