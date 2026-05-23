# Arrays in Modern C++: From Raw Memory to `std::vector`, `std::span`, and C++26

## Opening Hook

Imagine you are standing in a library.

You have books scattered everywhere.

One book is on the table.

One is near the window.

One is on the floor.

One is inside someone’s bag.

Now suppose I ask you:

> “Give me the third book.”

That question is hard because the books are not arranged in order.

But now imagine the books are placed on a shelf.

Book 0, book 1, book 2, book 3, book 4.

Suddenly, “the third book” becomes easy.

That is the heart of an array.

An array is not merely a list of values.

An array is a promise:

> “I will keep values of the same type next to each other in memory.”

That promise is why arrays are fast.

That promise is why arrays are the foundation of vectors, strings, matrices, heaps, hash tables, dynamic programming tables, and almost every important data structure we will learn later.

But C++ makes this story especially interesting.

Because C++ does not give us only one kind of array.

It gives us an entire evolution of array-like tools.

We have the old raw array:

```cpp
int arr[5];
```

We have modern fixed-size arrays:

```cpp
std::array<int, 5> arr;
```

We have dynamic arrays:

```cpp
std::vector<int> v;
```

We have non-owning views:

```cpp
std::span<int> s;
```

We have modern algorithms and ranges:

```cpp
std::ranges::sort(v);
```

And in newer C++, we even get tools like `std::mdspan` and C++26’s `std::inplace_vector`, depending on compiler support.

So today, we are not solving array problems yet.

Today, we are learning the object that lives underneath those problems.

Because before we ask:

> “How do I solve Two Sum?”

we should ask:

> “What is this `nums` thing actually doing in memory?”

---

## Part 1: Why Do We Need Arrays?

Suppose we want to store five marks.

We could write:

```cpp
#include <iostream>
using namespace std;

int main() {
    int m1 = 90;
    int m2 = 85;
    int m3 = 78;
    int m4 = 92;
    int m5 = 88;

    cout << m1 << " " << m2 << " " << m3 << " " << m4 << " " << m5 << endl;
}
```

This works.

But it is clumsy.

If I ask you to find the maximum mark, you must compare five separate variables.

If I ask you to print all marks, you must manually mention all five names.

If I ask you to store five thousand marks, this approach collapses.

So instead of five separate boxes, we create one row of boxes.

```cpp
int marks[5] = {90, 85, 78, 92, 88};
```

Now the marks have one name:

```cpp
marks
```

And each value has a position:

```text
marks[0] = 90
marks[1] = 85
marks[2] = 78
marks[3] = 92
marks[4] = 88
```

This is the first major idea:

> An array lets us treat many values as one organized structure.

That is why arrays are the first real data structure.

Not because they are complicated.

But because they introduce structure.

---

## Part 2: The Memory Picture

Let us imagine memory as a long street.

Each house has an address.

A normal variable is like one house.

```cpp
int x = 10;
```

Somewhere in memory, one box stores `10`.

But an array is a row of neighboring houses.

```cpp
int arr[5] = {10, 20, 30, 40, 50};
```

In memory, it may look conceptually like this:

```text
Index:     0     1     2     3     4
Value:     10    20    30    40    50
Address:   A     A+4   A+8   A+12  A+16
```

If each `int` takes 4 bytes, then every next element is 4 bytes after the previous one.

This is the reason array access is fast.

When we write:

```cpp
arr[3]
```

the computer does not search from the beginning.

It calculates.

The formula is:

```text
address of arr[i] = base address + i × size of each element
```

So if the starting address is `1000`, and each integer takes 4 bytes:

```text
arr[0] -> 1000 + 0 × 4 = 1000
arr[1] -> 1000 + 1 × 4 = 1004
arr[2] -> 1000 + 2 × 4 = 1008
arr[3] -> 1000 + 3 × 4 = 1012
```

This is why indexing into an array is:

```text
O(1)
```

Constant time.

Not because the computer is magically fast.

Because the layout is predictable.

---

## Part 3: Why Indexing Starts from Zero

A beginner often asks:

> “Why does the first element have index 0? Why not 1?”

The answer is distance.

The index tells us:

> “How far should I move from the beginning?”

The first element is zero steps away from the beginning.

So it is index `0`.

The second element is one step away.

So it is index `1`.

The third element is two steps away.

So it is index `2`.

For:

