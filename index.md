## 一、一个NSObject对象占多少内存

iOS是<font color=red>**小端模式**</font>（数据的高字节保存在内存的高地址中，而数据的低字节保存在内存的低地址中——反过来存放数据）

1、系统分配16个字节给NSObject（通过malloc_size函数可以查看）

2、但NSObject 对象内部只使用了8个字节的空间（64位）

```cpp
    // Class's ivar size rounded up to a pointer-size boundary.
    // 类的 ivar 大小向上舍入到指针大小边界。
    uint32_t alignedInstanceSize() const {
        return word_align(unalignedInstanceSize());
    }

    inline size_t instanceSize(size_t extraBytes) const {
        if (fastpath(cache.hasFastInstanceSize(extraBytes))) {
            return cache.fastInstanceSize(extraBytes);
        }

        size_t size = alignedInstanceSize() + extraBytes;
        // CF requires all objects be at least 16 bytes.
        // CF 要求所有对象至少为 16 个字节。
        if (size < 16) size = 16;
        return size;
    }
```

1.  **<font color=red>sizeof()</font> : 它是一个运算符，在编译时就可以获取类型所占内存的大小**
  
  ```objectivec
  Person *p = [Person new];
  sizeof(p)  // 在编译过程中会变成一个常数， p 是一个指针，实际上就是一指针\
  的size 是 8个字节
  ```
  
2. <font color=red>class_getInstanceSize</font>:依赖于`<objc/runtime.h>`，返回创建一个实例对象所需内存大小
  
3. <font color=red>malloc_size</font>:依赖于`<malloc/malloc.h>`，返回系统实际分配的内存大小
  
  ```swift
  //  右移3位，然后左移3位 8的倍数。 
  malloc_size = class_getInstanceSize()+7 >> 3 << 3
  ```
  
  **方法不会放在对象内存中，因为对象可以多份，方法一份就可以了**
