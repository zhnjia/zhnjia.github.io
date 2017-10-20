---

layout: page
title: How Would You Get the Count of an Array in C++?

---

Chromium源码里计算数组个数的算法：

    template <typename T, size_t N>
    char (&ArraySizeHelper(T (&array)[N]))[N];
    #define arraysize(array) (sizeof(ArraySizeHelper(array)))

关于这个宏定义，有这样一段解释：

> The arraysize(arr) macro returns the # of elements in an array arr.  The
> expression is a compile-time constant, and therefore can be used in defining
> new arrays, for example.  If you use arraysize on a pointer by mistake, you
> will get a compile-time error.  For the technical details, refer to
> <http://blogs.msdn.com/b/the1/archive/2004/05/07/128242.aspx>.

下面摘取了大部份原文，以供阅读：

The question is simple: given a C++ array (e.g. x as in int x[10]), how would you get the number of elements in it?
An obvious solution is the following macro (definition 1):

    #define countof( array ) ( sizeof( array )/sizeof( array[0] ) )

I cannot say this isn’t correct, because it does give the right answer when you give it an array.  However, the same expression gives you something bogus when you supply something that is not an array.  For example, if you have

    int * p;

then countof( p ) always give you 1 on a machine where an int pointer and an int have the same size (e.g. on a Win32 platform).
This macro also wrongfully accepts any object of a class that has a member function operator[].  For example, suppose you write

    class IntArray
    {
    private:
        int * p;
        size_t size;

    public:
        int & operator [] ( size_t i );

    } x;

then sizeof( x ) will be the size of the x object, not the size of the buffer pointed to by x.p.  Therefore you won’t get a correct answer by countof( x ).
So we conclude that definition 1 is not good because the compiler does not prevent you from misusing it.  It fails to enforce that only an array can be passed in.
What is a better option?
Well, if we want the compiler to ensure that the parameter to countof is always an array, we have to find a context where only an array is allowed.  The same context should reject any non-array expression.
Some beginners may try this (definition 2):

    template <typename T, size_t N>
    size_t countof( T array[N] )
    {
        return N;
    }

They figure, this template function will accept an array of N elements and return N.
Unfortunately, this doesn’t compile because C++ treats an array parameter the same as a pointer parameter, i.e. the above definition is equivalent to:

    template <typename T, size_t N>
    size_t countof( T * array )
    {
        return N;
    }

It now becomes obvious that the function body has no way of knowing what N is.
However, if a function expects an array reference, then the compiler does make sure that the size of the actual parameter matches the declaration.  This means we can make definition 2 work with a minor modification (definition 3):

    template <typename T, size_t N>
    size_t countof( T (&array)[N] )
    {
        return N;
    }

This countof works very well and you cannot fool it by giving it a pointer.  However, it is a function, not a macro.  This means you cannot use it where a compile time constant is expected.  In particular, you cannot write something like:

    int x[10];
    int y[ 2*countof(x) ]; // twice as big as x

Can we do anything about it?

Someone (I don’t know who it is – I just saw it in a piece of code from an unknown author) came up with a clever idea: moving N from the body of the function to the return type (e.g. make the function return an array of N elements), then we can get the value of N without actually calling the function.

To be precise, we have to make the function return an array reference, as C++ does not allow you to return an array directly.

The implementation of this is:

    template <typename T, size_t N>
    char ( &_ArraySizeHelper( T (&array)[N] ))[N];
    #define countof( array ) (sizeof( _ArraySizeHelper( array ) ))

Admittedly, the syntax looks awful.  Indeed, some explanation is necessary.
First, the top-level stuff

    char ( &_ArraySizeHelper( ... ))[N];

says “\_ArraySizeHelper is a function that returns a reference (note the &) to a char array of N elements”.
Next, the function parameter is

    T (&array)[N]

which is a reference to a T array of N elements.
Finally, countof is defined as the size of the result of the function \_ArraySizeHelper.  Note we don’t even need to define \_ArraySizeHelper(), &#x2013; a declaration is enough.
With this new definition,

    int x[10];
    int y[ 2*countof(x) ]; // twice as big as x

becomes valid, just as we desire.
Am I happy now?  Well, I think this definition is definitely better than the others we have visited, but it is still not quite what I want.  For one thing, it doesn’t work with types defined inside a function.  That’s because the template function \_ArraySizeHelper expects a type that is accessible in the global scope.