```cpp
int arr[5] = {10, 20, 30, 40, 50};
```

we get:

```text
Index:  0   1   2   3   4
Value: 10  20  30  40  50
```

For an array of size `n`, valid indices are:

```text
0 to n - 1
```

So this loop is natural:

```cpp
for (int i = 0; i < n; i++) {
    cout << arr[i] << " ";
}
```

The condition is not:

```cpp
i <= n
```

It is:

```cpp
i < n
```

Because `n` itself is already outside the array.

This tiny detail will decide the correctness of hundreds of DSA problems.

---

## Part 4: Raw C-Style Arrays

The oldest array style in C++ is the C-style array.

```cpp
#include <iostream>
using namespace std;

int main() {
    int arr[5] = {10, 20, 30, 40, 50};

    cout << arr[0] << endl;
    cout << arr[1] << endl;
    cout << arr[2] << endl;
}
```

This is simple and fast.

But it is also dangerous.

Let us update an element:

```cpp
arr[2] = 100;
```

Now the array becomes:

```text
[10, 20, 100, 40, 50]
```

Full example:

```cpp
#include <iostream>
using namespace std;

int main() {
    int arr[5] = {10, 20, 30, 40, 50};

    arr[2] = 100;

    for (int i = 0; i < 5; i++) {
        cout << arr[i] << " ";
    }
}
```

Output:

```text
10 20 100 40 50
```

So far, everything looks fine.

But now comes the dark side.

---

## Part 5: The Danger of Out-of-Bounds Access

What happens here?

```cpp
#include <iostream>
using namespace std;

int main() {
    int arr[5] = {10, 20, 30, 40, 50};

    cout << arr[10] << endl;
}
```

There is no `arr[10]`.

Valid indices are:

```text
0, 1, 2, 3, 4
```

But C++ may still compile this.

At runtime, it may print garbage.

It may crash.

It may appear to work.

And that is the scary part.

This is called **undefined behavior**.

Undefined behavior means:

> The C++ standard no longer promises what your program will do.

This is why raw arrays are powerful but risky.

C++ gives you the steering wheel, the engine, and the road.

But it does not always put a guardrail at the cliff.

So as DSA students, we must develop index discipline.

Whenever you write a loop, ask:

```text
What is the first valid index?
What is the last valid index?
Can this loop ever reach an invalid index?
```

That habit alone prevents a huge number of bugs.

---

## Part 6: Finding the Length of a Raw Array

Inside the same scope, we can find the number of elements like this:

```cpp
#include <iostream>
using namespace std;

int main() {
    int arr[5] = {10, 20, 30, 40, 50};

    int n = sizeof(arr) / sizeof(arr[0]);

    cout << n << endl;
}
```

Output:

```text
5
```

Why does this work?

```cpp
sizeof(arr)
```

gives the total size of the array in bytes.

```cpp
sizeof(arr[0])
```

gives the size of one element.

So:

```text
number of elements = total bytes / bytes per element
```

But there is a trap.

When a raw array is passed to a function, it often decays into a pointer.

That means the function loses the size information.

Example:

```cpp
void print(int arr[]) {
    // Here arr behaves like a pointer.
}
```

This is why modern C++ prefers safer containers like `std::array`, `std::vector`, and views like `std::span`.

---

## Part 7: `std::array` — A Safer Fixed-Size Array

Now let us move from old C-style arrays to modern C++.

C++ gives us:

```cpp
std::array
```

To use it:

```cpp
#include <array>
#include <iostream>
using namespace std;

int main() {
    array<int, 5> arr = {10, 20, 30, 40, 50};

    cout << arr[0] << endl;
    cout << arr.size() << endl;
}
```

Output:

```text
10
5
```

This line:

```cpp
array<int, 5> arr;
```

means:

```text
A fixed-size array of 5 integers.
```

The size is known.

The array behaves like a proper object.

It has useful functions:

```cpp
arr.size()
arr.front()
arr.back()
arr.fill(value)
arr.at(index)
```

Example:

```cpp
#include <array>
#include <iostream>
using namespace std;

int main() {
    array<int, 5> arr = {10, 20, 30, 40, 50};

    cout << "Size: " << arr.size() << endl;
    cout << "First: " << arr.front() << endl;
    cout << "Last: " << arr.back() << endl;
}
```

Output:

```text
Size: 5
First: 10
Last: 50
```

