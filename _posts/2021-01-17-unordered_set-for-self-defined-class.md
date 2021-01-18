---
category: program
---

In this article, I will show you how to use `unordered_set` for a self-defined class.

Let's start with a short example below. In this example, we have a self-defined class `Student`. We also declared a roster, which is an `unordered_set` of students.

```cpp
#include <string>
#include <iostream>
#include <unordered_set>
using namespace std;

class Student {
public:
    Student(int id, string name, bool enrolled): \
        _id(id), _name(name), _enrolled(enrolled) {}

private:
    int _id;
    string _name;
    bool _enrolled;
};


int main() {
    unordered_set<Student> roster;
    roster.insert(Student(1, "miaxu", true));
    return 0;
}
```

Compiling this program, you will see a long list of errors. Depending on which compiler you are using, you may see <small>error: call to implicitly-deleted default constructor of 'unordered_set\<Student\>'</small> or <small>error: implicit instantiation of undefined template 'std::__1::hash\<Student\>'</small>. You will see other errors as well, but we will talk about that later.

To figure out the reason of compiler errors, let's see the declaration of `unordered_set`:

```cpp
template<
    class Key,
    class Hash = std::hash<Key>,
    class KeyEqual = std::equal_to<Key>,
    class Allocator = std::allocator<Key>
> class unordered_set;
```

As you can see above, `unordered_set` is a template class. In the parameter list of the template, the first one is a `Key`, which should be our self-defined class `Student`. The second template parameter, however, is a class of `Hash`, which should define the behavior of a hash function on the class `Student`. Obviously we didn't define any hash function on `Student`, so the compiler doesn't know how to deal with it. To make the compiler happy, we need to define a hash function against
our self-defined class.

Now that we know we need a hash function for our class, let's see what it is supposed to look like. You can find the information <a href="https://en.cppreference.com/w/cpp/utility/hash">here</a>. As what it states, the `std::hash` is a struct type with a member function `operator()`, which calculates the hash of the argument. On that page, we also learned that the standard library has already provided spcializations for basic types such as `int` and `char`.

A well-defined hash function is important because it can greatly reduce conflict and improve the access efficiency. Given that the identity of a `Student` object is an integer, we can use `std::hash<int>` directly inside our hash implementation:

```cpp
size_t operator() (const Student &s) const {
    return hash<int>()(s._id);
}
```

Another compiler error that we see is <small>error: invalid operands to binary expression ('const Student' and 'const Student')</small>. This is because when we insert an object into an unordered set, the program must know whether this object equals to any existing object in the container. To clear up this error, we must define `bool operator==()` for our self-defined class.

The implementation after the fix will look like this:

{% highlight cpp linenos %}
#include <string>
#include <iostream>
#include <unordered_set>
using namespace std;

class Student {
public:
    Student(int id, string name, bool enrolled): \
        _id(id), _name(name), _enrolled(enrolled) {}


private:
    int _id;
    string _name;
    bool _enrolled;


friend bool operator== (const Student &here, const Student &there);
friend struct Student_hash;
};

struct Student_hash {
    size_t operator() (const Student &s) const {
        return hash<int>()(s._id);
    }
};

bool operator== (const Student &here, const Student &there) {
    return here._id == there._id;
}

int main() {
    unordered_set<Student, Student_hash> roster;
    roster.insert(Student(1, "miaxu", true));
    return 0;
}
{% endhighlight %}

Before closing this topic, I have one comment to address. In some cases (especially during an interview) you may solve a problem by using `unordered_map<int, Student>` instead of `unordered_set<Student>`, where the first template parameter refers to the identity. If you work in a project and your task, on the other hand, is to provide others with a well-defined class, then implementing a hash function and `operator==` might be a better choice because you never know how your class is used.

<strong>MESSAGES TO TAKE AWAY</strong>

1. To use `unordered_set` for a self-defined class, you need to implement a hash function and `operator==()`.
2. Sometimes you may solve a problem by using `unordered_map` instead of `unordered_set`. That way you can avoid above APIs.
