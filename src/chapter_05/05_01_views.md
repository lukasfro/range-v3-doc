## Overview `range-v3/views`

### Range Adaptors

```cpp
using namespace ranges::views;

adjacent_filter(Rng<T>, f(T x, T y) -> bool) -> Rng<T>;
adjacent_remove_if(Rng<T>, f(T x, T y) -> bool) -> Rng<T>;
all(Rng<T>) -> Rng<T>;
c_str(std::string) -> Rng<char>;
cartesian_product(Rng<T>, Rng<S>, ...) -> Rng<tuple<T, S, ...>>;
chunk(Rng<T>, int) -> Rng<Rng<T>>;
chunk_by(Rng<T>, f(T x, T y) -> bool) -> Rng<Rng<T>>;
concat(Rng<T>, Rng<T>, ...) -> Rng<T>;
const_(Rng<T>) -> Rng<const T&>;
counted(It<T>, int) -> Rng<T>;
cycle(Rng<T>) -> Rng<T>;
drop(Rng<T>, int) -> Rng<T>;
drop_exactly(Rng<T>, int) -> Rng<T>;
drop_last(Rng<T>, int) -> Rng<T>;
drop_while(Rng<T>, f(T x) -> bool) -> Rng<T>;
enumerate(Rng<T>) -> Rng<pair<int, T>>;
exclusive_scan(Rng<T>, T, f(T x, T y) -> S) -> Rng<S>;
filter(Rng<T>, f(T x) -> bool) -> Rng<T>;
for_each(Rng<T>, f(T x));
getlines(std::ifstream) -> Rng<std::string>;
indirect(Rng<T*>) -> Rng<T>;
intersperse(Rng<T>, T) -> Rng<T>;
join(Rng<Rng<T>>) -> Rng<T>;
keys(std::map<T, U>>) -> Rng<T>;
partial_sum(Rng<T>) -> Rng<T>;
remove(Rng<T>, T) -> Rng<T>;
remove_if(Rng<T>, f(T x) -> bool) -> Rng<T>;
replace(Rng<T>, T, T) -> Rng<T>;
replace_if(Rng<T>, f(T x) -> bool, T) -> Rng<T>;
reverse(Rng<T>) -> Rng<T>;
sample(Rng<T>, int, std::default_random_engine) -> Rng<T>;
set_difference(Rng<T>, Rng<T>) -> Rng<T>;
set_intersection(Rng<T>, Rng<T>) -> Rng<T>;
set_symmetric_difference(Rng<T>, Rng<T>) -> Rng<T>;
set_union(Rng<T>, Rng<T>) -> Rng<T>;
single(T) -> Rng<T>;
slice(Rng<T>, int, int) -> Rng<T>;
sliding(Rng<T>, int) -> Rng<Rng<T>>;
split(Rng<T>, T) -> Rng<Rng<T>>;
split_when(Rng<T>, f(T x) -> bool) -> Rng<Rng<T>>;
stride(Rng<T>, int) -> Rng<T>;
ranges::subrange(Rng<T>) -> pair<T*, T*>;
tail(Rng<T>) -> Rng<T>;
take(Rng<T>, int) -> Rng<T>;
take_exactly(Rng<T>, int) -> Rng<T>;
take_last(Rng<T>, int) -> Rng<T>;
take_while(Rng<T>, f(T x) -> bool) -> Rng<T>;
tokenize(std::string, std::regex) -> Rng<std::string>;
transform(Rng<T>, f(T x) -> U) -> Rng<U>;
trim(f(T x) -> bool) -> Rng<T>;
unbounded(T*) -> Rng<T>;
unique(Rng<T>) -> Rng<T>);
values(std::map<T, U>>) -> Rng<U>;
zip(Rng<T>, Rng<U>, ...) -> Rng<tuple<T, U, ...>>;
zip_with(f(T x, U y, ...) -> S, Rng<T>, Rng<U>, ...) -> Rng<S>
```

### Range Factories
```cpp
using namespace ranges::views;

empty() -> Rng<T>;
generate(f() -> T) -> Rng<T>;
generate_n(f() -> T, int) -> Rng<T>;
indices(int [, int]) -> Rng<int>;
iota(int [, int]) -> Rng<int>;
closed_iota(int, int) -> Rng<int>;
istream(std::istream, char) -> Rng<std::string>;
linear_distribute(double, double, int) -> Rng<double>;
repeat(T) -> Rng<T>;
repeat_n(T, int) -> Rng<T>;
single(T) -> Rng<T>;
ranges::subrange(T*, T*) -> Rng<T>;
```