This is much nicer than raw arrays.

---

## Part 8: `arr[index]` vs `arr.at(index)`

With `std::array`, we can access like this:

```cpp
arr[2]
```

But there is also:

```cpp
arr.at(2)
```

The difference is safety.

```cpp
arr[2]
```

is fast and does not check bounds in the same way.

```cpp
arr.at(2)
```

performs bounds checking and throws an exception if the index is invalid.

Example:

```cpp
#include <array>
#include <iostream>
using namespace std;

int main() {
    array<int, 5> arr = {10, 20, 30, 40, 50};

    cout << arr.at(2) << endl;
}
```

Output:

```text
30
```

But:

```cpp
cout << arr.at(10) << endl;
```

throws an exception.

So the beginner rule is:

```text
Use [] when you are sure the index is valid.
Use .at() when you want checked access.
```

For DSA contests, people often use `[]` for speed and brevity.

For learning and debugging, `.at()` can be very helpful.

---

## Part 9: Traversing `std::array`

We can use an index loop:

```cpp
#include <array>
#include <iostream>
using namespace std;

int main() {
    array<int, 5> arr = {10, 20, 30, 40, 50};

    for (int i = 0; i < arr.size(); i++) {
        cout << arr[i] << " ";
    }
}
```

But there is a small type detail.

`arr.size()` returns an unsigned type, usually `size_t`.

So many modern C++ programmers write:

```cpp
for (size_t i = 0; i < arr.size(); i++) {
    cout << arr[i] << " ";
}
```

Or even better, if we do not need the index:

```cpp
for (int value : arr) {
    cout << value << " ";
}
```

This is called a range-based for loop.

It means:

> For each value in the array, do something.

If we want to modify values:

```cpp
for (int& value : arr) {
    value *= 2;
}
```

The `&` is important.

Without `&`, `value` is a copy.

With `&`, `value` refers to the actual element.

Example:

```cpp
#include <array>
#include <iostream>
using namespace std;

int main() {
    array<int, 5> arr = {1, 2, 3, 4, 5};

    for (int& value : arr) {
        value *= 2;
    }

    for (int value : arr) {
        cout << value << " ";
    }
}
```

Output:

```text
2 4 6 8 10
```

This is a crucial C++ habit:

```cpp
int value
```

means copy.

```cpp
int& value
```

means reference.

```cpp
const int& value
```

means read-only reference.

For large objects, use `const auto&`.

For integers, copying is fine.

---

## Part 10: `std::vector` — The DSA Workhorse

Now we come to the most important array-like container for DSA:

```cpp
std::vector
```

A vector is a dynamic array.

Unlike `std::array`, it can grow.

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> nums;

    nums.push_back(10);
    nums.push_back(20);
    nums.push_back(30);

    for (int x : nums) {
        cout << x << " ";
    }
}
```

Output:

```text
10 20 30
```

This is the container you will use constantly in DSA.

LeetCode-style function?

```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    // solution
}
```

Graph adjacency list?

```cpp
vector<vector<int>> graph;
```

Dynamic programming table?

```cpp
vector<int> dp(n);
```

Matrix?

```cpp
vector<vector<int>> matrix;
```

Frequency array?

```cpp
vector<int> freq(26, 0);
```

So although this video is about arrays, in practical C++ DSA, `std::vector` is the star.

---

## Part 11: Creating Vectors

Empty vector:

```cpp
vector<int> v;
```

Vector of size 5, filled with zeroes:

```cpp
vector<int> v(5);
```

Vector of size 5, filled with `-1`:

```cpp
vector<int> v(5, -1);
```

Vector with given values:

```cpp
vector<int> v = {10, 20, 30, 40, 50};
```

2D vector:

```cpp
vector<vector<int>> grid(3, vector<int>(4, 0));
```

This creates a `3 × 4` grid filled with zeroes.

Conceptually:

```text
0 0 0 0
0 0 0 0
0 0 0 0
```

This is very useful for matrix problems and dynamic programming.

---

## Part 12: Vector Operations

Common operations:

```cpp
v.push_back(x);   // add to end
v.pop_back();     // remove from end
v.size();         // number of elements
v.empty();        // true if size is zero
v.front();        // first element
v.back();         // last element
v.clear();        // remove all elements
```

Example:

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v = {10, 20, 30};

    v.push_back(40);
    v.push_back(50);

    cout << "Size: " << v.size() << endl;
    cout << "First: " << v.front() << endl;
    cout << "Last: " << v.back() << endl;

    v.pop_back();

    for (int x : v) {
        cout << x << " ";
    }
}
```

