## Views

Checklist for examples:
- [ ] Vectors and other containers are const
- [ ] Vectors are called `vec`
- [ ] View are called `rng`
- [ ] View are non-const
- [ ] Use `'\n'` for break line
- [ ] Add a note for all variadic templates (see `cartesian_product`)

### `views::adjacent_filter`

The `adjacent_filter` view is a range adaptor that filters a range based on a binary predicate. The predicate function acts on two adjacent elements in the input range and removes the *second* element if the predicate returns `false`.

> **Note:** Independent of the predicate, the *first* element of the input range is always included in the result.

Here's an example of how to use `ranges::views::adjacent_filter` to remove non-unique elements from a range:
```cpp
const std::vector<int> vec{0, 1, 1, 2, 2, 2, 3, 4, 4, 5};
auto rng = vec | rv::adjacent_filter(std::not_equal_to{});
std::cout << rng << '\n';
// prints: [0,1,2,3,4,5]
```

### `views::adjacent_remove_if`

The `adjacent_remove_if` view is a range adaptor that filters a range based on a binary predicate. The predicate function acts on two adjacent elements in the input range and removes the *first* element if the predicate returns `true`.

> **Note:** Independent of the predicate, the *last* element of the input range is always included in the result.

Here's an example of how to use `ranges::views::adjacent_remove_if` to remove non-unique elements from a range:
```cpp
const std::vector<int> vec{0, 1, 1, 2, 2, 2, 3, 4, 4, 5};
auto rng = vec | rv::remove_if(std::equal_to{});
std::cout << rng << '\n';
// prints: [0,1,2,3,4,5]
```

### `views::all`

The `all` view is a range adaptor that simply returns a unmodified view to the source range. That way, you can copy the view to a range at constant cost or directly print the content of a container.

