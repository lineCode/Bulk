Bulk
====

<center>

![image](images/logo.svg)

</center>

Bulk is a new interface for writing bulk synchronous parallel programs in C++.
It lets you implement parallel algorithms by providing common communication mechanisms
between processing elements, no matter if these corresponds to cores in a many-core accelerator,
threads, or nodes in a cluster.

Bulk does away with unnecessary boilerplate  code and the unsafe API's that are found in for example MPI, or the BSPlib standard. It provides a unified syntax for parallel programming across different platforms and modalities.
Our BSP interface supports and encourages the use of modern C++ features, enabling safer and more efficient distributed programming. We have a flexible backend architecture,
so that programs written with Bulk work for both shared memory, distributed memory, or mixed systems.

About BSP
---------

The bulk synchronous parallel (BSP) programming model, is a way of writing parallel and distributed programs. BSP is the underlying model for Bulk. Instead of communicating between processors (or nodes, or cores) asynchronously, all communication is staged and resolved at fixed _synchronization points_. These synchronizations delimit so-called _supersteps_. This way of writing parallel programs has a number of advantages over competing models:

- The resulting programs are structured, easy to understand and maintain, and their performance and correctness can be reasoned about precisely.
- Data races are eliminated almost by construction, only have to follow a small set of simple rules which can be enforced at runtime.
- Scalability is easy to obtain, since programs are written in a SPMD fashion, and this scalability can be analyzed explicitely.
- The number of communication mechanisms required are very limited, roughly only distinguishing between anonymous (message passing) or _named_ communication (through distributed variables). This makes BSP based libraries very economic (you can do much with very little).
- It has a low cost of entry. It is easy to write _correct_ BSP programs, while it is notoriously hard to write correct asynchronous parallel programs.

The bulk synchronous communication style does mean losing some flexibility, and comes with a (usually minor) performance penalty. This tradeoff is often well worth it.

Authors
-------

Bulk is developed by:

* Jan-Willem Buurlage (jwbuurlage)
* Tom Bannink (tombana)

Examples
--------

Hello world!

```cpp
bulk::thread::environment env;
env.spawn(env.available_processors(), [](auto& world) {
    auto s = world.rank();
    auto p = world.active_processors();

    world.log("Hello world from processor %d / %d\n", s, p);
});
```

Distributed variables are the easiest way to communicate.

```cpp
auto a = bulk::var<int>(world);
a(world.next_rank()) = s;
world.sync();
// ... a is now updated

auto b = a(world.next_rank()).get();
world.sync();
// ... b.value() is now available
```

Coarrays provide a convenient syntax for working with distributed arrays.

```cpp
auto xs = bulk::coarray<int>(world, 10);
xs(world.next_rank())[3] = s;
```

Message passing can be used for more flexible communication.

```cpp
auto q = bulk::queue<int, float>(world);
for (int t = 0; t < p; ++t) {
    q(t).send(s, 3.1415f);  // send (s, pi) to processor t
}
world.sync();

// messages are now available in q
for (auto [tag, content] : q) {
    world.log("%d got sent %d, %f\n", s, tag, content);
}
```

API
---

Bulk provides a modern bulk synchronous parallel API which is [documented in detail here](api/index.md).

License
-------

Bulk is released under the MIT license, for details see the file LICENSE.md.

Contributing
------------

We welcome contributions. Please submit pull requests against the develop branch.


[^1]: Single program multiple data, meaning that every processor runs the same code but on different data.