Output:

```text
Size: 5
First: 10
Last: 50
10 20 30 40
```

---

## Part 13: `size()` and the Signed/Unsigned Trap

This is a subtle but important C++ lesson.

`v.size()` returns `size_t`, which is unsigned.

That means this can be dangerous:

```cpp
for (int i = v.size() - 1; i >= 0; i--) {
    cout << v[i] << " ";
}
```

Why?

If `v.size()` is `0`, then `v.size() - 1` underflows because unsigned integers cannot represent `-1`.

A safer reverse loop is:

```cpp
for (int i = static_cast<int>(v.size()) - 1; i >= 0; i--) {
    cout << v[i] << " ";
}
```

But only if the vector size fits in `int`.

Another modern style:

```cpp
for (auto it = v.rbegin(); it != v.rend(); ++it) {
    cout << *it << " ";
}
```

Or use ranges in modern C++, which we will discuss soon.

This is one reason DSA in C++ needs both algorithmic thinking and language discipline.

---

## Part 14: Vector Capacity

A vector has size and capacity.

```cpp
vector<int> v;
```

The size is how many elements are currently inside.

The capacity is how many elements the vector can hold before it needs to reallocate.

Example:

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v;

    for (int i = 1; i <= 10; i++) {
        v.push_back(i);
        cout << "Size: " << v.size()
             << ", Capacity: " << v.capacity() << endl;
    }
}
```

The output depends on the implementation, but you may see capacity grow like:

```text
Size: 1, Capacity: 1
Size: 2, Capacity: 2
Size: 3, Capacity: 4
Size: 4, Capacity: 4
Size: 5, Capacity: 8
...
```

This tells us something deep.

A vector does not allocate memory one element at a time forever.

When it runs out of space, it usually allocates a bigger block and moves or copies elements there.

That is why `push_back` is usually amortized `O(1)`.

Most insertions are cheap.

Occasionally, one insertion triggers reallocation and becomes expensive.

But averaged over many insertions, it is efficient.

---

## Part 15: Reserving Capacity

If we know we will insert many elements, we can reserve space.

```cpp
vector<int> v;
v.reserve(1000);
```

This says:

> Please allocate enough capacity for 1000 elements.

It does not create 1000 elements.

It only reserves memory.

Compare:

```cpp
vector<int> a(1000);
```

This creates 1000 integers.

But:

```cpp
vector<int> b;
b.reserve(1000);
```

This creates zero elements but reserves space for future growth.

This is useful for performance.

In DSA, if we know the input size, reserving can avoid repeated reallocations.

Example:

```cpp
int n;
cin >> n;

vector<int> nums;
nums.reserve(n);

for (int i = 0; i < n; i++) {
    int x;
    cin >> x;
    nums.push_back(x);
}
```

This is a professional habit.

---

## Part 16: Passing Arrays and Vectors to Functions

Raw arrays are tricky.

This:

```cpp
void print(int arr[]) {
    // arr behaves like int*
}
```

does not know the size.

So we often need:

```cpp
void print(int arr[], int n) {
    for (int i = 0; i < n; i++) {
        cout << arr[i] << " ";
    }
}
```

But with `vector`, we can do better.

If we only read:

```cpp
void print(const vector<int>& nums) {
    for (int x : nums) {
        cout << x << " ";
    }
}
```

Why `const vector<int>&`?

Because:

```cpp
vector<int> nums
```

would copy the whole vector.

But:

```cpp
const vector<int>& nums
```

borrows it without copying and promises not to change it.

If we want to modify:

```cpp
void doubleValues(vector<int>& nums) {
    for (int& x : nums) {
        x *= 2;
    }
}
```

So the rule is:

```text
Read only:       const vector<int>&
Modify:          vector<int>&
Need a copy:     vector<int>
```

This is one of the most important C++ DSA function habits.

---

## Part 17: `std::span` — A Modern View Over Arrays

Now let us introduce a C++20 feature that students should know:

```cpp
std::span
```

`std::span` is a non-owning view over contiguous elements. It can refer to a raw array, `std::array`, or `std::vector`.

Think of it like this:

> A vector owns data.  
> An array owns data.  
> A span only views data.

Example:

```cpp
#include <iostream>
#include <span>
#include <vector>
#include <array>
using namespace std;

