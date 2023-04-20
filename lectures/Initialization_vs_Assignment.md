# Initialization vs Assignment
Considering the importance of resource management when using more complex types like structures or classes we will have a closer look at how to dynamically allocate memory and how to initialize objects and copy them. Throughout this lecture we will be using a very simple class to explain the basic concepts:
```cpp
#include <iostream>
using namespace std;

class CRectangle {
    private:
        int *w, *h;
        string name;
    public:
        //constructor with three parameters --- meaning that the default one will not be created
        CRectangle(int a, int b, string name){
            cout << "Constructor CRectangle(int a, inb b, string "<< name << ") is being called" << endl;
            w = new int;
            h = new int;
            *w = a;
            *h = b;
            this->name = name;
        }

        //copy constructor --- this copy constructor creates a deep copy
        //(allocates new memory and copies the values into these memeory locations)
        CRectangle(const CRectangle &r){
            cout << "the copy constructor CRectangle(const CRectangle &r) is being called" << endl;
            w = new int;
            h = new int;
            *w = *(r.w);
            *h = *(r.h);
            name = r.name;
        }

        CRectangle &operator=(const CRectangle &other){
            cout << "Assignment operator CRectangle &operator=(const CRectangle &other) is being called by " << name << endl;
            *(this->w) = *(other.w);
            *(this->h) = *(other.h);
            return *this;
        }

        //destructor
        ~CRectangle(){
            cout << "Destructor ~CRectangle() is being called" << endl;
            delete w;
            delete h;
        }

        void set_values (int a, int b) {
            *w = a;
            *h = b;
        }

        int getW(){
            return *w;
        }

        int getH(){
            return *h;
        }

        friend void pass_by_value(CRectangle r);
        friend void pass_by_reference(CRectangle &r);
};


void pass_by_value(CRectangle r){
    cout << "pass by value" << endl;
}

void pass_by_reference(CRectangle &r){
    cout << "pass by reference" << endl;
}
```

Considering the simplicity of the above class, at this stage you should be able to understand the code presented above. One thing to highlight is that since we have defined a constructor with three input parameters, the default constructor will not be created. Also notice, that according to the rule of three, we have define a destructor, a copy constructor and we also overloaded the assignment operator.

**Initialization** is the process of allocating the memory and assigning the values to an instance of a class. Initialization is performed by calling the appropriate constructor. **Assignment** is the process of copying the value of one variable/object into another variable/object. Assignment is performed by calling the appropriate assignment operator. Due to the ambiguous syntax (i.e. the "=" operator can be used for initialization as well), beginners tend to confuse the concept of initialization and assignment. Let's write a simple main function to get a better understanding of the differences.
```cpp
int main (){
    CRectangle rectA(2,4, "rectA"), rectB(4,5,"rectB");
    CRectangle rectC = CRectangle(3,3,"rectC");
    CRectangle rectD(rectA);
    rectA = rectB;
    rectD = CRectangle(6,6,"temp");
    return 0;
}
```

Let's analyze the main function line by line. The first line initializes rectA and rectB by calling the constructor with three input parameters. The second line initializes rectC by calling the constructor with three input parameters as well. Notice that we are using the "=" operator, but since this is an initialization of the rectC instance, the constructor is being called instead of the overloaded assignment operator. The third line initializes rectD by using the copy constructor. The forth line assigns rectB to rectA by calling the overloaded assignment operator. This line can be interpreted and can be rewritten to rectA.opeartor=(rectB). It is important to notice the difference between this line and the line used to initialize rectC, which was an example of object initialization (the main difference is that rectA, already existed in the memory, and then an assignment happened, whereas in line two we allocate the memory and assigne the values in the same line). Finally, the fifth line is also an example of assignment in which the value of the temporarily created object CRectangle(6,6,"temp") is being assigned to rectD. At the end the destructor is called for all the created objects. After executing the code above, we get the following output:
```
Constructor CRectangle(int a, inb b, string rectA) is being called
Constructor CRectangle(int a, inb b, string rectB) is being called
Constructor CRectangle(int a, inb b, string rectC) is being called
the copy constructor CRectangle(const CRectangle &r) is being called
Assignment operator CRectangle &operator=(const CRectangle &other) is being called by rectA
Constructor CRectangle(int a, inb b, string temp) is being called
Assignment operator CRectangle &operator=(const CRectangle &other) is being called by rectA
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
```

Now, let's analyze the difference in the interpretation when passing by value vs passing by reference. For our experiment we will write a new main function:
```cpp
int main (){
    CRectangle rectA(2,4, "rectA");
    cout << endl << "calling the function pass_by_value" << endl;
    pass_by_value(rectA);
    cout << endl << "calling the function pass_by_reference" << endl;
    pass_by_reference(rectA);
    return 0;
}
```

