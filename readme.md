# C++ STL Iterators

One of the keys to understanding how to use the containers of C++ Standard Template Library (STL) is understanding how iterators work. Containers such as `list`s and `map`s do not behave like arrays, so you can't use a `for` loop to go through the elements in them. Likewise, because these containers are not accessible randomly, you cannot use a simple integer index. You can use iterators to refer to elements of a container.

> The reason that STL containers and algorithms work so well together, is that they know nothing of each other - Alex Stepanov

Iterators are pointer-like objects that allow programs to step through the elements of a container sequentially without exposing the underlying representation. Iterators can be advanced from one element to the next by incrementing and decrementing them. Each container type has a distinct iterator associated with it. For instance, the iterator for `list<int>` is declared as:

```C++
 std::list<int>::iterator
```
### Iterator Category

Iterators falls into categories because different algorithms require different iterators to be used.  For example, the `std::copy()` algorithm needs an iterator that can be advanced by incrementing it, whereas the 	`std::reverse()` algorithm needs an iterator that can be decremented as well. In C++ language, the standard defines five different categories.

- **Input Iterator**
  - Read-only and can be read only once.
  - Example: `std::istream_iterator(istream& is)`
- **Output Iterator**
  - Write-only
  - Example: `std::ostream_iterator<int> out_it (std::cout,", ")`;
- **Forward Iterator**
  - Gathering input + output iterators
  - Example: `std::forward_list::iterator`, `std::unordered_map::iterator`
- **Bidirectional Iterator**
  - Like Forward Iterator, but also has `operator–`
  - Example: `std::list::iterator`
- **Random Access Iterator**
  - Has overloaded `operator[]`, pointer arithmetic
  - Example: `std::vector<int>::iterator`.