void print(span<int> nums) {
    for (int x : nums) {
        cout << x << " ";
    }
    cout << endl;
}

int main() {
    int raw[] = {1, 2, 3};
    array<int, 3> arr = {4, 5, 6};
    vector<int> vec = {7, 8, 9};

    print(raw);
    print(arr);
    print(vec);
}
```

This is beautiful.

One function can accept many kinds of contiguous data.

Raw array?

Yes.

`std::array`?

Yes.

`std::vector`?

Yes.

That is the power of `std::span`.

For DSA teaching, this is useful because students often write functions that should not care whether the input came from a vector or an array.

---

## Part 18: Read-Only Span

If the function should not modify data, use:

```cpp
span<const int>
```

Example:

```cpp
#include <iostream>
#include <span>
#include <vector>
using namespace std;

int sum(span<const int> nums) {
    int total = 0;

    for (int x : nums) {
        total += x;
    }

    return total;
}

int main() {
    vector<int> v = {10, 20, 30};

    cout << sum(v) << endl;
}
```

Output:

```text
60
```

This is a modern alternative to writing many overloads.

Instead of:

```cpp
int sum(vector<int>& v);
int sum(array<int, 5>& a);
int sum(int arr[], int n);
```

we can write:

```cpp
int sum(span<const int> nums);
```

This is clean, safe, and expressive.

For teaching DSA, `std::span` gives students an excellent mental model:

> “I do not own this array. I am just looking at it.”

---

## Part 19: `std::to_array` — Creating `std::array` Easily

C++20 also introduced `std::to_array`, which helps create `std::array` from a raw array-like initializer.

Example:

```cpp
#include <array>
#include <iostream>
using namespace std;

int main() {
    auto arr = to_array({10, 20, 30, 40, 50});

    cout << arr.size() << endl;

    for (int x : arr) {
        cout << x << " ";
    }
}
```

This is useful because we do not have to write:

```cpp
array<int, 5> arr = {10, 20, 30, 40, 50};
```

The compiler can infer the type and size.

This is not something beginners must use immediately, but it is good to recognize.

---

## Part 20: C++20 Ranges — Treat the Whole Sequence as One Thing

Now let us talk about one of the most important modern C++ ideas:

```cpp
ranges
```

Before ranges, algorithms often looked like this:

```cpp
sort(v.begin(), v.end());
```

This is already good.

But it exposes the machinery:

```text
begin iterator
end iterator
```

With C++20 ranges, we can write:

```cpp
ranges::sort(v);
```

The sequence is treated as one object.

Example:

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v = {5, 2, 9, 1, 3};

    ranges::sort(v);

    for (int x : v) {
        cout << x << " ";
    }
}
```

Output:

```text
1 2 3 5 9
```

For students, this is a beautiful mental shift.

Instead of saying:

> “Sort from this iterator to that iterator.”

we say:

> “Sort this collection.”

That is closer to how humans think.

---

## Part 21: Ranges with Views

Ranges become even more interesting with views.

A view is a lazy way of looking at data.

Example:

```cpp
#include <iostream>
#include <ranges>
#include <vector>
using namespace std;

int main() {
    vector<int> nums = {1, 2, 3, 4, 5, 6};

    auto even = nums | views::filter([](int x) {
        return x % 2 == 0;
    });

    for (int x : even) {
        cout << x << " ";
    }
}
```

Output:

```text
2 4 6
```

This does not immediately create a new vector.

It creates a view that filters while we iterate.

Another example:

```cpp
auto squares = nums | views::transform([](int x) {
    return x * x;
});
```

Now we can print squares:

```cpp
for (int x : squares) {
    cout << x << " ";
}
```

Output:

```text
1 4 9 16 25 36
```

This style feels very high-level, but it still keeps C++’s performance spirit.

For DSA, I would not recommend beginners use ranges everywhere on day one.

But they should see them.

Because modern C++ is moving toward clearer expression.

---

## Part 22: `std::print` in C++23

Traditionally, C++ output uses:

```cpp
cout << x << endl;
```

C++23 introduced `std::print`, which gives a simpler formatted printing style.

Example:

