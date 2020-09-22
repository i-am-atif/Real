
![Boost.Real](doc/other/logo/logo.png)

Boost.Real numerical data type for real numbers representation using range arithmetic.

[![Build Status](https://travis-ci.org/BoostGSoC19/Real.svg?branch=master)](https://travis-ci.org/BoostGSoC19/Real) 
[![codecov](https://codecov.io/gh/BoostGSoC19/Real/branch/master/graph/badge.svg)](https://codecov.io/gh/BoostGSoC19/Real)


## Documentation
   * [Project documentation main page](https://boostgsoc19.github.io/Real/)
   * [Doxygen documentation](https://boostgsoc19.github.io/Real/doc/html/index.html)
   * [Medium post 2018](https://medium.com/@laobelloli/boost-real-9e2dfbfbed5b)


## Introduction

### The problem addressed by boost::real
Several times, when dealing with complex mathematical calculus, numerical errors can be carried from one operation to the next and after several steps, the error may significantly increase obtaining a non-trustworthy result. Normally, in these situations, the error is generated by the representation precision limit that truncates those numbers that do not fit in the representation precision generating errors without at least keeping track of the magnitude of the error. When this is the case, the real result is equal to the obtained approximated result plus the error (which is unknown).

Another major problem when dealing with real numbers is the representation of the irrational number as the number π or e<sup>π</sup>, they are not handled by the native number data types causing limitations when calculations are based on those numbers. Normally we define a truncation of those numbers that are good enough for our purposes, but many times, the needed precision depends on the operation to do and to the composition of multiple operations, therefore, we are unable to determine which is the correct precision until we run the program.

There are a lot of libraries and tools available where we can always specify the initial precision to be used for calculations but in case of problems stated above, where we do not know the precision prior to calculations, our calculations can produce wrong or inconclusive results when precision is less than required or we will use too high precision. So, there should be a tool or library which performs all these operations and controls the precision according to requirements.

### The boost::real solution
Boost::real is a real number representation data type that address the mentioned issues using range arithmetic [1] and defining the precision as dynamical to be determined in run-time. The main goal of this data type is to represent a real number x as a program that returns a finite or infinite set of intervals containing the number x x(k) = [m<sub>k</sub> - e<sub>k</sub>, m<sub>k</sub> + e<sub>k</sub>], K ∈ N ≥ 0, e<sub>k</sub> ≥ 0. Where K1 < K2 ⇒ |x(k2)| < |x(k1)|. For this purposes, any Boost::real number has a precision const iterator that iterates over the series of intervals representing the number. The size of intervals within the series are sorted from larger to smaller, thus, each time the iterator is increased, the number precision increases due the new calculated interval is smaller than the previous one until the interval is the number itself (if possible).

Also, to allow representing irrational numbers as π or e<sup>π</sup>, boost::real has a constructor that takes as parameter a function pointer, functor (function object with the operator ()) or lambda expression that for any integer n > 0, the function returns the n-th digit of the represented number. For example, the number 1/3 can easily be represented by a program that for any input n > 0, the function returns 3.

Although it may seem that it is similar to several range/interval arithematic libraries like boost::interval arithematic but goals and features of this library are much more different. We provide a data type which can represent all computable real numbers, and the goal is to evaluate the expression whose required precision for evaluation is unknown in advance. The implementation of interval arithematic comes because every number is represented as a function which returns an interval containing that number, and interval keeps on getting more precise and close to number as we increase the precision of number. In abstract, this is not an pure interval arithematic library, the goal of this library, which is described above will keep on getting clear as you dive more into libray, is to represent numbers using intervals and adjust the precision according to requrement of calculations.

## The boost::real numbers representation
In boost::real, a number has one of the next three representations:

1. Explicit number: A number is a vector of digits sorted as in the number natural representation. To determine where the integer part ends and the fractional part starts, an integer is used as the exponent of a floating point number and determines where the integer part start and the fractional ends. Also a boolean is used to set the number as positive (True) or negative (False)
2. Algorithmic number: This representation is equal to the Explicit number but instead of using a vector of digits, a lambda function must be provided. The lambda function takes an unsigned integer "n" as parameter and returns the n-th digit of the number.
3. Operation number: A number is a composition of two numbers related by an operator (+, -, *), the number creates pointers to the operands and each time the number is used, the operation is evaluated to return the result.
4. Integer number: Specific data type for integer type numbers. The use and benefit of this specified type will be explained later.
5. Rational number: Specific data type to represent rational numbers (in a/b form). It also has some representational and performance benefits which will be explained later, after general structure and working of library is explained.

Because of the third representation type, a number resulting from a complex calculus is a binary tree where each internal vertex is an operation and the vertex children are its operands. The tree leaves are those numbers represented by either (1) or (2) or (4) or (5) while the internal vertex are those numbers represented by (3). More information about the used number representation can be found in [3]
#### Use of specialized types for integer and rational numbers:
1. They do not create an operation tree or real::operation number when we do any operation between two integer or rational numbers. So, reptitive calculations involving integer and rational number, user should user these specific types so no complex tree is generated. Any operation between two rational or integer numbers will be simply a rational or integer number which is resultant to those two. As at every precision integers are same, there is no need to make precision iterations for those calculations. So, **user should use integer and rational number for small repetitive calculations to optimize space and reduce complexity but should not use them for very large and complex calulcations because they do whole calculation in one go so it may become unnecessary complex expressions to do calculations at full precision when a small precision can give results.**
2. Some rational numbers can not be represented by a simple string, like 1/3, so we can represent them using this data type. They can be represented as a real operation number between two explicit numbers also.

## The boost::real precision iterator.
The boost::real::const_precision_iterator is a forward iterator [4] that iterates through the number interval precisions. The iterator returns two numbers, a lower and an upper boundary that represent the [m<sub>k</sub> - e<sub>k</sub>, m<sub>k</sub> + e<sub>k</sub>] limits of the number approximation interval for a given precision. Each time the iterator is incremented, the interval approximation_interval is decreased and a new interval with a better precision is obtained. Normally, there is no need to interact with the precision iterator and it is used by the boost::real operators <<, < and >.

## Interface

### Constructors and destructors
    1. boost::real(const std::string& number)
    2. boost::real(const std::string& number, std::string type)
    3. boost::real(const std::string& number, boost::real::TYPE)
    4. boost::real(const initializer_vector<int> digits)
    5. boost::real(const initializer_vector<int> digits, bool positive)
    6. boost::real(const initializer_vector<int> digits, int exponent)
    7. boost::real(const initializer_vector<int> digits, int exponent, bool positive)
    8. boost::real((unsigned int) -> int digits, int exponent)
    9. boost::real((unsigned int) -> int digits, int exponent, bool positive)
    10. boost::real(const boost::real& x)
    11. boost::~real()
  
> (1) **String constructor** 
> Creates a real instance by parsing the string. The string must have a valid number, in other case, the constructor will throw an boost::real::invalid_string_number_exception exception. Constructs a number of type real::explicit.
>
>(2) **String Constructor (with type of number passed as string)**
> Creates a real instances of type "explicit", "integer", or "rational" based by parsing the string. The string "type" must have a valid type ("explicit"/"integer"/"rational") and number represented by string should also be a valid number.
> 
> (3) **String Constructor (with type of number passed as enumerator object)**
> Creates a real instance of type "explicit", "integer", or "rational" based on type passed by enumerator boost::real::TYPE (TYPE::EXPLICIT/TYPE::INTEGER/TYPE::RATIONAL) and number given by string.
> 
> (4) **Initializer list constructor** 
> Creates a real instance that represents an integer number where all the digits numbers are form the integer part in the same order.
>
> (5) **Signed initializer list constructor** 
>Creates a real instance that represents the number where the positive parameter is used to set the number sign and the elements of the digits parameter list are the number digits in the same order.
>
> (6) **Initializer list constructor with exponent** 
> Creates a real instance that represents the number where the exponent is used to set the number integer part and the elements of the digits list are the digits the number in the same order.
>
> (7) **Initializer list constructor with exponent and sign** 
> Creates a real instance that represents the number where the exponent is used to set the number integer part and the elements of the digits list are the digits the number in the same order. This constructor uses the sign to determine if the number is positive or negative.
>
> (8) **Lambda function constructor with exponent** 
> Creates a real instance that represents the number where the exponent is used to set the number integer part and the lambda function digits is used to know the number digit, this function returns the n-th number digit.
>
> (9) **Lambda function constructor with exponent and sign** 
> Creates a real instance that represents the number where the exponent is used to set the number integer part and the lambda function digits is used to know the number digit, this function returns the n-th number digit. This constructor uses the sign to determine if the number is positive or negative.
>
> (10) **Copy constructor** 
> Creates a copy of the number x, if the number is an operation, then, the constructor creates new copies of the x operands.
>
> (11) **Default destructor** 
> If the number is an operator, the destructor destroys its operands.

### Operators

    1. boost::real operator+=(const boost::real& x)
    2. boost::real operator-=(const boost::real& x)
    3. boost::real operator*=(const boost::real& x)
    4. boost::real operator+(const boost::real& x) const
    5. boost::real operator-(const boost::real& x) const
    6. boost::real operator*(const boost::real& x) const
    7. boost::real& operator=(const boost::real& x)
    8. boost::real& operator=(const std::string& x)
    9. boost::real& operator==(const boost::real& x)
    10. bool operator<(const real& other) const
    11. std::ostream& operator<<(std::ostream& os, const boost::real& x)
    12. int operator[](unsigned int n) const

> (1) Modifies the number to use the third representation. and sets copies of *this and x respectively as the left and right operands and sets addition as the operation.
>
> (2) Modifies the number to use the third representation. and sets copies of *this and x respectively as the left and right operands and sets subtraction as the operation.
>
> (3) Modifies the number to use the third representation. and sets copies of *this and x respectively as the left and right operands and sets multiplication as the operation.
>
> (4) Creates a new boost::real number using the third representation. For this purpose, the operator creates copies of *this and x to use as the new real number operands and defines the addition as the operation.
>
> (5) Creates a new boost::real number using the third representation. For this purpose, the operator creates copies of *this and x to use as the new real number operands and defines the subtraction as the operation.
>
> (6) Creates a new boost::real number using the third representation. For this purpose, the operator creates copies of *this and x to use as the new real number operands and defines the multiplication as the operation.
>
> (7) Uses the copy constructor to create a copy of x stored in *this
>
> (8) Uses the string constructor to create a real that represents the number specified in the x string.
>
> (9) compares *this with x to check if they represent the same number. **WARNING:** Because different numbers can have an arbitrary number of equal intervals, numbers equality can be checked only for those numbers that can be fully represented with the maximum_precision or their approximation intervals stop overlapping before the maximum_precision is reached. In other case, it will throw a boost::real::precision_exception.
>
> (10) Compares *this with x to check if *this is lower than x. This operator creates two precision iterators (one for each number) and iterates until the number intervals stop overlapping when that happens, it compares the intervals boundaries to determine if *this is less than x. **WARNING:** If *this is equal to x, then the intervals will always overlap, because of this, and if the numbers intervals still overlap once the maximum_precision is reached, the operator throws a boost::real::precision_exception.
>
> (11) Creates a const_precision_iterator to print the number using the iterator << operator.
>
> (12) Returns the n-th digit of the represented number. **WARNING:** This operator throws invalid_representation_exception for the third representation because only explicit and algorithmic numbers can be asked for the n-th digit.

### Other methods

    1. boost::real::const_precision_iterator boost::real::cbegin()
    2. boost::real::const_precision_iterator boost::real::cend()
    3. unsigned int boost::real::maximum_precision()
    4. void boost::real::set_maximum_precision(unsigned int)

> (1) Construct a new const_precision_iterator that iterate over the *this number precisions. The constructed iterator points to the first approximation interval (the ine with less precision).

> (2) Construct a new const_precision_iterator that iterate over the *this number precisions. The constructed iterator points to the last approximation interval (the one with more precision) within the maximum precision.

> (3) Returns the instance maximum precision at which the operators will throw a boost::real::precision_exception if they cannot determine the operation value before reaching the maximum precision.

> (4) Sets a new maximum precision. If the set maximum precision is zero, the static default maximum precision will be used instead.

### Some Functions
    1. boost::real power(boost::real number, boost::real power)
    2. boost::real sqrt(boost::real number)
    3. boost::real exp(boost::real number)
    4. boost::real log(boost::real number)
    5. boost::real sin(boost::real number)
    6. boost::real cos(boost::real number)
    7. boost::real tan(boost::real number)
    8. boost::real sec(boost::real number)
    9. boost::real cosec(boost::real number)
    10.boost::real cot(boost::real number) 
>(1) **Power function (<i> a<sup>b</sup></i> )** 
> Takes number and power as input and returns number <sup> power </sup> as output. Fractional power of a negative number is not supported, as the result is a imaginary number (of form a+ib) and imaginary numbers are not supported in the library. The function will throw "non_integral_power_of_negative_number" exception.
>
>(2) **Square Root function**
> This functions returns square root of the number, number<sup> 1/2</sup>. Here also, if we feed a negative number, the functions throws an error. As the square root is basically number raised to power 0.5, which is an fractional power so if the number is negative, then the result would be an imaginary number. If negative number is fed to functions, it throws "sqrt_not_defined_for_negative_number" error.
>
>(3) **Exponent function (<i> e<sup>x</sup></i> )**
> Gives exponent of the number fed or return <i> e<sup>number</sup></i>.
>
>(4) **Logarithmic funtion**
> Return <i>log<sub>e</sub>(number)</i>. The <i>logarithm</i> only supports positive number or we can say non negative number (-&#8734;, 0] are out of domain, so if a non negative number is fed then function throws "logarithm_not_defined_for_non_positive_number" error.
>
>(5) - (10) **Trigonometric functions**
> These are trigonometric functions, and their name corresponds to the trigonometric function which they represent. 

## boost::real::const_precision_iterator interface

### Constructors
    1. boost::real::const_precision_iterator()
    2. boost::real::const_precision_iterator(real const* x)
    3. boost::real::const_precision_iterator(const boost::real::const_precision_iterator& x)
    
> (1) ** Default constructor **
> Create an empty iterator that does not point any number and thus, cannot yet be iterated.
>
> (2) Creates an iterator that points to the x number and iterates the number precision. If the number is deleted, the iterator behaviour is undefined.
>
> (3) ** Copy constructor ** 
> Creates an iterator that points the same number than the x iterator does and is initialized in the same precision that x.


### Operators
    1. void operator++()
    2. bool operator==(const boost::real::real::const_precision_iterator& other)
    3. bool operator!=(const boost::real::real::const_precision_iterator& other)
    
> (1) Increases the pointer to the next precision interval of the pointed number. If the pointed number is represented by a real_explicit number and the full number precision is reached, then the operator has no effect because the number approximation lower and upper boundaries are equals and the number interval is the number itself.
>
> (2) It compare by value equality; two boost::real::real::const_precision_iterators are equals if they are pointing to the same real number and are in the same precision iteration.
>
> (3) It compare by value not equal; two boost::real::real::const_precision_iterators.

## Examples

### Example 1
Defining simple real explicit numbers and creating an operation between them.
```cpp
#include <iostream>
#include <string>
#include <real/real.hpp>

int main() {
    boost::real::real c = "0.999999"_r;
    boost::real::real d = "0.999999"_r;
    boost::real::real e = c + d;

    std::cout << "c: " << c << std::endl;
    std::cout << "d: " << d << std::endl;

    for (auto it = e.get_real_itr().cbegin(); it != e.get_real_itr().cend(); ++it) {
        std::cout << "e iteration " << it.get_interval() << std::endl;
    }

    boost::real::real g = "0.999998"_r;

    if (g < d) {
        std::cout << "g < d --> true" << std::endl;
    } else {
        std::cout << "g < d --> false" << std::endl;
    }

    boost::real::real h = d - g;

    std::cout << "d: " << d << std::endl;
    std::cout << "g: " << g << std::endl;
    std::cout << "h: " << h << std::endl;

    return 0;
}
```

### Output
```cpp
c: [0.99999899999999999999999999999999999999999999999999999999999999999999999999999999999999999933600778729, 0.99999899999999999999999999999999999999999999999999999999999999999999999999999999999999999982691714296]
d: [0.99999899999999999999999999999999999999999999999999999999999999999999999999999999999999999933600778729, 0.99999899999999999999999999999999999999999999999999999999999999999999999999999999999999999982691714296]
e iteration [1.99999799765646084706, 1.99999800138175115246]
e iteration [1.99999799999999999805656152704, 1.99999800000000000152600849191]
e iteration [1.99999799999999999999999999829434064267, 1.99999800000000000000000000152551492851]
e iteration [1.99999799999999999999999999999999999840487258578, 1.99999800000000000000000000000000000141413814631]
e iteration [1.99999799999999999999999999999999999999999999839920146096, 1.99999800000000000000000000000000000000000000120179841572]
e iteration [1.99999799999999999999999999999999999999999999999999999776247829313, 1.9999980000000000000000000000000000000000000000000000003726001095]
e iteration [1.99999799999999999999999999999999999999999999999999999999999999774936870769, 1.9999980000000000000000000000000000000000000000000000000000000001802340823]
e iteration [1.99999799999999999999999999999999999999999999999999999999999999999999999807650128953, 1.99999800000000000000000000000000000000000000000000000000000000000000000034042109297]
e iteration [1.99999799999999999999999999999999999999999999999999999999999999999999999999999999850405365298, 1.99999800000000000000000000000000000000000000000000000000000000000000000000000000061249327697]
g < d --> true
d: [0.99999899999999999999999999999999999999999999999999999999999999999999999999999999999999999933600778729, 0.99999899999999999999999999999999999999999999999999999999999999999999999999999999999999999982691714296]
g: [0.9999979999999999999999999999999999999999999999999999999999999999999999999999999999999999991629249303, 0.99999799999999999999999999999999999999999999999999999999999999999999999999999999999999999965383428597]
h: [0.00000099999999999999999999999999999999999999999999999999999999999999999999999999999999999968217350127, 0.00000100000000000000000000000000000000000000000000000000000000000000000000000000000000000066399221262]
```
### Example 2
Use of specialized data types for integer and rational types of numbers.
```cpp
#include <iostream>
#include <string>
#include <real/real.hpp>

int main(){
    boost::real::real a("4/5", boost::real::TYPE::RATIONAL);
    boost::real::real b("5", "integer");
    boost::real::real c = a * b; 
    std::cout<<"(4/5)*5 = "<<c<<std::endl;

    a = "10"_integer;
    b = "20"_rational;
    c = b/a;
    std::cout<<"20/10 = "<<c<<std::endl; 
    return 0;
}
```
### Output
```cpp
(4/5)*5 = 4
20/10 = 2
```

### Example 3
Use of power, exp and logarithm functions.
```cpp
#include <iostream>
#include <string>
#include <real/real.hpp>

using real = boost::real::real<int>;
using type = boost::real::TYPE;

int main(){
    real a("2", type::INTEGER); // declared as integer data type
    real b("2"); // declare as real explicit number
    real c = real::power(a, b);
    std::cout<<"2^2 = "<<c<<std::endl;    

    c = real::sqrt(a);
    std::cout<<"sqrt(2) = "<<c<<std::endl;
    for (auto it = c.get_real_itr().cbegin(); it != c.get_real_itr().cend(); ++it) {
        std::cout << "c iteration " << it.get_interval() << std::endl;
    }
    
    std::cout<<std::endl;
    real d = real::power(c, b);
    std::cout<<"sqrt(2)^2 = "<<d<<std::endl;
    
    a = real("5.7893452");
    b = real::exp(a);
    std::cout<< "b = exp(5.7893452)"<<std::endl;
    for (auto it = b.get_real_itr().cbegin(); it != b.get_real_itr().cend(); ++it) {
        std::cout << "b iteration " << it.get_interval() << std::endl;
    }

    c = real::log(a);
    std::cout<<std::endl<<"c = log(5.7893452)"<<std::endl;
    for (auto it = c.get_real_itr().cbegin(); it != c.get_real_itr().cend(); ++it) {
        std::cout << "c iteration " << it.get_interval() << std::endl;
    }
    return 0;
}
```
### Output
```cpp
2^2 = 4
sqrt(2) = [1.41421356237309504880168872420969807856967187537694807317667973799073247846210703824455626906, 1.41421356237309504880168872420969807856967187537694807317667973799073247846210703929877608105]
c iteration [0, 2]
c iteration [1.4142135612931355113, 1.414213563155780664]
c iteration [1.41421356237309504771882979579, 1.41421356237309504945355327823]
c iteration [1.41421356237309504880168872268419715022, 1.41421356237309504880168872429978429314]
c iteration [1.41421356237309504880168872420969807748793104591, 1.41421356237309504880168872420969807899256382617]
c iteration [1.41421356237309504880168872420969807856967187429260878896, 1.41421356237309504880168872420969807856967187569390726634]
c iteration [1.41421356237309504880168872420969807856967187537694807235712586083, 1.41421356237309504880168872420969807856967187537694807366218676902]
c iteration [1.41421356237309504880168872420969807856967187537694807317667973727681290323, 1.41421356237309504880168872420969807856967187537694807317667973849224559054]
c iteration [1.41421356237309504880168872420969807856967187537694807317667973799073247782435363771, 1.41421356237309504880168872420969807856967187537694807317667973799073247895631353943]

sqrt(2)^2 = [1.99999999999999999999999999999999999999999999999999999999999999999999999999999999828645041643286755050686006659533432017866892149815259493304241342943026179263302251985740872, 2.00000000000000000000000000000000000000000000000000000000000000000000000000000000126823432812658600908997445370289935447077017980067733307582202211968265635911670874892079273]
b = exp(5.7893452)
b iteration [53, 404]
b iteration [326.79896596036658801206, 326.79896657131419809779]
b iteration [326.79896633785966276370220447871, 326.79896633785966333269150671875]
b iteration [326.79896633785966320252799767479510460165, 326.79896633785966320252799820632327462247]
b iteration [326.79896633785966320252799794778214250652298662653, 326.79896633785966320252799794778214300004253855277]
b iteration [326.7989663378596632025279979477821427593817405130885539864, 326.79896633785966320252799794778214275938174097341510380423]
b iteration [326.79896633785966320252799794778214275938174090441527940672129521515, 326.79896633785966320252799794778214275938174090441527983478127309979]
b iteration [326.79896633785966320252799794778214275938174090441527970907222338267738767102, 326.79896633785966320252799794778214275938174090441527970907222378194702545062]
b iteration [326.79896633785966320252799794778214275938174090441527970907222374555635709806047973128, 326.79896633785966320252799794778214275938174090441527970907222374555635746934332749572]

c = log(5.7893452)
c iteration [0, 2]
c iteration [1.75601919229332207197, 1.75601919415596722467]
c iteration [1.75601919365244082427619048642, 1.75601919365244082601091396886]
c iteration [1.75601919365244082540382634155702620033, 1.75601919365244082540382634398040691472]
c iteration [1.75601919365244082540382634326620308056183430644, 1.75601919365244082540382634326620308281878347684]
c iteration [1.75601919365244082540382634326620308214439845673423000187, 1.75601919365244082540382634326620308214439845813552847924]
c iteration [1.75601919365244082540382634326620308214439845791921676163955823715, 1.75601919365244082540382634326620308214439845791921676294461914534]
c iteration [1.75601919365244082540382634326620308214439845791921676263247861654663653931, 1.75601919365244082540382634326620308214439845791921676263247861836978557027]
c iteration [1.75601919365244082540382634326620308214439845791921676263247861783286263563401048786, 1.75601919365244082540382634326620308214439845791921676263247861783286263733195034044]
```

## References
    1. Computable calculus / Oliver Aberth, San Diego : Academic Press, c2001
    2. Lambov, B. (2007). RealLib: An efficient implementation of exact real arithmetic. Mathematical Structures in Computer Science, 17(1), 81-98.
    3. Aberth, O., & Schaefer, M. J. (1992). Precise computation using approximation_interval arithmetic, via C++. ACM Transactions on Mathematical Software (TOMS), 18(4), 481-491.
    4. https://en.cppreference.com/w/cpp/concept/ForwardIterator