You can get more information about these [here](https://www.math.hkbu.edu.hk/parallel/pgi/doc/pgC++_lib/stdlibug/var_0565.htm)

### Iterator traits

Iterator traits allows algorithms to access information about a particular iterator in a uniform way to avoid re-implement of all iterators for each specific case, when it needs to traverse different style of containers. For example, Finding elements in `std::list` is `O(n)` complexity whereas in `std::vector` the random access to an element is `O(1)` complexity (given the index position). It is better for an algorithm to know that the container can be traversed using `+=` operator (Random Access), or only `++` operator (Forward), to choose what is the better choice to reduce the complexity of the algorithm that is computed.

The iterator traits are the following ones:
- `difference_type`:
	- the type for representing iterator distances
	- The type of iterator difference `p2 - p1`.
- `value_type`:
    - the type of the value that iterator points
- `pointer`: 
    - the pointer value that iterator points
	- usually `value_type*`
- `reference`: 
    - the reference value that iterator points
	- usually `value_type&`
- `iterator category`: 
    - Identifies the iterator concept modeled by the iterator.
	- one of the following ones: 
	```C++
    struct input_iterator_tag {};
    struct output_iterator_tag {};
    struct forward_iterator_tag : input_iterator_tag {};
    struct bidirectional_iterator_tag : forward_iterator_tag {};
    struct random_access_iterator_tag : bidirectional_iterator_tag {};
   ```

The definition of `iterator_traits` looks like:

```C++
// The basic version works for iterators with the member type
template <class Iterator>
struct iterator_traits
{
    typedef typename Iterator::value_type value_type;
    typedef typename Iterator::difference_type difference_type;
    typedef typename Iterator::pointer pointer;
    typedef typename Iterator::reference reference;
    typedef typename Iterator::iterator_category iterator_category;
};

// A partial specialization takes care of pointer types
template <class T>
struct iterator_traits<T *>
{
    typedef T value_type;
    typedef ptrdiff_t difference_type;
    typedef T *pointer;
    typedef T &reference;
    typedef random_access_iterator_tag iterator_category;
};

// pointers to const type
template <class T>
struct iterator_traits<const T *>
{
    typedef T value_type;
    typedef ptrdiff_t difference_type;
    typedef const T *pointer;
    typedef const T &reference;
    typedef random_access_iterator_tag iterator_category;
};
```

Sometimes a generic algorithm needs to know the value type of its iterator arguments, i.e., the type pointed to by the iterators. For example, to swap the values pointed by two iterators, a temporary variable is needed.

```C++
template <class Iterator>
void swap (Iterator a, Iterator b) 
{
  typename Iterator::value_type tmp = *a;
  *a = *b;
  *b = tmp;
}
```
The traits also improve the efficiency of algorithms by making use of knowledge about basic iterator categories provided by the `iterator_category` member. An algorithm can use this "tag" to select the most efficient implementation an iterator is capable of handling  without compromising the flexibility to deal with a variety of iterator types.

In the below example we aim to have a single `advance` algorithm that automatically executes the right version based on the iterator category.

```C++
template <class InputIterator, class Distance>
void advance(InputIterator &i, Distance n,
             input_iterator_tag)
{
    for (; n > 0; --n)
        ++i;
}

template <class BidirectionalIterator, class Distance>
void advance(BidirectionalIterator &i, Distance n
                                           bidirectional_iterator_tag)
{
    if (n <= 0)
        for (; n > 0; --n)
            ++i;
    else
        for (; n < 0; ++n)
            --i;
}

template <class RandomAccessIterator, class Distance>
void advance(RandomAccessIterator &i, Distance n,
             random_access_iterator_tag)
{
    i += n;
}

// Generic advance algorithm using compile-time dispatching based on function overloading
template <class InputIterator, class Distance>
void advance(InputIterator i, Distance n)
{
    advance(i, n, typename iterator_traits<Iterator>::iterator_category());
}
```

### Writing Custom iterator
Iterator traits will automatically work for any iterator class that defines the appropriate member types. The Custom iterator should support following pointers:
- How to retrieve the value at the point
- How the point of iteration can be incremented/decremented
- How to compare it with other points of iteration

```C++
#include <algorithm>
#include <exception>
#include <iostream>
#include <iterator>
#include <typeinfo>
#include <vector>

template <typename ArrType> 
class MyArray {
private:
  ArrType *m_data;
  unsigned int m_size;

public:
  class Iterator {
  public:
    // iterator_trait associated types
    typedef Iterator itr_type;
    typedef ArrType value_type;
    typedef ArrType &reference;
    typedef ArrType *pointer;
    typedef std::bidirectional_iterator_tag iterator_category;
    typedef std::ptrdiff_t difference_type;

    Iterator(pointer ptr) : m_itr_ptr(ptr) {}
    itr_type operator++() {
      itr_type old_itr = *this;
      m_itr_ptr++;
      return old_itr;
    }

    itr_type operator++(int dummy) {
      m_itr_ptr++;
      return *this;
    }

    itr_type operator--() {
      itr_type old_itr = *this;
      m_itr_ptr--;
      return old_itr;
    }

    itr_type operator--(int dummy) {
      m_itr_ptr--;
      return *this;
    }

    reference operator*() const { return *m_itr_ptr; }

    pointer operator->() const { return m_itr_ptr; }

    bool operator==(const itr_type &rhs) { return m_itr_ptr == rhs.m_itr_ptr; }

    bool operator!=(const itr_type &rhs) { return m_itr_ptr != rhs.m_itr_ptr; }

  private:
    pointer m_itr_ptr;
  };

  MyArray(unsigned int size) : m_size(size) { m_data = new ArrType[m_size]; }

  unsigned int size() const { return m_size; }

  ArrType &operator[](unsigned int idx) {
    if (idx >= m_size)
      throw std::runtime_error("Index out of range");
    return m_data[idx];
  }

  Iterator begin() { return Iterator(m_data); }

  Iterator end() { return Iterator(m_data + m_size); }
};

int main()
{
  MyArray<double> arr(3);
  arr[0] = 2.6;
  arr[1] = 5.2;
  arr[2] = 8.9;

  std::cout << "MyArray Contents: ";
  for (MyArray<double>::Iterator it = arr.begin(); it != arr.end(); it++) {
    std::cout << *it << " ";
  }

  std::cout << std::endl;

  std::vector<double> vec;
  std::copy(arr.begin(), arr.end(), std::back_inserter(vec));

  std::cout << "Vector Contents after copy: ";
  for (std::vector<double>::iterator it = vec.begin(); it != vec.end(); it++) {
    std::cout << *it << " ";
  }

  std::cout << std::endl;

  std::cout << typeid(std::iterator_traits<
                          MyArray<double>::Iterator>::iterator_category())
                   .name()
            << std::endl;
  return 0;
}

/*OUTPUT
MyArray Contents: 2.6 5.2 8.9 
Vector Contents after copy: 2.6 5.2 8.9 
FSt26bidirectional_iterator_tagvE
*/
```

### Iterators and the Range for Loop
The range-based for loop (or range-for in short), along with `auto`, is one of the most significant features added in the C++11 standard.

The syntax template for the range `for` loop looks like this:

```C++
for (range_declaration : range_expression) 
{ 
    // loop body 
}
```

In C++11/C++14, the above format results in a code similar to the following: 

```C++
{  
  auto&& range = range_expression ; 
  // beginExpr is range.begin() and endExpr is range.end()
  for (auto b = beginExpr, e = endExpr; b != e; ++b) { 
    range_declaration = *b; 
    // loop body
  } 
} 
```
A typical usages of range-based for loop:

```C++
// Iterate over STL container
std::vector<int> v{1, 2, 3, 4};
for (const auto &i : v)
    std::cout << i << "\n";
``` 
The way the range for loop works is by creating an iterator that points to the first element of the vector and then accessing each element of the vector in turn until the iterator reaches the last element of the vector and then the loop terminates. This behavior can be observed in cppinsight [here](https://cppinsights.io/s/d73fd501).


