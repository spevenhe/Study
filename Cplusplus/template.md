## temlpate<>
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
