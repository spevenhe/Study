## temlpate<>

It's a specialization. template<> means that the specialization itself is not templated- i.e., it is an explicit specialization, not a partial specialization.

```
template<class T1>
struct bar
{
  void doStuff() { std::cout << "generic bar"; }
};

template<>
struct bar<int>
{
 void doStuff() { std::cout << "specific bar with T1=int"; }
};
```
