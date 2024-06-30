
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


// Create function --------------------------------------------------------------
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

  
// Call the function ------------------------------------------------------------

int main() {

// Result 1
int num_1 = 42;

int result_1 = step_function(num_1);

printf("Num_1: %d, Step: %d\n", num_1, result_1);

// Result 2
int num_2 = 0;

int result_2 = step_function(num_2);

printf("Num_2: %d, Step: %d\n", num_2, result_2);


// Result 3
int num_3 = -32767;

int result_3 = step_function(num_3);

printf("Num_3: %d, Step: %d\n", num_3, result_3);

}
```

## Types
### Fundamental types
#### Integer

 
 Whole numbers that can be signed (variable that can be positive, negative or zero) or unsigned (variable that only can be positive ). Integers can be __short int__, __int__, __long int__ and __long long int__ 

	![Integer types ](media/cpp_integer_types.png)

+ Literal: Hardcoded value in a program.
	+ Binary literal: Uses the prefix 0b
	+ Octal literal : Uses the prefix 0
	+ Decimal: Default
	+ Hexadecimal: Uses the prefix 0x


```cpp
#include <cstdio>

int main(){
unsigned short a = 0b10101010
printf("%hu\n",a)

int b = 0123
printf("%d", b)


}


```

+ Floating-Point Types