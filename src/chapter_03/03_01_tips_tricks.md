## Tips and Tricks

### Creating Custom View Adaptors

#### Single-Argument View Adaptors

One of the greatest merits of using the range-v3 library is that one can freely combine several view adaptors/generators via the `|`-operator.  Moreover, if there is a sequence of view adaptors that you are using regularly, you can readily create a custom view to save you typing. See for example the following snippet:

```cpp
using namespace ranges::views;
auto filter_odd_numbers_and_square =
     filter([](const auto& val) { return val % 2; }) |
     transform([](const auto& val) { return val * val; });

const std::vector<int> vec{1, 2, 3, 4, 5};
auto rng = vec | filter_odd_numbers_and_square;
std::cout << rng << '\n';  // prints: [1, 9, 25]
```

In this  example, we are combining the `filter` and `transform` view adaptors to a new custom adpator called `filter_odd_numbers_and_square`. As the name suggests, this adaptor takes a range as an input, removes all values that are even, and then finally squares these values. You might be questioning the purpose of this admittedly quite constructed example, but I hope that you appreciate the ease with which we can combine multiple view adaptors.

#### Multi-Argument View Adaptors

In the previous section, we have created a custom view adaptor by simply combining two existing adaptors. In particular, note that the `|`-operator works readily for the new view adaptor, i.e., `vec | filter_odd_numbers_and_square` worked out of the box. Now, let's say we want to write a slightly more generic view adaptor. For instance, something along the lines of `vec | add_number(5)`. Instead of always adding a specified number, we want to pass the value to be added to our custom view adaptor. How might we go about this?

```cpp
auto add_number = [](auto&& rng, const auto& to_add) {
    return rng | transform([&](const auto& v) { return v + to_add; });
  };

const std::vector<int> vec{1, 2, 3, 4, 5};
auto rng = add_number(vec, 5);
std::cout << rng << '\n';  // prints: [6, 7, 8, 9, 10]
```

In the snippet above, we have wrapped the `transform` view adaptor with a lambda function. Inside the lambda function we then pipe the range `rng`  to a view adaptor that adds the value of `to_add` to all elements of the original range. Note the extensive use of the `auto` keyword. That way, the lambda function works with any kind of range as an input. For example. we could easily input a `std::set<double>` and add `5.2` to each element.
However, while the construction of the lambda does not pose much overhead, we have to use the new view adaptor as `add_number(vec, 5)`. The `|`-operator is not automatically created for us, i.e., the following does not compile `vec | add_number(5)`. This is because the `add_number` is not a view adaptor, but a lambda function; and lambda functions take care of a lot of code generation under the hood for us, they (unfortunately) do not implement the `|`-operator for us. Luckily, the range-v3 library provides a handy utility function for us, i.e., `make_pipeable`.

```cpp
auto add_number_pipeable = [&](const auto& to_add) {
  return ranges::make_pipeable([&](auto&& rng) {
    return add_number(std::forward<decltype(rng)>(rng), to_add);
  });
};

// pipe-operator can now be used with the new wrapper function.
auto rng2 = vec | add_number_pipeable(5);
std::cout << rng2 << '\n';  // prints: [6, 7, 8, 9, 10]
```

We could now leave this example as is, but I see the following problems:
1. The `add_number` function does not resemble a real-world use case. While it was good for understanding the basics, I think we can find a more appropriate example that we actually might find useful.
2. As lambda functions cannot be overloaded, we had to choose a different name for `add_number_pipeable`. While this name was descriptive for our small example, I don't want to add the `_pipeable` suffix to all my custom view adaptors that implement the `|`-operator. Moreover, maybe not everyone prefers the `|`-operator, and I don't want to have different names for functions that do exactly the same but use a slightly different syntax.

Here is an example that fixes both of these issues:

```cpp
namespace rv = ranges::views;

// Adds an element to the beginning of a range.
template <typename Rng, typename T>
auto prepend(Rng&& rng, T t) {
  return rv::concat(rv::single(std::move(t)), std::forward<decltype(rng)>(rng));
}

// Makes the `prepend` view adaptor pipeable.
template <typename T>
auto prepend(T t) {
  return ranges::make_pipeable([t_ = std::move(t)](auto&& rng) {
    return prepend(std::forward<decltype(rng)>(rng), std::move(t_));
  });
}

int main() {
  const std::vector<int> vec{1, 2, 3, 4, 5};
  auto rng = vec | prepend(0); // or: prepend(vec, 0)
  std::cout << rng << '\n';  // prints: [0, 1, 2, 3, 4, 5]
}
```

In the snippet above, we define a custom view adaptor that prepends any element to an already existing range - this is quite handy. Particularly, this circumvents the issue that the `concat` view adaptor is not pipeable. As the `concat` adaptor takes a variable number of arguments, it would not make sense to have a `|`-operator. But if we only want to prepend a single element, it very much does make sense. 

#### View Adaptors from Scratch

If neither of the two options above suit your needs, there is a third way to implement your custom view. That is, writing the view adaptor from scratch. While there are still some utilities in the range-v3 library that aid you along the process, this example is currently out of scope for this document. Maybe I will add such an example in the future. If you are still curious, I would recommend the following two sources:
- `interleave` view adaptor: In Eric Niebler's example on writing a CLI calender with range-v3, he implements the `interleave` view adaptor: https://github.com/ericniebler/range-v3/blob/master/example/calendar.cpp#L192. However, this view adaptor did not make it (yet?) to the official list of view adaptors, as he himself mentioned on Stackoverflow: https://stackoverflow.com/a/54025830


### Debugging

> [!NOTE] Debugging view
> Would be great to have a view that explains what is going on and prints some output information.

### `auto` Keyword is Your Friend

The concatenation of multiple view adaptors is encoded via the type.
Hence, it is almost impossible to use an explicit type for a range.
Use `auto` liberally - for declaring variables and as return type.

### Printing `std::vector` to a stream
If the contained type can be printed to a stream, e.g., most primitive types, ranges can easily printed via `std::cout << rng`.
To achieve the same for a `std::vector`, just wrap the vector in the `ranges::views::all` view adaptor: `std::cout << ranges::views::all(v)`.

### When to use `actions` vs `views`

As mentioned above, `actions` act eagerly on the underlying container while `views` are evaluated lazily. So, when would we want to use either one? I used to prefer `views` and almost rarely used `actions`. However, the following pattern occurred regularly:
```cpp
const auto tmp = computeSomething();
const auto vec = tmp | views::take(5) | ranges::to_vector;
```
Note that I needed to create the temporary variable `rng`. This is because `computeSomething()` is an r-value and thus cannot be piped into a view range adaptor. However, with `actions` this is not a problem:
```cpp
const auto vec = computeSomething() | actions::take(5);
```
Moreover, the following fails to compile:
```cpp
auto tmp = computeSomething();
const auto vec = tmp | actions::take(5);
```
This is because `tmp` would need to be *copied*, which can be inefficient. Hence, one can only pipe r-values into `actions`, so that the following code works:
```cpp
auto tmp = computeSomething();
const auto vec = std::move(tmp) | actions::take(5);
```
Note that `std::move` does nothing if the passed variable is declared `const`.

### Converting to Various Containers

- `std::vector`: `rng | ranges::to<std::vector<double>>` or `rng | ranges::to_vector` for short
- `std::set`: `rng = vec | ranges::views::unique; `
