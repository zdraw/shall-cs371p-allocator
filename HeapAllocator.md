# Introduction #

This program creates an Allocator object that decouples the allocate and construct from the **new** command and decouples the deallocate and destructor from the **delete** command. The Allocator is instantiated with a type T and size N in bytes. The users of the class can then allocate memory in sizes of T, construct T objects in those spaces, destroy them at the end of their code, and then deallocate the space. In this way the Allocator acts like a heap manager.


# Details #

When instantiating an Allocator object, the size N will create an char array of that size. This array is used as the heap.  There are 4 main functions that can be called:
  * pointer allocate (size\_type n)
  * void construct (pointer p, const\_reference v)
  * void deallocate (pointer p, size\_type = 0)
  * void destroy (pointer p)
Pointer is a T**typedef
size\_type is the number of Ts that Allocator will attempt to allocate space for.
const\_reference is previously instantiated T object that will be used by T's copy constructor**

The internal space monitors will be two sentinels the size of an int surrounding each allocated block of memory. The sentinels will be the same value and positive if free and negative if used. The value reflects an integer multiple of T (meaning it does not take into account the space used by the sentinels.

The default constructor for Allocator will just sets up the sentinels at the beginning and end with the value N - 2\*sizeof(int).

The allocate function is O(n) and follows a first-fit algorithm. If the first block that it encounters has enough space for the specified amount but does not have enough space for one more T and two sentinels then the allocate will set up the sentinels to give all of the space and return a pointer to the beginning of that block. This occurs when the available space is less than n\*sizeof(T) + sizeof(T) + 2\*sizeof(int) but greater than n\*sizeof(T). If it encounters a block with more than enough space then it will set the sentinels to have the value of -n\*size(T).

The deallocate function looks for free blocks before and after the block specified by the pointer. It will then change the sentinels at the beginning of the first free block and the end of the second free block to be sentinel1 + sentinel2 + 2\*sizeof(int) to reclaim and combine all the free space. As long as this method is the only code used to deallocate blocks, there should never be two free blocks side-by-side that have not been recombined.

The construct function creates a new T object at the specified pointer.

The destroy function destroys a T object at the specified pointer.

There is also an O(n) valid() function that checks the array for consistency. It checks that
  * Each sentinel has a corresponding sentinel with the same sign and value
  * There are no uncombined freeblocks
  * The sentinel values plus the size taken up by the sentinels adds up to N

Each Allocator function calls assert(valid()) before returning to make sure that the heap is in a good state.