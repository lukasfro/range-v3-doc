## Potential pitfalls

The range-v3 library is a powerful tool that brings many of the benefits of functional programming to C++.
However, like any powerful tool, it has some potential pitfalls.
In this section, we will discuss some things to be aware of when using range-v3.

TODO: check these explanation:
- https://github.com/ericniebler/range-v3/blob/0487cca29e352e8f16bbd91fda38e76e39a0ed28/include/range/v3/view/view.hpp#L150
- https://github.com/ericniebler/range-v3/issues/1406

### Lazy Evaluation

One of the core ideas of the library is _lazy evaluation_.
This means that operations on ranges, such as filtering or transforming elements, are not performed immediately when you request them.
Instead, they are performed on-the-fly as you iterate over the range.
While it can lead to significant efficiency improvements, it can also lead to the exact opposite, as the following example shows.
In the example, we keep track of how often the lambda function inside the `transform` view adaptor is called.

```cpp
int eval_counter = 0;
const std::vector<int> vec{1, 2, 3};
const auto rng = vec | ranges::views::transform([&](const int i) {
                     eval_counter++;
                     return 2 * i;
                 });

// Print the view.
std::cout << ranges::views::all(rng) << std::endl;

// Materialize the view.
const auto out_vec = rng | ranges::to_vector;

std::cout << "The lambda was called " << eval_counter << " times.";
// Prints: "The lambda was called 6 times."
```
Yes, you read correctly.
The lambda was called six times, not three times.
Lazily evaluated views are evaluated on the fly, meaning that every time we need to access an element of the view, it is evaluated.
In the example above, we are actually evaluating the view twice: First, when printing the view, and a second time when materializing the view.

In this example, the evaluation of the lambda is quite cheap.
But imagine that each evaluation of the lambda function comprises of a complex calculation.
This would lead to a huge performance hit.
To avoid this pitfall, you should first materialize the view and only print it afterwards.

### Mutating with Filter Range Adaptor

This [article](https://brevzin.github.io/c++/2023/04/25/mutating-filter/) describes the issue in more detail. 

### Lifetime Management

The range-v3 library operates on views of data, which are lightweight and don't own the data they operate on.
It's crucial to ensure the data being viewed remains alive and valid for the duration of the view's existence.
For example, if a view is created from a temporary container, accessing elements of the view after the temporary container is destroyed will result in undefined behavior.

Let's have a look at the following example.
```cpp

const auto f = []() {
    const std::vector<int> v{1, 2, 3};
    return v | ranges::views::transform([](const int i) { return 2 * i; });
};

const auto rng = f();
std::cout << ranges::views::all(rng) << std::endl;
```

Inside the lambda function `f`, we create a local vector `v` and fill it with three integers.
We then use the `transform` view adaptor and double each entry in the vector and immediately return the resulting view.
If you compile the code above, everything works fine until you run the code and look at the output: `[1318322912,65532,1318322656,65532,516,0, ...]` (your output will most likely differ).

The problem is that we only return a _view_ to the underlying vector `v`.
After the body of the lambda function has finished, the vector `v` goes out of scope and the view points to some unallocated point in memory.
In order to avoid this problem, we must _materialize_ the view into a container.
The following example shows how to do this:

```cpp
const auto g = []() {
    const std::vector<int> v{1, 2, 3};
    return v | ranges::views::transform([](const int i) { return 2 * i; }) |
           ranges::to_vector;
};

const auto rng = g();
std::cout << ranges::views::all(rng) << std::endl;
// prints: [2, 4, 6]

```

### Immutable Views

Generally, it is considered good practice (TODO: add a link to a good article) to declare variables that are not supposed to change as `const`.
However, certain views cannot be declared const.
This is because the view needs to keep track of the next position in the underlying range that satisfies the filter condition.
This state is modified as you iterate over the range, so it cannot be `const`.
This is in contrast to the `transform` view adaptor, which does not need to maintain any additional state.
For each position in the transformed range, it knows exactly which position in the underlying range it corresponds to, so it can apply the transformation function to that position without needing to store any additional state.

If you try to compile the following example, you will get a cryptic error message about a missing `<<`-operator.

```cpp
const std::vector<int> vec{1, 2, 3, 4, 5};
// Does not compile with `const` in front
const auto rng = vec | ranges::views::filter([](const int i) { return i % 2; });
std::cout << rng << std::endl;
```

Only when you remove the `const` at the declaration of `rng`, you should obtain the expected result and the program prints: `[1, 3, 5]`.
In you are uncertain whether a view adaptor maintains an internal state, a good rule of thumb is the following: If the output view has a different size than the input view, then the view probably has to maintain an internal state.
Examples for these view adaptors are: `views::filter`, `views::remove_if`, `views::unique`

If you are worried about losing the `const`ness of certain view, you don't have to worry too much about accidentally overwriting a view with another one.
This is because the copy assignment operator is not defined for views (TODO: double-check this statement)
The following example should clarify this issue:

```cpp
const std::vector<int> vec{1, 2, 3, 4, 5};
auto rng = vec | ranges::views::filter([](const int i) { return i % 2; });
// Does not compile because of missing copy assignment operator.
rng = vec | ranges::views::filter([](const int i) { return i % 2; });
```

To summarize: While it is generally recommended to declare immutable variables as `const`, this is not necessary for views.

### Adaptors Can Change View Type

TODO: Write about how different adaptors change the underlying view type. E.g., after `filter` , the view is not random access anymore.

### Infinite Ranges

The range-v3 library supports infinite ranges. These ranges can be used with range adaptors that don't need to process the entire range, like `ranges::views::take`. However, combining an infinite range with an adaptor that processes the entire range (like `ranges::to_vector` or `ranges::accumulate`) will result in an infinite loop.

### Compile Time 

TODO:  Make example that compares compile time from STL vs range-v3