```cpp
#include <print>
#include <vector>
using namespace std;

int main() {
    vector<int> nums = {10, 20, 30};

    for (int x : nums) {
        print("{} ", x);
    }
}
```

This looks like Python-style or Rust-style formatting.

However, students must know:

> `std::print` support depends on compiler and standard library version.

So for maximum compatibility today, many DSA students still use:

```cpp
cout
```

But it is useful to know the direction modern C++ is taking.

Old style:

```cpp
cout << "Value: " << x << endl;
```

Newer style:

```cpp
print("Value: {}\n", x);
```

For teaching, you can say:

> We will mostly use `cout` for compatibility, but modern C++ is gaining cleaner printing tools.

---

## Part 23: `std::mdspan` — A Modern View for Matrices

In DSA, arrays are not always one-dimensional.

We often need matrices:

```text
grid problems
dynamic programming tables
image processing
graph adjacency matrices
```

Traditionally, beginners write:

```cpp
vector<vector<int>> matrix;
```

That works, but it does not always mean the entire matrix is stored as one continuous block.

A more performance-oriented approach is to store data in one vector:

```cpp
vector<int> data(rows * cols);
```

Then access:

```cpp
data[r * cols + c]
```

This is fast but ugly.

C++23 introduced `std::mdspan`, a non-owning multidimensional view over contiguous data. It lets us treat flat memory as a multidimensional structure.

Conceptual example:

```cpp
#include <mdspan>
#include <vector>
using namespace std;

int main() {
    vector<int> data(3 * 4);

    mdspan matrix(data.data(), 3, 4);

    matrix(1, 2) = 99;
}
```

The idea is:

```text
data is still flat memory.
mdspan gives us matrix-like indexing.
```

So instead of:

```cpp
data[1 * 4 + 2] = 99;
```

we can write:

```cpp
matrix(1, 2) = 99;
```

This is powerful for advanced DSA, numerical computing, and performance-sensitive code.

Again, support depends on compiler and standard library version.

But students should know the concept:

> A multidimensional array can be represented as one flat contiguous block plus a smart indexing view.

That idea is valuable even if they do not use `mdspan` immediately.

---

## Part 24: C++26 `std::inplace_vector`

Now let us look at a future-facing feature: `std::inplace_vector`.

C++26 includes `std::inplace_vector`, a dynamically-resizable vector with fixed capacity.

Think of three containers:

```text
std::array
Fixed size.
All elements exist from the beginning.

std::vector
Dynamic size.
Can grow by allocating memory on the heap.

std::inplace_vector
Dynamic size up to a fixed capacity.
Storage is inside the object itself.
```

Conceptually:

```cpp
#include <inplace_vector>
using namespace std;

int main() {
    inplace_vector<int, 10> v;

    v.push_back(1);
    v.push_back(2);
    v.push_back(3);
}
```

This means:

> This vector can grow, but only up to capacity 10.

Why is this interesting?

Because sometimes in DSA or systems programming, we know a maximum size.

For example:

```text
At most 4 directions in a grid
At most 26 lowercase letters
At most 8 chess moves for a knight
At most 100 small temporary values
```

A normal vector may allocate dynamically.

An array forces all elements to exist.

An `inplace_vector` gives a middle path:

```text
Fixed capacity, variable size.
```

That is a beautiful idea.

For students, the lesson is not:

> “Use this everywhere immediately.”

The lesson is:

> Modern C++ is giving us more precise containers for more precise intent.

Use:

```cpp
array<int, 5>
```

when you always have exactly 5 elements.

Use:

```cpp
vector<int>
```

when the size can grow freely.

Use:

```cpp
inplace_vector<int, 10>
```

when the size changes but the maximum capacity is known.

---

## Part 25: `std::span` and C++26 Deduction Improvements

C++26 also improves class template argument deduction for tools like `std::span` and `std::mdspan` with integral constants.

This is not a beginner-level feature, but the direction matters.

Modern C++ wants code like this to become easier:

```cpp
std::span s = arr;
```

instead of forcing us to repeatedly spell out complex template types.

For students, you can explain it simply:

> New C++ standards often do not only add big new tools.  
> They also polish existing tools so code becomes easier to write correctly.

---

## Part 26: Choosing the Right C++ Array Tool

Now let us create a simple decision table.