Here's an example of how to use `ranges::views::all` to print the elements of a STL vector (without using the [`fmt` library](https://github.com/fmtlib/fmt) or C++20's [`std::format`](https://en.cppreference.com/w/cpp/utility/format)):

```cpp
const std::vector<int> vec = {1, 2, 3, 4, 5};
auto rng = vec | rv::all;
std::cout << rng << '\n';  // prints: [1,2,3,4,5]
```

### `views::c_str`

The `c_str` view is a range adaptor that creates a null-terminated C-style string representation of the input range, such as a `const char*`.

Here's an example of how you can use `ranges::views::c_str` to get a view of the individual characters of a `std::string`:

```cpp
const std::string s{"test"};
auto rng = rv::c_str(s.c_str());
std::cout << rng << '\n';  // prints: [t,e,s,t]
```

> **Note:** The `c_str` view does not have a pipe operator and hence the input range needs to be passed via the standard parentheses syntax.

### `views::cache1`

The `cache1` view caches the last element in the input view. That way, dereferencing a range's iterator multiple times does not lead to a recomputation.

Here's an example of how the `cache1` view can be used to prevent a (potentially expensive) recomputation of the first element in a range:

```cpp
auto double_verbose = [](int i) {
    std::cout << "Doubling: " << i << " -> " << 2 * i << "\n";
return 2 * i;
};
const std::vector<int> vec{1, 2, 3, 4, 5};
auto rng = vec | rv::transform(double_verbose) | rv::cache1;
auto rng_begin = ranges::begin(rng);
*rng_begin;  // prints: Doubling: 1 -> 2
*rng_begin;  // prints nothing
```

Note that in the example above, without the use of the `cache1` view, the program would have printed two lines with the same content. This is because views of ranges are evaluated lazily.

### `views::cartesian_product`

The `cartesian_product` view is a range adaptor that generates the [Cartesian product](https://en.wikipedia.org/wiki/Cartesian_product) of two or more ranges. Formally, the (n-fold) Cartesian product is defined as the set of all possible ordered tuples that can be formed by choosing one element from each of the input sets.

Here's an example of how you can use the `cartesian_product` view to produce all possible pairs between the numbers between 1 through 5, and the letters between "a" through "e":

```cpp
auto numbers = rv::iota(1, 6);
auto letters = rv::iota('a', 'f');
auto pairs = rv::cartesian_product(numbers, letters);
for (const auto [x, y] : pairs) {
  std::cout << "(" << x << ", " << y << ")\n";
}
// prints
// (1, a)
// (2, a)
// ...
// (5, f)
```


> **Note:** The `cartesian_product` view makes use of variadic templates so that we can pass an arbitrary number of ranges to it. As a consequence, it does not provide the pipe-operator.

### `views::chunk` and `views::chunk_by`

The `chunk`  and `chunk_by` views are range adaptors that group elements of a range into contiguous sequences of elements ("chunks") of a specified size or whether they fulfill a binary predicate.  When specifying a fixed chunk size `N`, the last sequence in the output range may have fewer elements than `N` if the number of elements in the original range is not divisible by `N` without remainder.

Here's an example of how you can use the `chunk` view to group elements of a range into fixed-size chunks:

```cpp
auto rng = rv::iota(1, 9);
auto chunk_rng = rng | rv::chunk(3);
std::cout << chunk_rng << '\n';  // prints: [[1,2,3],[4,5,6],[7,8]]
```

Here's an example of how you can use the `chunk_by` view to group elements that do not differ from each other too much:

```cpp
const std::vector<int> vec{1, 1, 2, 5, 4, 9};
auto rng = vec | rv::group_by([](int i, int j) { return std::abs(i - j) <= 2; });
std::cout << rng << '\n';  // prints: [[1,1,2], [5,4], [9]]
```

### `views::common`

> **TODO:** Fill with content.

From the official documentation: *"Convert the source range to a common range, where the type of the end is the same as the begin. Useful for calling algorithms in the std:: namespace."*

### `views::concat`

The `concat` view is a range adaptor that concatenates multiple sources ranges into a single range.
Here's an example of how you can use the `concat` view:

```cpp
auto rng1 = rv::iota(1, 4);
auto rng2 = rv::iota(4, 7);
auto rng3 = rv::iota(7, 10);
auto rng = rv::concat(rng1, rng2, rng3);
std::cout << rng << '\n';  // prints: [1,2,3,4,5,6,7,8,9]
```

> **Note:** The `concat` view makes use of variadic templates so that we can pass an arbitrary number of ranges to it. As a consequence, it does not provide the pipe-operator.

> **Note:** There is another range adaptor that combines multiple ranges into a single range, that is the `join` view. The difference is that `concat` takes multiple sources ranges, whereas `join` takes a range of ranges as a source.

### `views::const_`

The `const_` view is a range adaptor that creates a new range containing the same elements as the original range, but with each element cast to a const reference.
This is useful when you want to create a immutable view over a mutable range such as a vector.

Here's an example of how you can use the `const` view to avoid accidentally modifying elements in a mutable range:

```cpp
std::vector<int> vec{1, 2, 3};  // mutable (!)
auto rng = vec | rv::const_;
// rng[0] = 0; // this would cause a compile error
vec[0] = 0;  // works just as expected
std::cout << rng << '\n';  // prints: [0, 2, 3]
```

> **Note:** (Im)mutability with views can be a bit confusing. See the following hints to avoid potential pitfalls: TODO: link to appropriate examples in the "pitfalls" section.

### `views::counted`

The `counted` view is an adaptor view, which allows you to create a range of specified number of elements, starting from a given iterator. The `counted` view can be useful when you want to work with a portion of a range or a sequence that might not have a well-defined end. It takes an iterator and a count, and creates a view that includes exactly the specified number of elements starting from the given iterator.

Here's an example of using the counted view:

```cpp
std::vector<int> vec = {1, 2, 3, 4, 5, 6};
auto counted_view = rv::counted(vec.begin() + 1, 3);
std::cout << counted_view << std::endl;  // prints: [2, 3, 4]
```

In this program, we first create a `std::vector` `numbers` containing the integers from 1 to 6. We then use the `counted` view to create a new range `counted_view` that includes exactly 3 elements starting from the second element of the numbers vector (i.e., the element with the value 2).

NOTE:

The `counted` view is conceptually similar to the `subrange` view, which allows you to create a range that represents a subsequence of an existing range, given two iterators: a beginning iterator and an ending iterator. The main difference between the counted view and the subrange view is in how they define the resulting range:

- `counted` view: Takes a starting iterator and a count of elements to include. It generates a range of the specified number of elements, starting from the given iterator.
- `subrange` view: Takes two iterators, a beginning iterator and an ending iterator. It generates a range that includes all the elements in the original range from the beginning iterator up to (but not including) the ending iterator.

With the explanation above, it is easy to see that the following two lines are equivalent:
```cpp
auto counted_view = rv::counted(vec.begin() + 1, 3);
auto subrange_view = ranges::subrange(vec.begin() + 1, vec.begin() + 4);
```

### Cycle

The `cycle` view is an adaptor view, which allows you to create a new range that endlessly repeats the elements of an existing range. This can be useful when you need a repeating pattern or sequence generated from an existing range.

Here's an example of using the `cycle` view:

```cpp
std::vector<int> vec = {1, 2, 3};
auto cycle_view = vec | rv::cycle;
std::cout << cycle_view | rv::take(8) << std::endl;  // prints: [1, 2, 3, 1, 2, 3, 1, 2]
```

In this program, we first create a `std::vector` `vec` containing the integers 1, 2, and 3. We then use the `cycle` view to create a new range `cycle_view` that endlessly repeats the elements of the vector.

Since the `cycle_view` is an infinite range, we need to limit the number of elements we want to process. In this example, we use the `take` view to create a new range that includes only the first 10 elements of the `cycle_view`. 

Keep in mind that working with infinite ranges, like the ones generated by the `cycle` view, can be dangerous if you don't limit the number of processed elements. Combining the `cycle` view with other views like `take` is essential to avoid infinite loops and process a specific number of elements.

### Drop and its Variants

The `drop_x` range adaptors allow you to create new ranges by skipping or removing elements from the original range, based on different criteria like element count or a predicate function.

Here are four examples of using the different variants of this range adaptor:

- The `drop` view allows you to create a new range by skipping a specified number of elements from the beginning of an existing range:
  ```cpp
  std::vector<int> vec = {1, 2, 3, 4, 5};
  auto rng = vec | rv::drop(2);
  std::cout << rng << std::endl;  // prints: [3, 4, 5]

  // Ouf-of-bound access works without unexpected consequences:
  auto rng2 = vec | rv::drop(6);
  std::cout << rng2 << std::endl;  // prints: []
  ```
- The `drop_exactly` view is similar to the `drop` view, but it doesn't check whether the specified number of elements to skip is within bounds. This makes it slightly faster but potentially unsafe if the specified count is greater than the size of the range. The following example highlights this difference:
  ```cpp
  std::vector<int> vec = {1, 2, 3, 4, 5};
  auto drop_exactly_view = vec | rv::drop_exactly(2);
  std::cout << drop_exactly_view << std::endl;  // prints: [3, 4, 5]
  // auto drop_exactly_view = vec | rv::drop_exactly(6);  <-- undefined behavior
  ```
- The `drop_last` view allows you to create a new range by removing a specified number of elements from the end of an existing range.
  ```cpp
  std::vector<int> vec = {1, 2, 3, 4, 5};
  auto drop_view = vec | rv::drop_last(2);
  std::cout << drop_view << std::endl;  // prints: [1, 2, 3]
  ```
 
  > [!NOTE] Note
  > Unlike `drop`, `drop_last` requires the source range to be `sized`. 

  TODO: Add pitfall example where after using `drop_last` and `ranges::to_vector` the program can crash when we are dropping too many elements.

  - The `drop_while` view allows you to create a new range by skipping elements from the beginning of an existing range as long as a specified predicate is satisfied. Note that after the first time the predicate returns `false`, the elements are no longer dropped, which distinguishes it from the `remove_if` range adaptor. 
  ```cpp
  const std::vector<int> vec = {1, 2, 3, 4, 5, 1, 2, 3};
  auto drop_while_view = vec | rv::drop_while([](int i) { return i < 4; });
  std::cout << drop_while_view << std::endl;  // prints: [4, 5, 1, 2, 3]
  ```

### Empty

The `empty` view is a simple view that generates an empty range. It can be useful when you want to create an empty range of a specific type, for example as a default value for a function argument.

Here's an example of how you can use the `empty` view to create an empty range of integers:
```cpp
auto empty_rng = rv::empty<int>;
std::cout << empty_rng << std::endl;  // prints: []
```
In this example, the `empty` view is used to create an empty range of integers, which is then printed to the console.

### Enumerate

The `enumerate` view is an adaptor view that takes an input range and generates a new range with pairs of elements, where each pair consists of an index and the corresponding element from the input range.

Here's an example of how you can use the `enumerate` view to print out the indices and elements of a vector:
```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};
auto enumerated_rng = vec | rv::enumerate;
for (auto [index, value] : enumerated_rng)
{
    std::cout << index << ": " << value << std::endl;
}
// prints:
// 0: 1
// 1: 2
// ...
```
In this example, we first create a vector `vec` containing five integers. We then use the `enumerate` view to create a new range `enumerated_rng` that contains pairs of indices and elements from the `vec` vector. Finally, we use a range-based for loop with structured bindings to iterate over the pairs and print out the index and value of each element in the vector.

### Exclusive Scan

The `exclusive_scan` view is an adaptor view that takes an input range and performs an exclusive prefix sum operation on the elements, using a specified binary operation (by default, addition). It generates a new range with the same size as the input range.

Here's an example of how you can use the `exclusive_scan` view to compute the exclusive prefix sum of a range of integers:
```cpp
auto rng = rv::iota(1, 6);
auto exclusive_scan_rng = rng | rv::exclusive_scan(0);
std::cout << exclusive_scan_rng << std::endl; // prints: [0, 1, 3, 6, 10]
```
In this example, we create a range `rng` of integers from 1 to 5 using the `iota` view. We then use the `exclusive_scan` view to compute the exclusive prefix sum of the `rng` range, with an initial value of 0. 

TODO: Add example without the default add to give more intuition on how this view works

### Filter

The `filter` view is an adaptor view that takes an input range and a predicate function, and generates a new range containing only the elements from the input range that satisfy the predicate.

Here's an example of how you can use the `filter` view to filter out the even numbers from a range of integers:
```cpp
auto is_even = [](int x) { return x % 2 == 0; };
auto rng = rv::iota(1, 10);
auto even_rng = rng | rv::filter(is_even);
std::cout << even_rng << std::endl; // prints: [2, 4, 6, 8]
```
In this example, we first define a lambda function `is_even` that checks whether an integer is even or not. We then create a range `rng` using the `iota` view, which generates a range of integers from 1 to 9. Finally, we use the `filter` view to create a new range `even_rng`.

*Note:* Be aware that the `filter` view can lead to multiple evaluations of the predicated passed to the view adaptor. It is therefore best to avoid having filter predicates that are expensive to evaluate. 
TODO: Explain exactly in what circumstances this leads to problems.

### For Each

The `for_each` view is an adaptor view that takes an input range and a function, and applies the function to each element in the input range. This view is typically used for side effects, such as printing elements or modifying them in-place.

Here's an example of how you can use the `for_each` view to print each element in a range of integers:

```cpp
auto print = [](int x) { std::cout << x << " "; };
auto rng = rv::iota(1, 10);
rng | rv::for_each(print);  // prints: 1 2 3 4 5 6 7 8 9
```
In this example, we first define a lambda function `print` that prints an integer followed by a space. We then create a range `rng` using the `iota` view, which generates a range of integers from 1 to 9. Finally, we use the `for_each` view to apply the `print` function to each element in the `rng` range.

### Generate and Generate-N

The `generate` and `generate_n` views are factory views that take a function and generates an (infinite) range by repeatedly calling the function. As the name suggests, `generate_n` takes a second argument, which specifies the number of elements to generate.

Here's an example of how you can use the `generate` view to create an infinite range of random integers:
```cpp
std::random_device rd;
std::mt19937 gen(rd());
std::uniform_int_distribution<> dis(1, 100);

auto random_int = [&]() { return dis(gen); };
auto rng = rv::generate(random_int) | rv::take(5);
std::cout << rng << std::endl; // prints: [x1, x2, x3, x4, x5]
```
In this example, we first set up a random number generator `gen` and a uniform integer distribution `dis`. We then define a lambda function `random_int` that generates a random integer using the distribution. Next, we create an infinite range `rng` using the `generate` view with the `random_int` function, and then use the `take` view to limit the number of elements to 5.

If you don't want to generate a range with infinite size, here's an example of how you can use the `generate_n` view to create a range of 5 random integers:
```cpp
auto random_int = [&]() { return dis(gen); };
auto rng = rv::generate_n(random_int, 5);
std::cout << rng << std::endl; // prints: [x1, x2, x3, x4, x5]
```
In this example, we use the same random number generator and distribution as in the previous example. We create a finite range `rng` of random integers using the `generate_n` view with the `random_int` function and a count of 5.


### Getlines

The `getlines` view is an adaptor view that takes an `std::istream` and generates a range of lines read from the stream.

Here's an example of how you can use the `getlines` view to read lines from a file and print them:
```cpp
std::ifstream file("example.txt");
auto rng = rv::getlines(file);
for (const auto &line : rng)
{
    std::cout << line << std::endl;
}
```
In this example, we first create an `std::ifstream` object file to read from a file named `example.txt`. We then use the `getlines` view to create a range `rng` that represents the lines read from the file. Finally, we use a range-based for loop to iterate over the lines in the `rng` range and print each line to the console.

### Groupy-By


> [!NOTE] Note
> The `group_by` view has been renamed in latest version (v0.12) of range-v3 to `chunk_by`. See the respective section above for an example.

### Indices

The `indices` view is a factory view that takes a `count` and generates a range of integer indices from a start index (defaults to `0`) to `count - 1`.

Here's an example of how you can use the `indices` view to create a range of integer indices:
```cpp
auto rng_from_zero = rv::indices(5);
auto rng_from_two = rv::indices(2, 5);
std::cout << rng_from_zero << std::endl;  // prints: [0, 1, 2, 3, 4]
std::cout << rng_from_two << std::endl;   // prints: [2, 3, 4]
```
In this example, we use the `indices` view to create two ranges `rng_from_zero` and `rng_from_two` of integer indices from `0` to `4` and `2` to `4`, respectively.

NOTE:

The `indices` view is closely related to the `iota` and `closed_iota` views. However, `iota` allows us to create an infinite range. Be careful to remember the subtle differences between the views.

### Indirect

The `indirect` view is an adaptor view that takes an input range of iterators or pointers, and generates a new range by dereferencing the elements of the input range.

Here's an example of how you can use the `indirect` view to create a range of elements from a range of pointers:
```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};
auto ptrs = vec | rv::transform([](const int& i) {return &i;});
auto rng = ptrs | rv::indirect;
std::cout << rng << std::endl;  // prints: [1, 2, 3, 4, 5]
```
In this example, we first create a vector `vec` of integers and a range `ptrs` of pointers to the elements of `vec`. Finally, we use the `indirect` view to create a range `rng` containing the elements of `vec` by dereferencing the pointers in the `ptrs` range.

### Intersperse

The `intersperse` view is an adaptor view that takes an input range and a value, and generates a new range by inserting the value between each pair of elements in the input range.

Here's an example of how you can use the `intersperse` view to insert a separator between elements in a range:
```cpp
auto rng = rv::iota(1, 5);
auto separated_rng = rng | rv::intersperse(-1);
std::cout << separated_rng << std::endl; // prints: [1, -1, 2, -1, 3, -1, 4]
```
In this example, we first create a range `rng` using the `iota` view, which generates a range of integers from 1 to 4. We then use the `intersperse` view to create a new range `separated_rng` that inserts the value `-1` between each pair of elements in the `rng` range.

### Iota and Closed-Iota

The `iota` and `closed_iota` views are a factory views that generate ranges of incrementing values. The `iota` view can either generate an infinite range starting from a certain value or an open range starting from the first value and ending with the second value (exclusive). The `closed_iota` alos generates a finite range, but includes the second value in the range.

Here's an example of how you can use the `iota` and `closed_iota` views to create a range of incrementing integers:
```cpp
auto rng_inf = rv::iota(3) | rv::take(5);
auto rng_open = rv::iota(3, 9);
auto rng_closed = rv::closed_iota(3, 9);
std::cout << rng_inf << std::endl;     // prints: [3, 4, 5, 6, 7]
std::cout << rng_open << std::endl;    // prints: [3, 4, 5, 6, 7, 8]
std::cout << rng_closed << std::endl;  // prints: [3, 4, 5, 6, 7, 8, 9]
```

NOTE: 

Note the similarity of the `iota` views to the `indices` view.
### Istream

The `istream` view is a factory view that takes an `std::istream` object (like `std::ifstream` or `std::istringstream`) and a delimiter, and generates a range of values read from the stream, separated by the specified delimiter.

Here's an example of how you can use the `istream` view to read a sequence of integers from a string:
```cpp
std::istringstream iss("1 2 3 4 5");
auto rng = ranges::istream<int>(iss);
std::cout << rng << std::endl;  // prints: [1, 2, 3, 4, 5]
```
In this example, we first create an `std::istringstream` object `iss` containing a sequence of integers separated by spaces. We then use the `istream` view to create a range `rng` of integers read from the `iss` stream.

NOTE: 

For the use of the `std::istringstream` object, you need to include the `sstream` header of the standard library.


### Join

The `join` view is an adaptor view that takes an input range of ranges and flattens them into a single range by concatenating the elements from the inner ranges.

Here's an example of how you can use the `join` view to flatten a range of ranges:
```cpp
std::vector<std::vector<int>> nested = {{1, 2}, {3, 4}, {5, 6}};
auto rng = nested | rv::join;
std::cout << rng << std::endl; // prints: [1, 2, 3, 4, 5, 6]
```
In this example, we first create a nested vector `nested` containing three inner vectors of integers. We then use the `join` view to create a range `rng` that flattens the nested vector into a single range containing all the elements from the inner vectors.

### Keys  

The `keys` view is an adaptor view that takes an input range of key-value pairs (such as a map or an associative container) and creates a new range containing only the keys. This view can be useful when you want to iterate over the keys in a key-value collection without having to access the values.

Here's an example of how you can use the keys view to create a range of keys from a `std::map`:
```cpp
std::map<int, std::string> my_map = {{1, "one"}, {2, "two"}, {3, "three"}};
auto keys_rng = my_map | rv::keys;
std::cout << keys_rng << std::endl;  // prints: [1, 2, 3]
```
In this example, we first create a `std::map` named `my_map` containing three key-value pairs. We then use the `keys` view to create a new range `keys_rng` that contains only the keys from the `my_map` collection. 

NOTE:

The "sibling" view `values` exists to get the values of a map.

### Linear Distribute

The `linear_distribute` view is a factory view that takes a lower bound, an upper bound, and a count, and generates a range of evenly spaced values between the lower and upper bounds (inclusive), with the specified number of values.

Here's an example of how you can use the `linear_distribute` view to create a range of linearly distributed floating-point values:
```cpp
auto rng = rv::linear_distribute(0.0, 1.0, 5);
std::cout << rng << std::endl; // prints: [0, 0.25, 0.5, 0.75, 1]
```
In this example, we use the `linear_distribute` view to create a range `rng` of 5 floating-point values evenly distributed between 0.0 and 1.0, inclusive.


### Move

TODO: Find good example for this view. Else don't make example.

### Partial Sum

The `partial_sum` view is an adaptor view that takes an input range and creates a new range by computing the partial sums of the elements in the input range. Each element in the new range is the sum of all elements up to and including the corresponding element in the input range.

Here's an example of how you can use the `partial_sum` view to compute the partial sums of a range of integers:
```cpp
auto rng = rv::iota(1, 6);
auto partial_sum_rng = rng | rv::partial_sum;
std::cout << partial_sum_rng << std::endl; // prints: [1, 3, 6, 10, 15]
```
In this example, we first create a range `rng` using the `iota` view, which generates a range of integers from 1 to 5. We then use the `partial_sum` view to create a new range `partial_sum_rng` that contains the partial sums of the integers from the `rng` range.

### Remove and Remove-If

The `remove` and `remove_if` views are adaptor views that take an input range and create a new range by removing elements that match a specified value or satisfy a given predicate, respectively. These views can be useful when you want to create a filtered range without modifying the input range itself.

Here's an example of how you can use the `remove` view to create a new range by removing elements that match a specified value:
```cpp
std::vector<int> vec = {1, 2, 3, 2, 4, 2, 5};
auto removed_rng = vec | rv::remove(2);
std::cout << removed_rng std::endl;  // prints: [1, 3, 4, 5]
```
In this example, we first create a vector `vec` containing a few integers. We then use the `remove` view to create a new range `removed_rng` that contains the elements from the `vec` range with all occurrences of the value 2 removed.

Here's an example of how you can use the `remove_if` view to create a new range by removing elements that satisfy a given predicate:
```cpp
auto is_even = [](int x) { return x % 2 == 0; };
std::vector<int> vec = {1, 2, 3, 4, 5, 6, 7};
auto removed_rng = vec | rv::remove_if(is_even);
std::cout << removed_rng << std::endl;  // prints: [1, 3, 5, 7]
```
In this example, we first define a lambda function `is_even` that checks whether an integer is even or not. We then create a vector `vec` containing a few integers. We use the `remove_if` view to create a new range `removed_rng` that contains the elements from the `vec` range with all even numbers removed.

### Repeat and Repeat-N

The `repeat` and `repeat_n` views are factory views that create a new range by repeating a given value indefinitely or for a specified number of times, respectively. These views can be useful when you want to generate a range filled with a specific value or when you need to repeat a value multiple times.

Here's an example of how you can use the `repeat` view to create an infinite range filled with a specified value:
```cpp
auto repeated_rng = rv::repeat(42) | rv::take(5);
std::cout << repeated_rng << std::endl;  // prints: [42, 42, 42, 42, 42]
```
In this example, we use the `repeat` view to create a new infinite range `repeated_rng` that contains the value 42 repeated indefinitely. Since the range is infinite, we need to limit the output via the `take` view.

Here's an example of how you can use the `repeat_n` view to create a range filled with a specified value repeated for a specified number of times:
```cpp
auto repeated_rng = rv::repeat_n(42, 5);
std::cout << repeated_rng << std::endl;  // prints: [42, 42, 42, 42, 42]
```
In this example, we use the `repeat_n` view to create a new range `repeated_rng` that contains the value `42` repeated five times.

### Replace and Replace-If

The `replace` and `replace_if` views are adaptor views that create a new range by replacing elements in the input range that either match a specified value or satisfy a given predicate, respectively. These views can be useful when you want to create a modified range without altering the input range itself.

Here's an example of how you can use the `replace` view to create a new range with all occurrences of a specified value replaced by another value:
```cpp
std::vector<int> vec = {1, 2, 3, 2, 4, 2, 5};
auto replaced_rng = vec | rv::replace(2, 99);
std::cout << replaced_rng << std::endl;  // prints: [1, 99, 3, 99, 4, 99, 5]
```
In this example, we first create a vector `vec` containing a few integers. We then use the `replace` view to create a new range `replaced_rng` that contains the elements from the `vec` range with all occurrences of the value `2` replaced by the value `99`.

Here's an example of how you can use the replace_if view to create a new range by replacing elements that satisfy a given predicate:

```cpp
auto is_even = [](int x) { return x % 2 == 0; };
std::vector<int> vec = {1, 2, 3, 4, 5, 6, 7};
auto replaced_rng = vec | rv::replace_if(is_even, 99);
std::cout << replaced_rng << std::endl;  // prints: [1, 99, 3, 99, 5, 99, 7]
```
In this example, we first define a lambda function `is_even` that checks whether an integer is even or not. We then create a vector `vec` containing a few integers. We use the `replace_if` view to create a new range `replaced_rng` that contains the elements from the `vec` range with all even numbers replaced by the value `99`.

### Reverse

The `reverse` view is an adaptor view that creates a new range with the elements of the input range in reverse order. This view can be useful when you want to process elements in a range from the end to the beginning without modifying the input range.

Here's an example of how you can use the `reverse` view to create a new range with the elements of an input range in reverse order:
```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};
auto reversed_rng = vec | rv::reverse;
std::cout << reversed_rng << std::endl;  // prints: [5, 4, 3, 2, 1]
```
In this example, we first create a vector `vec` containing a few integers. We then use the `reverse` view to create a new range `reversed_rng` that contains the elements from the `vec` range in reverse order.

### Sample

The `sample` view is an adaptor view that creates a new range by randomly sampling a specified number of elements from the input range without replacement. This view can be useful when you want to create a random subset of elements from a range without modifying the input range.

Here's an example of how you can use the `sample` view to create a new range with a random subset of elements from an input range:
```cpp
std::vector<int> vec = {1, 2, 3, 4, 5, 6, 7, 8, 9};
std::default_random_engine rng(42); // using a fixed seed for reproducibility
auto sampled_rng = vec | rv::sample(3, rng);
std::cout << sampled_rng << std::endl;  // prints: [1, 4, 6]
```
In this example, we first create a vector `vec` containing a few integers. We then create a `std::default_random_engine` named `rng` with a fixed seed for reproducibility. We use the `sample` view to create a new range `sampled_rng` that contains a random subset of three elements from the `vec` range, generated using the random engine `rng`. 

NOTE:

Note that the results of the `sample` view may vary depending on the random engine and seed used. In this example, we used a fixed seed for reproducibility, but you can use a time-based seed or a different random engine if you need different random subsets each time your program runs.

### Set Algorithm Views

These four set operation views are adaptor views that allow you to create new ranges by performing set operations on sorted input ranges, such as set difference, set intersection, set symmetric difference, and set union.

Here are four examples of using the different views:
- The `set_difference` view allows you to create a new range by finding the elements that are present in the first range but not in the second range:
```cpp
std::vector<int> vec1 = {1, 2, 3, 4, 5};
std::vector<int> vec2 = {3, 4, 5, 6, 7};
auto set_diff_view = rv::set_difference(vec1, vec2);
std::cout << set_diff_view << std::endl;  // prints: [1, 2]
```
- The `set_intersection` view allows you to create a new range by finding the elements that are common to both input ranges:
```cpp
std::vector<int> vec1 = {1, 2, 3, 4, 5};
std::vector<int> vec2 = {3, 4, 5, 6, 7};
auto set_intersect_view = rv::set_intersection(vec1, vec2);
std::cout << set_intersect_view << std::endl;  // prints: [3, 4, 5]
```
- The `set_symmetric_difference` view allows you to create a new range by finding the elements that are present in either of the input ranges but not in both:
```cpp
std::vector<int> vec1 = {1, 2, 3, 4, 5};
std::vector<int> vec2 = {3, 4, 5, 6, 7};
auto set_sym_diff_view = rv::set_symmetric_difference(vec1, vec2);
std::cout << set_sym_diff_view << std::endl;  // prints: [1, 2, 6, 7]
```
- The `set_union` view allows you to create a new range by merging the input ranges, removing duplicate elements:
```cpp
std::vector<int> vec1 = {1, 2, 3, 4, 5};
std::vector<int> vec2 = {3, 4, 5, 6, 7};
auto set_union_view = rv::set_union(vec1, vec2);
std::cout << set_union_view << std::endl;  // prints: [1, 2, 3, 4, 5, 6, 7]
```
  
NOTE: 

In all of these examples, the input ranges `vec1` and `vec2` must be sorted before performing the set operations. The resulting ranges will also be sorted.

NOTE: 

These also work with `std::set`  -> TODO: show examples

### Single

The `single` view is an adaptor view that creates a range of size one from a single object. This view can be useful, for instance, when you want to append a single element to an existing range with the `concat` view adaptor. See [this example](tips-and-tricks#creating-custom-view-adaptors) where we implement a custom view that does exactly that. 

Here's an example of how you can use the `single` view to create a new range from a single object:
```cpp
const int i = 42;
auto rng = rv::single(i);
std::cout << rng << '\n';  // prints: [42]
```
In this example, we first create an integer  `i`. We then create a range named `rng` with the `single` view adaptor. When printing the values for `rng`, we see that the range only contains only a single value.

[click on this link](#slice)

### Slice

The `slice` view is an adaptor view that takes a contiguous subset of an existing range based on a start and end index. Note that the end index is *included* in the sliced view.
Here's an example of how you can use the `slice` view adaptor to create a contiguous view of a range:
```cpp
const std::vector<int> vec{0, 1, 2, 3, 4};
auto rng = vec | rv::slice(1, 4);
std::cout << rng << '\n';  // prints: [1, 2, 3]

// Use `ranges::end` for a slice including the last element
auto rng = vec | rv::slice(1, ranges::end);
std::cout << rng << '\n';  // prints: [1, 2, 3, 4]
```
In this example, we first create a vector `vec` of integers from 0 to 4. We then create a "sliced" view `rng` from index 1 up to and including index 4.

While this view adaptor can be quite handy, there are two important things to keep in mind: 
1. If you know in advance, that you only want to extract the last elements of a range, it is oftentimes better to use the `take_last` view adaptor. TODO: add a link to the section that explains the `take` view adaptors. 
2. If the end index exceeds the size of the range, the view will point to invalid memory. In case that you do not know the size of the input range in advance, make sure to use the `ranges::end` utility instead of `ranges::distance(rng)`. This is because the `distance` utility will iterate over the input range and eventually evaluate all function applied to its elements, which can lead to a lot of redundant computations (at least for ranges that don't have random access, i.e., whose size cannot be determined in constant time).


*Note:* See also the `subrange` view adaptor for ... TODO: fill sentence after the example is implemented.

### Sliding

The `sliding` view is an adaptor view that propagates a window of a fixed size along the input range. For every entry of the resulting range, the window is moved by one position so that the `sliding` view creates a range of ranges. 
Here's an example of how you can use the `sliding` view adaptor to create groups of three consecutive elements of a range:
```cpp
const std::vector<int> vec{0, 1, 2, 3, 4};
auto rng = vec | rv::sliding(3);
std::cout << rng << '\n';
// prints: [[0, 1, 2], [1, 2, 3], [2, 3, 4]]
```
In this example, we first crate a vector `vec` of integers from 0 to 4. We then create the new range `rng` via the `sliding` view adaptor. The result is a range of ranges where each element consists of three consecutive elements of `vec`. With the `sliding` view adaptor, we can for example easily implement a moving average filter of a time sequence.
Note that when the specified window size is larger than the size of the input range, the result is an empty range.
### Span

TODO:

Is this equivalent to std::span?

### Split and Split-When

The `split` and `split_when` views are view adaptors that help to break up ranges into subsequences. The locations at which the ranges are split are specified by either a or pattern that the values are compared against (`split`) or by a predicate that returns a bool (`split_when`). The result is a range of ranges. Note that the matched elements are not longer part of the result.
Here is an example of how you use the `split` view adaptor to break up a list of integers or a comma-separated string:
```cpp
const std::vector<int> vec{0, 1, 2, 3, 1, 4};
auto rng = vec | rv::split(1);
std::cout << rng << '\n';  // prints: [[0], [2,3], [4]]

const std::string s = "ab,cd,e";
auto rng2 = rv::split(s, ',');
std::cout << rng2 << '\n';  // prints: [[a,b], [c,d], [e]]
```
In the example above, we first create a vector `vec` of integers. The vector is then split at the every entry that is equal to one. Similarly, we create a comma-separated string `s` and split it up in the substrings inbetween the commas. Note that the resulting substrings are not interepreted as `std::string` but as a range of `char`. Hence, the printed output looks a little odd. If you want to obtain the substrings to be of type `std::string`, you need to actively convert them.
Note that, it is *not* possible to split strings based on regular expressions with the range-v3 library. This is because the `split` view adaptor has to work with all different types, not only strings. Fortunately, there exists a view adaptors that does exactly this, though: `tokenize`.

In case the simple pattern matching of a single element is not sufficient for you application, you can make arbitrary decision where to split your range based on a unary predicate. Here is an example of how you can use the `split_when` view adaptor:
```cpp
const std::vector<int> vec{0, 1, 2, 4, 3, 6};
auto rng3 = vec | rv::split_when([](int i) { return i % 2; });
std::cout << rng3 << '\n';  // prints: [[0], [2, 4], [6]]
```
In the example above, we make the decision of splitting the vector of integers `vec` whether the elements are odd.
### Stride

The `stride` view is a view adaptor that takes a every `n`-th element of the input range. Here is an example of you can use the `stride` view adaptor to "sparsify" a vector:
```cpp
const std::vector<int> vec{0, 1, 2, 3, 4, 5, 6};
auto rng = vec | rv::stride(3);
std::cout << rng << '\n';  // prints: [0, 3, 6]
```
In the example above, we first create a vector of integers `vec`. We then use the `stride` view adaptor with an argument value of three that results in a range `rng` that corresponds to every third element of `vec`.
### Subrange
The `subrange` range adaptor/factory has two overloads. With the first one, we can extract the `begin` iterator and the `sentinel` of a (finite) range as a pair. From these, we can simply create new ranges with the second overload of `subrange`. Here is an example of you can use both overloads to create a contiguous subrange of a given vector:

```cpp
const std::vector<int> vec{0, 1, 2, 3, 4};
// First overload for subrange
auto [i, j] = ranges::subrange(vec);
// Second overload for subrange
auto rng = ranges::subrange(i + 1, j - 1);
std::cout << rng << '\n';  // prints: [1, 2, 3]
```

In the example above, we first create a vector of integers `vec`. We then use the `subrange` to get the the `begin` iterator and sentinel (which for the case of `std::vector` corresponds to the `end` iterator). Here, we make use of structured bindings to directly access the elements of the returned pair. We then use the `subrange` range factory and simple iterator arithmetic to create a range without the first and last elements of the original vector. The same effect could have been achieved with the `drop`/`drop_last` range adaptors.

> [!NOTE] Note
> There are several peculiarities about `subrange` that make it slightly different compared to other range adaptors/factories.
> 1. There are two overloaded versions of `subrange`: One overload that takes a range as an input and outputs a iterator/sentinel pair. And one overload that takes a iterator/sentinel pair as an input and inputs a range.
> 2. While the implementation of `subrange` lies next to the other view implementations in `range/view/subrange.hpp`, the namespace is *not* `ranges::views`, but simply `ranges`.
> 3. The `subrange` view adaptor (the overload that takes a range as input) has no pipe operator, i.e., `vec | ranges::subrange` does not compile.

### Tail
The `tail` range adaptor simply simply takes all but the first element of the input range. Here is an example how you can use the `tail` range adaptor:
```cpp
const std::vector<int> vec{0, 1, 2, 3, 4};
auto rng = vec | ranges::views::tail;
std::cout << ranges::views::all(rng) << "\n";  // prints: [1, 2, 3, 4]
```
In the example above, we first create a vector of integers `vec` and then use the `tail` range adaptor to access the vector's "tail". Note that the `tail` range adaptor is equivalent to the `drop(1)` range adaptor.
### Take and its Variants
The `take_x` range adaptors allow you to create new ranges by taking only a subset of the from the input range, based on different criteria like element count or a predicate function.

Here are four examples of using the different variants of this range adaptor:
- The `take` range adaptor creates a new range by only taking a specified number of elements from the beginning of the input range:
```cpp
const std::vector<int> vec = {0, 1, 2, 3, 4};
auto rng = vec | rv::take(3);
std::cout << rng << "\n";  // prints: [0, 1, 2]

// Out-of-bound access works without unexpected consequences:
auto rng2 = vec | rv::take(9);
std::cout << rng2 << "\n";  // prints: [0, 1, 2, 3, 4]
```
- The `take_exactly` range adaptor is similar to the `take` adaptor, but it does not check whether the specified number of elements to take is within bounds. This makes it slightly faster but potentially unsafe if the specified count is greater than the size of the input range. The following example highlights this difference:
```cpp
const std::vector<int> vec = {0, 1, 2, 3, 4};
auto rng = vec | rv::take_exactly(3):
std::cout << rng << "\n";  // prints: [0, 1, 2]

// Out-of-bound access leads to undefined behavior:
auto rng2 = vec | rv::take(9);
std::cout << rng2 << "\n";  // prints: [0, 1, 2, 3, 4, 0,  4113, 0, 824979547]
```
- The `take_last` range adaptor creates a view on the input range by only taking a specified number of elements from the end:
```cpp
const std::vector<int> vec = {0, 1, 2, 3, 4};
auto rng = vec | rv::take_last(3);
std::cout << rng << "\n";  // prints: [2, 3, 4]
```
- The `take_while` range adaptor creates a view on the input range by taking elements that satisfy a specified predicate. Note that after the first time the predicate returns `false`, the elements are no longer taken, which distinguishes it from the `filter` range adaptor.
```cpp
const std::vector<int> vec = {0, 1, 2, 3, 4, 0, 1, 2};
auto rng = vec | rv::take_while([](int i) { return i < 3; });
std::cout << rng << "\n";  // prints: [0, 1, 2, 3]
```
### Tokenize
The `tokenize` range adaptor splits a `std::string` into a range of substrings based on a regular expression. Here is an example how you can split a camel case-based string:
```cpp
#include <regex>
#include <string>
const std::string s{"HelloWorld"};
auto rng = s | rv::tokenize(std::regex{"[A-Z]+[a-z]+"});
std::cout << rng << "\n";  // prints: [Hello, World]
```
In the example above, we create a `std::string` based on camel case. We then use the regular expression `[A-Z]+[a-z]+`, i.e., a capital letter followed by a lower case letter, to split the string into substrings.
### Transform
The `transform` range adaptor lazily applies a function (strictly speaking, any callable) to each element of the input range. The resulting view is then a range of the function's output type. Here is an example how you can square all elements of a list of integers:
```cpp
const std::vector<int> vec{0, 1, 2, 3, 4};
auto rng = vec | rv::transform([](int i) { return i*i; });
std::cout << rng << "\n";  // prints: [0, 1, 4, 9, 16]
```
As mentioned above, not only lambda functions can be used to the `transform` range adaptor. A particularly handy use-case for the `transform` adaptor is when you need to access specific properties of a struct:
```cpp
struct Foo { double bar; };
auto foo_rng = rv::indices(4) | rv::transform([](int i) { return Foo{i}; });
auto bar_rng = foo_rng | rv::transform(&Foo::bar);
std::cout << bar_rng << "\n";  // prints: [0, 1, 2, 3]
```
In the example above, we use the `transform` range adaptor to both create a range of `Foo` objects via a lambda function and access their `bar` properties via the pointer to the corresponding member (or member function if we would have a getter function).
### Trim
The `trim` range adaptor removes all elements at the front and the end of an input range that fulfill a unary predicate. This range adaptor is particularly helpful for removing whitespace characters from strings, but can also used for any other values as shown in the following example:
```cpp
const std::string s{"\n Hello World "};
auto s_rng = s | rv::trim([](char c) { return (c == ' ') || (c == '\n'); });
std::cout << s_rng << "\n";  // prints: [H,e,l,l,o, ,W,o,r,l,d]

const std::vector<int> vec{0, 0, 1, 2, 3, 4, 4};
auto rng = vec | rv::trim([](int i) { return (i == 0) || (i == 4); });
std::cout << rng << "\n";  // prints: [1, 2, 3]
```
The same functionality can also be implemented by using the `drop_while` and `reverse` range adaptors, but the `trim` adaptor makes the usage much cleaner.

### Unbounded
The `unbounded` range adaptor works similar to the to the `bounded` adaptor, but instead of acting on ranges of finite size it works with *infinite* ranges. It takes an iterator of an unbounded range as its beginning as demonstrated in the following example:
```cpp
auto rng1 = rv::iota(1);
auto rng2 = rv::unbounded(ranges::begin(rng1) + 2);
std::cout << (rng2 | rv::take(3)) << '\n'  // prints: [4, 5, 6]
```
Note that we just use the `take` range adaptor so that we can print the (infinite) range to standard output.

### Unique
The `unique` range adaptor removes all duplicate elements that appear *consecutively* in the input range. Hence, when the input range is sorted we can get rid of all duplicate elements:
```cpp
const std::vector<int> vec1{1, 1, 2, 3, 3, 1};
auto rng1 = vec1 | rv::unique;
std::cout << rng1 << "\n";  // prints: [1, 2, 3, 1]

const auto vec2 = std::vector<int>{1, 1, 2, 3, 3, 1} | ranges::actions::sort;
auto rng2 = vec2 | rv::unique;
std::cout << rng2 << "\n";  // prints: [1, 2, 3]
```

> [!NOTE] Note
> Keep in mind that you will need to pass a *sorted* range to the `unique` range adaptor if your aim is to remove *all* duplicates.

### Values
The `values` view is an adaptor view that takes an input range of key-value pairs (such as a map or an associative container) and creates a new range containing only the values. This view can be useful when you want to iterate over the values in a key-value collection without having to access the keys.

Here's an example of how you can use the values view to create a range of values from a `std::map`:
```cpp
std::map<int, std::string> my_map = {{1, "one"}, {2, "two"}, {3, "three"}};
auto values_rng = my_map | rv::values;
std::cout << values_rng << std::endl;  // prints: [one, two, three]
```
In this example, we first create a `std::map` named `my_map` containing three key-value pairs. We then use the `values` view to create a new range `values_rng` that contains only the values from the `my_map` collection. 
> [!NOTE] Note
> The "sibling" range adaptor `keys` exists to get all keys of a map.

### Zip 
With the `zip` range adaptor we can iterate over multiple ranges simultaneously and access the respective elements via a pair/tuple. This adaptor is particularly helpful to avoid index-based for loops to access the elements of several ranges, as can be seen in the following example:
```cpp
const std::vector<int> vec{0, 1, 2, 3, 4};
const std::string s{"abcde"};
auto rng = rv::zip(vec, s);
for (const auto& e : rng) std::cout << "[" << e.first << "," << e.second << "],";
// prints: [0,a],[1,b],[2,c],[3,d],[4,e],
```
In the example above, the elements `e` in the zipped view `rng`  are of type `ranges::common_pair`. Hence, we can access their elements via the `e.first/second` shortcuts. If you zip more than two ranges the respective elements of the zipped view will be of type `ranges::common_tuple` and have to be accessed via `ranges::get<i>(e)`, where `i` is the index of the `i`-th argument in the `zip` adaptor.
> [!NOTE] Note
> For ranges of different lengths, the length of the resulting view is determined by the shortest input range.

### Zip-With
The `zip_with` range adaptor acts similar to the `zip` adaptor but immediately invokes a function on the elements as can be seen in the following example:
```cpp
const std::vector<int> vec1{0, 1, 2, 3, 4};
const std::vector<int> vec2{2, 2, 2, 2, 2};
auto rng = rv::zip_with(
    [](const int i, const int j) { return std::min(i, j); }, vec1, vec2);
std::cout << rng << "\n";  // prints: [0, 1, 2, 2, 2]
```
Note that the same rule as for the `zip` adaptor applies regarding the length of the zipped view.
