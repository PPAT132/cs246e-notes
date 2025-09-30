[Where do I even start << ](./problem_12.md) | [**Home**](../README.md) | [>> Less Copying](./problem_14.md) 

# Problem 13: Actually... I want a vector of Posns
## **2025-09-30**

```C++
struct Posn {
    int x, y;
    Posn(int x, int y): x{x}, y{y} {}
};

int main() {
    vector<Posn> v; // Won't compile, why not?
}
```

Take a look at Vector's constructor:

```C++
template<typename T> vector<T>::vector(): n{0}, cap{1}, theVector{new T[cap]} {}
```
`T[cap]` creates an array of T objects. Which `T` objects will be stored in the array?
- C++ always calls a constructor when creating an object.
- Which constructor gets called? The default constructor
- But `Posn` doesn't have one
- Creating a default constructor just to make the compiler happy is not always good. Sometimes we need to make the struct not initialized. 
- And now, essentially what we just did was to propagate this problem from compile time to run time, and this is not good.

Need to separate memory allocation (Object creation step 1) from initialization (steps 2-4) (refer to [p4](./problem_4.md))

**Allocation only:** `void* operator new(size_t)`
- Allocates `size_t` bytes
- No initialization
- Returns `void*`

**Note:** 
- In C, `void*` implicitly converts to any pointer type
- In C++, the conversion requires a cast

**Initialization:** "Placement new"
- `new (address) type`
- Constructs a "type" object at "address"
- Does not allocate memory (memory should already be allocated at "address")

```C++
template<typename T> class vector {
    // ...
    public:
        vector(): n{0}, cap{1}, theVector{static_cast<T*>(operator new(sizeof(T)))} {}
        vector(size_t n, T x = T{}): 
            n{n}, cap{n}, theVector{static_cast<T*>(operator new(n *sizeof(T)))} {
            
            for (size_t i = 0; i < n; ++ i)
                new(theVector + i) T(x);
        }
        // ...
        void push_back(T x) {
            increase_cap();
            new(theVector + (n++)) T(x);
        }

        void pop_back() {
            if (n) {
                theVector[--n].~T() // Must explicitly invoke destructor
            }
        }
        void destroy_items() {
            for (auto &x: *this)
                x.~T();
        }
        // another version of destroy_items:
        void clear() {
            // doing --n here is kinda sus but I guess it's fine because we don't use n anymore
            while (n) {
                pop_back;
            }
        }
        ~vector() {
            // we can use clear() here
            destroy_items();
            operator delete(theVector);
        }
};
```

---
[Where do I even start << ](./problem_12.md) | [**Home**](../README.md) | [>> Less Copying](./problem_14.md) 