As expected, the first line will use the constructor with three parameters to initialize rectA. Then we print a sentence saying that we are calling the function "pass_by_value", and then we call that function. It is interesting to notice, that when this function is being called, since we pass the value of rectA to the function, the input parameter has to be initialized, and it is being initialized with the copy constructor (same as rectD in the previous example). Then, the function is executed and the copy gets destroyed (the destructor is being called on the copy (remember this creates problems when using shallow copies - which is not the case here). The next line prints a sentence saying that we are calling the function "pass_by_reference", and then we call that function. Here, we see that the function gets executed without creating any new instances or copies of our instance rectA, because the function parameter is just used as an alias (i.e. reference) of our original instance (that is why the destructor is not being called at the end of the function execution). After executing the above code we get the following output:
```
Constructor CRectangle(int a, inb b, string rectA) is being called

calling the function pass_by_value
the copy constructor CRectangle(const CRectangle &r) is being called
pass by value
Destructor ~CRectangle() is being called

calling the function pass_by_reference
pass by reference
Destructor ~CRectangle() is being called
```

### Pointers and Dynamic memory allocation
Now that we understand the difference between initialization and assignment, let's have a closer look at dynamic memory allocation. So far, we have seen how to dynamically allocate the memory for primary and derived datatypes (e.g. int, char, float, arrays). The syntax is the same when allocating the memory for instances of classes. However, there are a few things to keep in mind (it is good to keep in mind that whenever we are working with classes or structures, since they require more memory resources, we should dynamically allocate the memory for the instances). We will use the same class from the beginning of the lecture. We will write a different main function.
```cpp
int main (){
    cout << "We first initialize a null pointer that can point at an instance of CRectangle" << endl;
    CRectangle *pA = nullptr;
    cout << "Now we dynamicaly allocate the memeory for the instance" << endl;
    pA = new (nothrow) CRectangle(2,2,"rectA");
    return 0;
}
```

The first line of the above code prints a sentence to the screen, and then in the next line we create a null pointer that can point at an instance of our class. Then, we dynamically allocate the memory for our instance. We have made a mistake in the above code on purpose. Do you know what it is? Let's first have a look at the output of the code above:
```cpp
We first initialize a null pointer that can point at an instance of CRectangle
Now we dynamicaly allocate the memeory for the instance
Constructor CRectangle(int a, inb b, string rectA) is being called
Destructor ~CRectangle() is being called
```

The last line shows that the destructor is being called. Even though this is very obvious from this short function, it can be harder to spot when our function is more complicated and when it has multiple return statements. Let's have a look at a simple example. We will write a simple function, that has one input parameter (reference of type CRectangle) and it dynamically allocates memory for a local instance of CRectangle and simply prints the values of w and h if w is greater than 5 and returns True or prints "not greater than 5" if it is not and returns False.
```cpp
bool print_w_h(CRectangle &r){
    CRectangle *p = new (nothrow) CRectangle(r);
    if (p->getW() > 5){
        cout << "w: " << p->getW() << ", h: " << p->getH() << endl;
        delete p;
        return true;
    }
    else{
        cout << "not greater than 5" << endl;
        return false;
    }
}
```

Now let's write the main function to test the function above:
```cpp
int main (){
    CRectangle rectA(20,20,"rectA"), rectB(1,1,"rectB");
    cout << "calling print_w_h(rectA)" << endl;
    print_w_h(rectA);
    cout << "calling print_w_h(rectB)" << endl;
    print_w_h(rectB);
    cout << "finished the function calls" << endl;
    return 0;
}
```

After executing the code above, we get the following output:
```
Constructor CRectangle(int a, inb b, string rectA) is being called
Constructor CRectangle(int a, inb b, string rectB) is being called
calling print_w_h(rectA)
the copy constructor CRectangle(const CRectangle &r) is being called
w: 20, h: 20
Destructor ~CRectangle() is being called
calling print_w_h(rectB)
the copy constructor CRectangle(const CRectangle &r) is being called
not greater than 5
finished the function calls
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
```

A closer look at the output reveals a big mistake: the first function call releases the dynamically allocated memory, but the second one does not release the dynamically allocated memory inside of the function. The reason is that we have forgotten to add a delete p; before calling return false. This was a simple example that shows how memory leakage can happen if we are not careful with our actions.

#### Dynamically allocating the memory for an array
The syntax for allocating the memory for an array of objects is the same as the one we are used to when dynamically allocating the memory for primary and derived types. Let's start from a simple main function:
```cpp
int main (){
    CRectangle *pA = nullptr;
    pA = new (nothrow) CRectangle[5];
    delete []pA;
    return 0;
}
```

As shown, we first initialize the pA pointer to null. Then we dynamically allocate the memory for an array of size 5. And then we simply release the allocated memory. However, when we run the above code, we get an error:
```
example.cpp:94:24: error: no matching constructor for initialization of 'CRectangle [5]'
    pA = new (nothrow) CRectangle[5];
```

The problem is that when initializing an array as shown above, we are calling the default constructor to initialize all array elements. However, our class does not have a default constructor. An easy fix is to add a default constructor as shown below:
```cpp
//default constructor
CRectangle(){
    cout << "Default CRectangle() constructor was called" << endl;
    w = new int;
    h = new int;
    *w = 0;
    *h = 0;
    name = "";
}
```

If we run our code again, we get the following output:
```
Default CRectangle() constructor was called
Default CRectangle() constructor was called
Default CRectangle() constructor was called
Default CRectangle() constructor was called
Default CRectangle() constructor was called
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
```

As expected, the default constructor was called 5 times (for each array element) and then the destructor is called for all our instances when delete []pA is called. A default constructor does not always make sense. For example, if we had a class that represents temperature sensor measurements, or a class that stores dates it would be much harder to decide what the default values should be initialized to. However, that is not the only reason why the syntax above is not the best way to initialize an array of objects. The problem with the syntax above is that we are trying to find a block of consecutive memory addresses that would be large enough to store the whole array. Considering that our array could be very large, we can imagine that memory availability can become an issue. Therefore, when working with arrays of instances of classes and structures it is always better to have an array of pointers that are pointing at instances of those classes and then to dynamically allocate the memory for each element. This allows us to find space in the memory to store an array of pointers (small amount of memory), and then the dynamically allocated elements do not have to be stored next to each other in the memory. Another benefit could arise if we try to sort this array. Instead of moving the objects around the memory, we can simply make the pointers point at different memory locations. This approach also allows us to avoid having a dummy default constructor and to keep only the constructors that we find useful.

**For the rest of this lecture remove the default constructor that was added above.**

To achieve the same output as above, without using the default constructor we will use an array of pointers at instances of our class. A simple main function is shown below.
```cpp
int main (){
    CRectangle *pA[5] = {};
    for(int i = 0; i < 5; i++){
        pA[i] = new (nothrow) CRectangle(0,0,"");
    }
    for(int i = 0; i< 5; i++){
        delete pA[i];
    }
    return 0;
}
```

The first line initializes an array of 5 null pointers that point at instances of our class. The first for loop dynamically allocates the memory for each element in the array, and sets the values to (0,0,""). The second for loop releases the memory that was dynamically allocated by each element of the array. The output of the code above is is the same as the previous one, except that now we do not use the default constructor (remember, we deleted the default constructor and in the first for loop we use the constructor with three parameters).
```
Constructor CRectangle(int a, inb b, string ) is being called
Constructor CRectangle(int a, inb b, string ) is being called
Constructor CRectangle(int a, inb b, string ) is being called
Constructor CRectangle(int a, inb b, string ) is being called
Constructor CRectangle(int a, inb b, string ) is being called
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
```

There is one more thing that we can do to improve our code even more. Instead of initializing an array of 5 pointers, we could dynamically allocate this array, which allows us to even ask the user how many elements the array should have. Remember that in the previous example we used the number 5 and the other alternative was to use a const int to initialize the array size. However, if we want to be able to use any variable or to define the size based on a user input, we have to dynamically allocate this array too. The implementation is shown below:
```cpp
int main (){
    cout << "Input the array size: ";
    int arr_size = 0;
    cin >> arr_size;
    CRectangle **pA = new (nothrow) CRectangle*[arr_size];
    for(int i = 0; i < 5; i++){
        pA[i] = new (nothrow) CRectangle(0,0,"");
    }
    for(int i = 0; i< 5; i++){
        delete pA[i];
    }
		delete []pA;
    return 0;
}
```

The first line in the main function asks the user to input the array size, then we input the size and store it in the integer variable arr_size. Then, we dynamically allocate an array of pointers at elements that are instances of our class. This is implemented by creating a pointer at an array of pointers (`CRectangle **pA = new (nothrow) CRectangle*[arr_size];`) - remember that `CRectangle *pA[arr_size];` is not correct. The rest of the code is identical to the previous example. The output is going to be the same as the previous one:
```
Input the array size: 5
Constructor CRectangle(int a, inb b, string ) is being called
Constructor CRectangle(int a, inb b, string ) is being called
Constructor CRectangle(int a, inb b, string ) is being called
Constructor CRectangle(int a, inb b, string ) is being called
Constructor CRectangle(int a, inb b, string ) is being called
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
Destructor ~CRectangle() is being called
```
