# The Rule of Three
The rule of three is a rule in C++ which claims that if a class defines any of the following, then it should explicitly define all three:

1.  destructor
2.  copy constructor
3.  copy assignment operator

So far, you have learned how to define all three, but now let's see what kind of problems you might face if you choose to ignore the above-mentioned rule. Let's start with a simple class:
```cpp
#include <iostream>
using namespace std;

class CRectangle {
    private:
        int *w, *h;
    public:
        //constructor with two parameters --- meaning that the default one will not be created
        CRectangle(int a, int b){
            w = new int;
            h = new int;
            *w = a;
            *h = b;
        }

        //destructor
        ~CRectangle(){
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

        friend void print_w_h(CRectangle r);
};


void print_w_h(CRectangle r){
    cout << *(r.w) << endl;
    cout << *(r.h) << endl;
}


int main (){
    CRectangle rect(2,4);
    cout << "Value of w is: " << rect.getW() << " and the value of h is: " << rect.getH() << endl;
    cout << "First time calling print" << endl;
    print_w_h(rect);
    cout << endl;
    cout << "Value of w after calling the print_w_h function is: " << rect.getW();
    cout << " and the value of h is: " << rect.getH() << endl;
    return 0;
}
```

Copy and run the code above. After the code is executed you notice something strange. After calling the function _print_w_h_ the member of the _rect_ instance change their values. However, this is not what we expected. Let's have a closer look at what is happening.

In the main function we first create an instance of our class by calling the defined constructor. Then, we print the values of _w_ and _h_, and we see that the values are 2 and 4 respectively (as set by the constructor). Remember that _w_ and _h_ are **pointers** that point at int values, meaning that the constructor has to **dynamically allocate the memory** to store the assigned values. Let's see how this has been implemented in the class:
```cpp
//constructor with two parameters --- meaning that the default one will not be created
CRectangle(int a, int b){
      w = new int;
      h = new int;
      *w = a;
      *h = b;
}
```

In the main function, after printing the values of w and h, we call the function print_w_h. Let's analyze how that call is being implemented:
```cpp
void print_w_h(CRectangle r){
    cout << *(r.w) << endl;
    cout << *(r.h) << endl;
}
```

It is worth keeping in mind that this function is declared as a friend member function of class CRectangle and that is why expressions like *(r.w) or *(r.h) are allowed. Ok, back to what happens when this function is called from the main function. The function call looks like this print_w_h(rect). The instance rect is being copied into the parameter r of the function. Considering that the class CRectangle does not have a copy constructor defined, the compiler will create one automatically. The automatically created copy constructor does the following:
```cpp
CRectangle(const CRectangle &r){
    w = r.w;
    h = r.h;
}
```

As shown above, the copy constructor simply copies each member of the original class (in our case that is _rect_ from the main function) into the members of the new class (in our case the _r_ input parameter of function _print_w_h_). However, since the members in our class are actually pointers, this means that we actually don't copy the values from _rect_ to _r_, we just copy the addresses in the memory in which these values are stored. This means that now, we have two instances with members pointing at the same location in memory. This type of copy is called a shallow copy.

Shallow copies, if we are not aware of them, can create a lot of problems. Those problems are usually associated to the implementation of the destructor of the class. Let's continue with the analysis of our code. Now, that we have called the _print_w_h_ function, this function will simply print the values written into the memory at the locations stored in _w_ and _h._ Then, the function is finished and the instance _r_ is out of scope. This means that the destructor will be called and the memory allocated by _w_ and _h,_ will be released. However, since _w_ and _h_ of the instance _r,_ point at the same memory locations as _w_ and _h_ of the original copy from the main function —- _rect —-_ the memory allocated by our original instance gets released too. Now, we have two dangling pointers _w_ and *h (*members of _rect_). That is why, when we print the values of _w_ and _h,_ after the function call, we get random values (note: we should not even be accessing this locations in the memory, because we do not own that memory any more - it has been released).

A simple workaround, to make this code work, without implementing the copy constructor is to change the input parameter of our function to be a reference:
```cpp
void print_w_h(CRectangle &r){
  cout << *(r.w) << endl;
  cout << *(r.h) << endl;
}
```

Remember that you have to change the function declaration inside of the class:
```cpp
friend void print_w_h(CRectangle &r);
```

Now, if you run the code, you will notice that we don't have a problem with unauthorised memory access. Since the input parameter of our function is a reference to an instance of our class, at the end of the function call, the reference is being destroyed but not the instance, meaning that the destructor is not being called.

Sometimes, we actually want to pass an instance by value to a function. To do so, we have to define a copy constructor that will allow us to create a deep copy. The implementation is shown below:
```cpp
CRectangle(const CRectangle &r){
    w = new int;
    h = new int;
    *w = *(r.w);
    *h = *(r.h);
}
```

Now, a real (deep) copy is created. The members _w_ and _h_ of the new copy dynamically allocate the memory to hold their own values, and then the values stored in the memory are being copied from one location to the new location. This means that now, two instances exist, and both instances own their own memory locations and store two separate copies (there is no overlap in the memory). If we revert the changes made to the function _print_w_h_ (i.e. if we remove the &, and pass by value instead of passing by reference) everything is still going to work (note: when testing this remember to remove the operators & from the function definition and the function declaration).

Now, let's change our main function. The new main function should look like this:
```cpp
int main (){
    CRectangle rectA(2,4);
    CRectangle rectB(1,1);
    cout << "This is rectA(" << rectA.getW() << ", " << rectA.getH() << ")" << endl;
    cout << "This is rectB(" << rectB.getW() << ", " << rectB.getH() << ")" << endl;
    cout << "Now we will assign rectB = rectA" << endl;
    rectB = rectA;
    cout << "setting the values of rectB to (10,10)" << endl;
    rectB.set_values(10,10);
    cout << endl;
    cout << "Checking the values of rectA and rectB after the asignment" << endl;
    cout << "This is rectA(" << rectA.getW() << ", " << rectA.getH() << ")" << endl;
    cout << "This is rectB(" << rectB.getW() << ", " << rectB.getH() << ")" << endl;
    return 0;
}
```

After running the code again, even with our copy constructor, we still have a problem. The problem arises due to a shallow copy again. The shallow copy is created in the line rectB = rectA. This line of code assigns the member of rectA to the members of rectB. The problem is the same as the problem we had with the copy constructor. Since we have not overloaded the assignment operator, the default one is being used. Considering that our members are pointers, instead of copying the values, the addresses are being copied, which results in shallow copies. This is what the default assignment operator does:
```cpp
CRectangle &operator=(const CRectangle &other){
    (this->w) = (other.w);
    (this->h) = (other.h);
    return *this;
}
```

In order to ensure that new problems won't arise due to shallow copies, we have to overload the assignment operator as well.
```cpp
CRectangle &operator=(const CRectangle &other){
    *(this->w) = *(other.w);
    *(this->h) = *(other.h);
    return *this;
}
```

Unlike the default assignment operator, the one defined above copies the values rather than the addresses, avoiding the creation of shallow copies.

In conclusion, whenever we use dynamic memory allocation and we define a destructor, we should define the copy constructor and overload the assignment operator. One more thing to keep in mind: Even if the copy constructor and the assignment operator are defined properly it is smart to pass classes and structures by reference, to avoid the creation of unnecessary copies of large objects. Shallow copies are not always bad (they allow us to avoid copying large objects), but we have to be aware of them, and avoid problems that might arise (as discussed above).
