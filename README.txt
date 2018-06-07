 * Dynamic-Memory-Allocator
 *
 * HingOn Miu (hmiu)
 * 
 * Actually I developed 7 different combinations of malloc, eg. implicit list
 * plus best fit, explicit plus best fit, etc. In the end, I find that segragated
 * free list plus a combination of best fit and first fit of free blocks.
 * 
 * I implemented singly linked list embedded inside the free blocks, such that
 * all free blocks with same size class are linked together. The linked list
 * top address (a total of 47 size classes: 16, 24, 32... 3976-4096, >=4097).
 * Hence, I visualize the linked list as a form of stack, so the way it pops a
 * the next free block pointer address is first fit since it simply go inside
 * that size class and find the first free block that has size bigger or equal
 * to the asked size. While I think it is also best fit because I have some
 * part of the small size classes (eg. 16, 24, 32,... 128) that only contains
 * one size, and also, it must find a size that perfectly matches the asked
 * size if it decides to pop one pointer address from those lists.
 *
 * Here is a way to visualize my implementation:
 *Allocated block of any size:
 * ____________________________
 *|   size               |allo | <--- header
 *|______________________|_____|
 *|                            |
 *|                            | 
 *|                            |
 *|                            |
 *|                            |
 *|                            |
 *|                            |
 *|                            |
 *|                            |
 *|____________________________|
 *|   size               |allo | <--- footer
 *|______________________|_____|
 *<----------32 bits----------->
 *
 *
 *Free block of any size: (singly linked list)
 * ____________________________
 *|   size               |free | <--- header
 *|______________________|_____|
 *|                            |
 *|        next_free_bp        | <--- 64 bits pointer to next_free_bp
 *|                            |
 *|____________________________|
 *|                            |
 *|                            |
 *|                            |
 *|                            |
 *|                            |
 *|___________________________|
 *|   size               |free | <--- footer
 *|______________________|_____|
 *<----------32 bits----------->
 *
 *
 *One example of size class 16 that only has one element:
 * ___________________
 *|         |         | <--- 64 bits global scalar variables to store
 *|         |         |   the address of the first free block pointer address
 *|_________|_________|     (eg. 0x10)
 *<------2 words------>
 *         |
 *         |                                        all 8 btye alignment
 *         |__________                                         |
 *                    |                                        v
 * ___________________V_________________________________________________
 *|         |         |         |         |         |         |         |
 *|footer  1|header  0|        0x48       |footer  0|header  1|         | 
 *|_________|_________|_________|_________|_________|_________|_________|
 *0x0        0x4       0x8    |  0xc       0x10      0x14      0x18 ....
 *....                        |  
 *...                         | 
 *..                   _______| 
 *.                   |
 * ___________________V_________________________________________________
 *|         |         |         |         |         |         |         |
 *|footer  1|header  0|       NULL        |footer  0|header  1|         | 
 *|_________|_________|_________|_________|_________|_________|_________|
 *0x40       0x44      0x48   ^   0x4c      0x50      0x54      0x58 ....
 *                            |
 *               since the address field stores NULL
 *                it means there is no next free
 *                block address stored in the list
 *
 * Hence, when free is called, it simply "push" the free bp into the right
 * size class, or do any coalescing if needed, and insert the coalesced free bp
 * in the right size class. Later on, when malloc is called, it finds the right
 * size class, and check one by one in the size class until it finds a size that
 * matches the asked size, and pops it off the size class linked list, and
 * return that pointer address.
 *
 * But if the asked size cannot match any free bp in all the size classes, then
 * malloc will call extendheap with the asked size and coalesce with the last
 * block if it is free, and return that pointer address.
 * 
