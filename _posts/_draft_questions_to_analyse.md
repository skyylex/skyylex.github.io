### Memory Introspection

https://www.mikeash.com/pyblog/friday-qa-2010-01-15-stack-and-heap-objects-in-objective-c.html

- Mike Ash pointed that there are other than stack and heap types of memory. What are they?

"The heap is, essentially, everything else in memory. (Yes, there are things other than the stack and heap, but let's ignore that for this discussion.)"

If to start from point that stack handles all about "local variables, as well as internal temporary values and housekeeping" and heap "malloc and free". I assume that those type could be system memory (provided by kernel) and global memory - for keeping static variables **(to prove that)**.

- Why heap is used for memory allocating?