```text
Raw array: int arr[5]
Use when learning memory basics or working with low-level APIs.

std::array<int, 5>
Use when size is fixed and known at compile time.

std::vector<int>
Use for most DSA problems where size changes or comes from input.

std::span<int>
Use when a function should view contiguous data without owning it.

std::mdspan
Use for matrix-like views over flat contiguous memory.

std::inplace_vector<int, N>
Use when size changes but maximum capacity is fixed.
```

For beginners in DSA, the practical rule is:

```text
Learn raw arrays to understand memory.
Use std::vector for most problem solving.
Use std::array when the size is fixed.
Learn std::span to write clean modern functions.
```

---

## Part 27: Mini Program 1 — Basic Raw Array

```cpp
#include <iostream>
using namespace std;

int main() {
    int arr[5] = {10, 20, 30, 40, 50};

    cout << "Original array: ";

    for (int i = 0; i < 5; i++) {
        cout << arr[i] << " ";
    }

    cout << "\nElement at index 2: " << arr[2] << endl;

    arr[2] = 100;

    cout << "After update: ";

    for (int i = 0; i < 5; i++) {
        cout << arr[i] << " ";
    }

    cout << endl;
}
```

This teaches:

```text
creation
indexing
updating
traversal
```

But then say:

> This is the memory model. Now let us write modern C++.

---

## Part 28: Mini Program 2 — `std::array`

```cpp
#include <array>
#include <iostream>
using namespace std;

int main() {
    array<int, 5> arr = {10, 20, 30, 40, 50};

    cout << "Size: " << arr.size() << endl;
    cout << "First: " << arr.front() << endl;
    cout << "Last: " << arr.back() << endl;

    arr.at(2) = 100;

    for (int value : arr) {
        cout << value << " ";
    }

    cout << endl;
}
```

This teaches:

```text
fixed-size modern array
size()
front()
back()
at()
range-based loop
```

---

## Part 29: Mini Program 3 — `std::vector`

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> nums;

    nums.reserve(5);

    nums.push_back(10);
    nums.push_back(20);
    nums.push_back(30);
    nums.push_back(40);
    nums.push_back(50);

    cout << "Size: " << nums.size() << endl;
    cout << "Capacity: " << nums.capacity() << endl;

    nums[2] = 100;

    for (int x : nums) {
        cout << x << " ";
    }

    cout << endl;
}
```

This teaches:

```text
dynamic growth
push_back
reserve
size
capacity
mutation
```

---

## Part 30: Mini Program 4 — Function with `const vector<int>&`

```cpp
#include <iostream>
#include <vector>
using namespace std;

int sum(const vector<int>& nums) {
    int total = 0;

    for (int x : nums) {
        total += x;
    }

    return total;
}

int main() {
    vector<int> nums = {10, 20, 30, 40, 50};

    cout << sum(nums) << endl;
}
```

This teaches:

```text
avoid copying
read-only function parameter
basic DSA accumulation pattern
```

---

## Part 31: Mini Program 5 — Function with `std::span`

```cpp
#include <array>
#include <iostream>
#include <span>
#include <vector>
using namespace std;

int sum(span<const int> nums) {
    int total = 0;

    for (int x : nums) {
        total += x;
    }

    return total;
}

int main() {
    int raw[] = {1, 2, 3};
    array<int, 3> arr = {4, 5, 6};
    vector<int> vec = {7, 8, 9};

    cout << sum(raw) << endl;
    cout << sum(arr) << endl;
    cout << sum(vec) << endl;
}
```

This teaches:

```text
one function
many contiguous containers
non-owning view
modern C++20 style
```

---

## Part 32: Mini Program 6 — Ranges

```cpp
#include <algorithm>
#include <iostream>
#include <ranges>
#include <vector>
using namespace std;

