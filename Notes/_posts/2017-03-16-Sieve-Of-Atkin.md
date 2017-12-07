## Introduction
For some reason, everybody is obsessed with primes. Well, not everybody. Rather mathematicians and someone else. And an interesting problem with primes is how to generate them fast. Really fast. One of the proposals to reach this goal, which is pretty recent, is the Sieve of Atkin.

This sieve has in my opinion too much mathematics concepts for someone who doesn't have a good mathematical knowledge, like me. That's why I'm going to try to explain it with "easy" terms.

<br/>
## Paper
If you are interested in this algorithm or the concepts that it covers I beg you to take a look at the <a href="http://www.ams.org/mcom/2004-73-246/S0025-5718-03-01501-1/S0025-5718-03-01501-1.pdf">paper</a> published by <a href="https://en.wikipedia.org/wiki/A._O._L._Atkin">Atkin</a> and <a href="https://en.wikipedia.org/wiki/Daniel_J._Bernstein">Bernstein</a> which is much more complete (although much more complex).<!--more-->

<br/>
## Modular Arithmetic
First we are going to take all the <a href="https://en.wikipedia.org/wiki/Modular_multiplicative_inverse">invertible</a> numbers in Z<sub>60</sub>. Z<sub>60 </sub>represents the natural numbers in module 60. So Z<sub>60</sub> is set of the numbers from 0 to 59.

If a number is >= 60 you can convert it to Z<sub>60</sub> by doing the mod 60 operation. And the invertible numbers in Z<sub>n</sub> are those numbers that have a modular multiplicative inverse. Which means that if you multiply the number in Z<sub>n</sub> by another number in Z<sub>n</sub> you obtain 1 in module n. And which numbers have inverse? The numbers in Z<sub>n</sub> that are <a href="https://en.wikipedia.org/wiki/Coprime_integers">coprime</a> with n. <br />Two numbers are coprime if their greatest common divisor is 1. For example:
>5 is the highest number that both 25 and 60 are divisible by.<br/> 
>The only common divisor of 7 and 60 is 1, so they are coprime.

In the case of Z<sub>60</sub>, the invertible numbers are:<br/>
1 7 11 13 17 19 23 29 31 37 41 43 47 49 53 59

<br/>
## Binary Quadratic Form
Now, we are going to generate numbers with the form n = 60k + s where *k* is just a natural number and *s* an invertible number in Z<sub>60</sub>. If we want to generate all the primes from 60 to a limit, *k* will take values from 1 to &lfloor;*limit*/60&rfloor;-1.
>For example we will generate (60+1), (60+7) ... (60+59), (120+1) ...

We know that all the numbers with the previous form are primes candidates. Each one will be prime if it satisfies some rules which are based on binary quadratic forms. The sieve uses 3 algorithms (named 3.1, 3.2, and 3.3) depending on the module of the *s* used to generate the number. 

There are three cases depending on the form of *s*. The number can be prime if the equation associated with its form has an odd number of solutions:

 - <b>s &isin; 1+4Z</b> Which means that <b>s mod 4 = 1</b>. It will be prime if the equation <b>4x<sup>2</sup> + y<sup>2</sup> = n</b> has an odd number of solutions (x,y) where x,y > 0.
 - <b>s &isin; 1+6Z</b> Which means that <b>s mod 6 = 1</b>. It will be prime if the equation <b>3x<sup>2</sup> + y<sup>2</sup> = n</b> has an odd number of solutions (x,y) where x,y > 0.
 - <b>s &isin; 11+12Z</b> Which means that <b>s mod 12 = 11</b>. It will be prime if the equation <b>3x<sup>2</sup> - y<sup>2</sup> = n</b> has an odd number of solutions (x,y) where x > y > 0.

For example, the number 67 (60 + 7):<br/>
>s=7 so it has the form 1 + 6Z. The corresponding equation is 3x<sup>2</sup> + y<sup>2</sup> = n.
>The solutions that can be found are: 
>x=1 y=8. So there are an odd number of solutions. Therefore 67 is still a prime candidate.

The hard part of this is how to find the solutions for those equations. The paper exposes efficient methods for this task but I'm not going to explain them because are quite complex and they are not important to understand the philosophy of this algorithm.

<br/>
## Square Free
Finally, if after the previous algorithm the number is still a prime candidate we have to check if the number is square free. If it is, the number is prime!
<br/>
By the way, square free means that if you factorize the number you won't find any factor repeated. For example:<br/>
>12 is not square free because it's equal to 2·2·3, and we can see that the factor 2 is repeated.

How do we know if a number is square free? Well, actually we don't know any algorithm that check it in polynomial time, so you can check if the number is divisible by the square of any of the primes that you have already calculated. 

<br/>
## Performance
I'm sure that if you are interested in this algorithm is because of its performance. Unfortunately, I can't swear that this sieve is faster than the <a href="https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes">Sieve of Eratosthenes</a>. I don't pretend to analyze deeply the reasons, although if you are interested in knowing the details I recommend you to have a look at <a href="http://stackoverflow.com/questions/19388106/the-sieve-of-atkin/22161595#22161595">this answer</a> in StackOverflow.
<br/><br/>
Remember that performance is not only about the number of operations that you have to do to obtain the result; is also about cache misses, instruction level parallelism, branch predictions and much more. 

<br/>
## Implementation
I have made an implementation of this algorithm focused on making the code illustrative and simple, forgetting about performance related issues that would make it harder to understand.<br/>
<a href="https://github.com/fylux/SieveOfAtkin">Implementation in C++</a><br/>
<a href="https://github.com/fylux/SieveOfAtkin">Implementation in Maple</a>


## Acknowledgements
I want to thank Dr. G.M. Diaz-Toca her support, especially for understanding the original paper which requires a good mathematical background. I also want to thank her for her implementation of this sieve in Maple. 

## References
A.O.L. Atkin, D.J. Bernstein, <a href="http://www.ams.org/mcom/2004-73-246/S0025-5718-03-01501-1/S0025-5718-03-01501-1.pdf">Prime sieves using binary quadratic forms</a>, Math. Comp. 73 (2004), 1023-1030.
