---
title: "Numeric Integration - Calculating Areas"
teaching: 20
exercises: 20
questions:
- "How can we calculate integrals?"
objectives:
- "Learn about critical sections"
keypoints:
- "Need to control access to global variables"
---

In this section, we will use the problem of numeric integration, i.e. calculating areas under curves, to look at how to control access to global variables. As our example, let's say we wanted to integrate the sine function from 0 to Pi. This is the same as the area under the first half of a sine curve. The single-threaded version is below.

~~~
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

int main(int argc, char **argv) {
   int steps = 1000;
   float delta = M_PI/steps;
   float total = 0.0;
   int i;
   for (i=0; i<steps; i++) {
      total = total + sin(delta*i) * delta;
   }
   printf("The integral of sine from 0 to Pi is %f\n", total);
}
~~~
{: .source} 

> ## Compiling with math
> In order to include the math functions, you need to link in the math library. In GCC, you would use the following:
> ~~~
> gcc -o pi pi.c -lm
> ./pi
> ~~~
> {: .bash}
{: .callout}

The answer in this case should be 2. It will be off by a small amount because of the limits of computer representations of numbers.

> ## Step size
> What happens if you change the step size?
{: .challenge}

You can get better or worse accuracy based on step size. We still want to see what happens to the time this program takes, but we will do it a different way. Since we just want to see the total amount of time, we will use the program time.

> ## Timing
> You can use the time utility to get the amount of time it takes for a program to run.
> ~~~
> time ./pi
> ~~~
> {: .bash}
{: .callout}

> ## Parallelizing numerical integration
> How would you parallelize this code to get it to run faster?
> > ## Parallel integration
> > ~~~
> > #include <stdio.h>
> > #include <stdlib.h>
> > #include <math.h>
> > #include <omp.h>
> >
> > int main(int argc, char **argv) {
> >    int steps = 1000;
> >    float delta = M_PI/steps;
> >    float total = 0.0;
> >    int i;
> >    #pragma omp parallel for
> >    for (i=0; i<steps; i++) {
> >       #pragma omp critical
> >       total = total + sin(delta*i) * delta;
> >    }
> >    printf("The integral of sine from 0 to Pi is %f\n", total);
> > }
> > ~~~
> > {: .source}
> {: .solution}
{: .challenge}

This challenge highlights a problem called a race condition. Since we are updating a global variable, there is a race between the various threads as to who can read and then write the value of 'total'. Multiple threads could read the current value, before a working thread can write their addition. So these reading threads essentially miss out on some additions to the total. This is handled by adding a critical section. A critical section only allows one thread at a time to run some code block.

The `critical` pragma is a very general construct that lets you ensure a code
line is executed exclusively.  However, making a sum is a very common operation
in computing, and OpenMP provides a specific mechanism to handle this: 
*Reduction variables*. We'll look at those in the next section.