int main() {
    vector<int> nums = {5, 2, 9, 1, 3};

    ranges::sort(nums);

    for (int x : nums) {
        cout << x << " ";
    }

    cout << endl;

    auto even = nums | views::filter([](int x) {
        return x % 2 == 0;
    });

    for (int x : even) {
        cout << x << " ";
    }

    cout << endl;
}
```

This teaches:

```text
ranges::sort
views::filter
modern expressive traversal
```

---

## Part 33: DSA Intuition — Access, Insert, Delete

Now let us connect this to algorithmic thinking.

Arrays and vectors are excellent for access.

If I know the index:

```cpp
nums[i]
```

I can access it in:

```text
O(1)
```

But inserting in the middle is expensive.

Suppose:

```text
[10, 20, 30, 40, 50]
```

Now insert `99` at index `2`:

```text
[10, 20, 99, 30, 40, 50]
```

To make space, `30`, `40`, and `50` must shift right.

So middle insertion is:

```text
O(n)
```

Deleting from the middle is also:

```text
O(n)
```

because elements must shift left.

So the tradeoff is:

```text
Fast access.
Slow middle insertion/deletion.
Great cache locality.
Simple memory layout.
```

This is why arrays and vectors are so popular.

They are not perfect for everything.

But they are very fast for many real problems because memory is contiguous.

The CPU loves predictable memory.

---

## Part 34: Common Beginner Mistakes

### Mistake 1: Going Out of Bounds

Wrong:

```cpp
for (int i = 0; i <= n; i++) {
    cout << nums[i] << " ";
}
```

Correct:

```cpp
for (int i = 0; i < n; i++) {
    cout << nums[i] << " ";
}
```

### Mistake 2: Copying a Vector Accidentally

Bad for large vectors:

```cpp
void print(vector<int> nums) {
    // copies entire vector
}
```

Better:

```cpp
void print(const vector<int>& nums) {
    // no copy
}
```

### Mistake 3: Forgetting `&` When Modifying

This does not modify the original elements:

```cpp
for (int x : nums) {
    x *= 2;
}
```

This does:

```cpp
for (int& x : nums) {
    x *= 2;
}
```

### Mistake 4: Confusing Size and Capacity

```cpp
v.reserve(100);
```

does not create 100 elements.

It only reserves memory.

This is invalid after only reserve:

```cpp
vector<int> v;
v.reserve(100);

v[0] = 10; // wrong: size is still 0
```

Correct:

```cpp
vector<int> v(100);
v[0] = 10;
```

or:

```cpp
vector<int> v;
v.reserve(100);
v.push_back(10);
```

---

## Part 35: The Big Mental Model

Let us summarize the C++ array world.

A raw array teaches memory.

```cpp
int arr[5];
```

A `std::array` teaches fixed-size safety.

```cpp
array<int, 5> arr;
```

A `std::vector` teaches dynamic arrays.

```cpp
vector<int> nums;
```

A `std::span` teaches views.

```cpp
span<int> s;
```

Ranges teach expressive algorithms.

```cpp
ranges::sort(nums);
```

`mdspan` teaches multidimensional views.

```cpp
mdspan matrix(data.data(), rows, cols);
```

`inplace_vector` teaches fixed-capacity dynamic storage.

```cpp
inplace_vector<int, 10> v;
```

The deeper lesson is:

> Modern C++ is not just about writing faster code.  
> It is about expressing ownership, size, mutability, and structure more clearly.

Raw arrays say:

> “Here is memory. Be careful.”

`std::array` says:

> “Here is a fixed-size array object.”

`std::vector` says:

> “Here is a growable contiguous sequence.”

`std::span` says:

> “I do not own the data. I am just viewing it.”

`std::mdspan` says:

> “This flat memory can be viewed as a matrix.”

`std::inplace_vector` says:

> “I can grow, but only up to a known limit.”

Each tool says something different.

And good C++ programmers choose the tool whose meaning matches the problem.

---

## Closing

Today we learned arrays in C++ not as syntax, but as a story.

The story begins with raw memory:

```cpp
int arr[5];
```

Then it becomes safer:

```cpp
std::array<int, 5>
```

Then it becomes flexible:

```cpp
std::vector<int>
```

Then it becomes more expressive:

```cpp
std::span<int>
std::ranges
std::mdspan
std::inplace_vector
```

For DSA, your main tool will be:

```cpp
std::vector
```

But your understanding should go deeper.

Because when you understand arrays, you understand contiguous memory.

When you understand contiguous memory, you understand why indexing is fast.

When you understand why indexing is fast, you understand why vectors, strings, heaps, hash tables, and dynamic programming tables work the way they do.

Arrays are not just the first topic.

They are the first window into performance.

And in C++, performance is never far from memory.

So before we solve array problems, remember this:

> An array is a row of memory.  
> A vector is a row that can grow.  
> A span is a window into a row.  
> And DSA begins when we learn how to move through that row intelligently.

In the next video, we can start solving simple array problems using `std::vector`: sum, maximum, reverse, frequency count, prefix sums, and two pointers.
