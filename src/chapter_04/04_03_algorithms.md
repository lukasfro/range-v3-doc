## Algorithms

### Any Of

```cpp
auto is_odd = [](int x) { return x % 2; };
const std::vector<int> vec{0, 2, 4, 5, 6};
const bool any_odd = ranges::any_of(vec, is_odd);
std::cout << (any_odd ? "yes" : "no") << std::endl;  // prints: yes
```
