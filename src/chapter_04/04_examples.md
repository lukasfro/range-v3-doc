# Examples

In the following, we will give example on how to use various range adaptors and factories, and algorithms. For sake of brevity and readability, the preamble of the exemplary programs is left out. If you want to run the examples, I highly recommend using the [compiler explorer](https://godbolt.org/z/x9TW144b4). All examples that follow can be pasted in the following template program to run:

```cpp
// For some examples, you might need to include additional header files
#include <iostream>
#include <vector>
#include <string>

#include <range/v3/all.hpp>

// Alias for brevity
using rv = ranges::views;

int main() {
  // Paste the code snippets from the examples here.
  // ...

  return 0;
}
```


> [!tip] Tip
> In the template program above, we include the `range/v3/all.hpp` header. In your application, this is not recommended as it will unnecessarily increase the size and compile time of your application. Instead, you should only [include what  you use](https://github.com/include-what-you-use/include-what-you-use).
