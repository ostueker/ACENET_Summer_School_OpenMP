---
title: "Linear algebra"
teaching: 20
exercises: 30
questions:
- "How do I parallelize a loop?"
objectives:
- "Use the PARALLEL FOR pragma (or PARALLEL DO)"
- "Use the PRIVATE clause"
keypoints:
- "The PARALLEL FOR (or PARALLEL DO) pragma makes a loop execute in parallel"
- "A single variable accessed by different threads can cause wrong results"
- "The PRIVATE clause makes a copy of a variable for each thread"
---

One of the classic problems in parallel programming is linear algebra, in all of its beauty and complexity. We will use a couple of simple problems to see how to execute loops in paralle using OpenMP.

## Multiply an array by a constant

The simplest problem is applying some function to an array of numbers. An example is multiplying each value by some constant number. In serial you would use a for loop (in C; a DO loop in Fortran) to do this multiplication for each element, like this:

~~~
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main(int argc, char **argv) {
   struct timespec ts_start, ts_end;
   int size = 1000000;
   int multiplier = 2;
   int a[size];
   int c[size];
   int i;
   float time_total;
   clock_gettime(CLOCK_MONOTONIC, &ts_start);
   for (i = 0; i<size; i++) {
      c[i] = multiplier * a[i];
   }
   clock_gettime(CLOCK_MONOTONIC, &ts_end);
   time_total = (ts_end.tv_sec - ts_start.tv_sec)*1000000000 + (ts_end.tv_nsec - ts_start.tv_nsec);
   printf("Total time is %f ms\n", time_total/1000000);
}
~~~
{: .source}

We added some calls to 'clock_gettime()' from the 'time.h' header file to get the start and end times of the heavy work being done by the for loop. In this case, we get a count of how many seconds and how many nanoseconds elapsed, given in two parts of the time structure. We did some math to get the elapsed time in milliseconds.

> ## Time and Size
> What happens to the run time of your program through multiple runs?
> What happens if you change the size and recompile and rerun?
{: .challenge}

How complicated is it to turn this program into a parallel program?

~~~
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <omp.h>

int main(int argc, char **argv) {
   struct timespec ts_start, ts_end;
   int size = 1000000;
   int multiplier = 2;
   int a[size];
   int c[size];
   int i;
   float time_total;
   clock_gettime(CLOCK_MONOTONIC, &ts_start);
   #pragma omp parallel for
   for (i = 0; i<size; i++) {
      c[i] = multiplier * a[i];
   }
   clock_gettime(CLOCK_MONOTONIC, &ts_end);
   time_total = (ts_end.tv_sec - ts_start.tv_sec)*1000000000 + (ts_end.tv_nsec - ts_start.tv_nsec);
   printf("Time in loop is %f ms.  %d iterations.\n", time_total/1000000, size);
}
~~~
{: .source}

> ## Compiling with GCC
> Don't forget to add the option '-fopenmp' to your GCC compiler command.
{: .callout}

Here, we added a line to include the header file 'omp.h' and a line to include the 'parallel for' pragma.

> ## Using more threads
> How many threads did you use?
> What happens to the runtime when you change the number of threads?
{: .challenge}

In this case, the number of iterations around the for loop gets divided across the number of threads available. In order to do this, however, OpenMP needs to know how many iterations there are in the for loop. It also can't change part way through the loops. To ensure this, there are some rules: 
* You must not change the value of 'size' within one of the iterations. 
* You must not use a call to 'break()' or 'exit()' within the for loop, either. These functions pop you out of the for loop before it is done.

## Summing the values in a matrix

Now let's try adding up the elements of a matrix.
Moving up to two dimensions adds a new layer of looping. The basic code looks like the following.

~~~
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <omp.h>

int main(int argc, char **argv) {
   struct timespec ts_start, ts_end;
   int size = 1000;
   int a[size][size];
   int i, j;
   // Set the matrix values to 1
   for (i=0; i<size; i++) {
      for (j=0; j<size; j++) {
         a[i][j] = 1;
      }
   }
   int c[size];
   // Zero the accumulator 
   for (i=0; i<size; i++) {
      c[i] = 0;
   }
   int total = 0;
   float time_total;
   clock_gettime(CLOCK_MONOTONIC, &ts_start);
   #pragma omp parallel for
   for (i = 0; i<size; i++) {
      for (j=0; j<size; j++) {
         c[i] = c[i] + a[i][j];
      }
   }
   for (i=0; i<size; i++) {
      total = total + c[i];
   }
   clock_gettime(CLOCK_MONOTONIC, &ts_end);
   time_total = (ts_end.tv_sec - ts_start.tv_sec)*1000000000 + (ts_end.tv_nsec - ts_start.tv_nsec);
   printf("Total is %d, time is %f ms\n", total, time_total/1000000);
}
~~~
{: .source}

> ## Is the result correct?
> What should be the result of this code? 
> Is that what it does?  If not, what might be wrong?
> Does it do the same for different values of OMP_NUM_THREADS?
> 
> > ## Solution
> > The elements all have value 1, and there are 1000*1000 of them, so the total should be 1,000,000. Why isn't it?
> > 
> > Remember that OpenMP threads *share memory*. This means that every thread
> > can see and access all of memory for the process. In the above case, 
> > multiple threads are all accessing the global variable `j` at the same time. 
> >
> > OpenMP includes a method to manage this correctly with the addition of a keyword, `private()`.
> > 
> > ~~~
> > #include <stdio.h>
> > #include <stdlib.h>
> > #include <time.h>
> > #include <omp.h>
> > 
> > int main(int argc, char **argv) {
> >    struct timespec ts_start, ts_end;
> >    int size = 1000;
> >    int a[size][size];
> >    int i, j;
> >    // Set the matrix values to 1
> >    for (i=0; i<size; i++) {
> >       for (j=0; j<size; j++) {
> >          a[i][j] = 1;
> >       }
> >    }
> >    int c[size];
> >    // Zero the accumulator
> >    for (i=0; i<size; i++) {
> >       c[i] = 0;
> >    }
> >    int total = 0;
> >    float time_total;
> >    clock_gettime(CLOCK_MONOTONIC, &ts_start);
> >    #pragma omp parallel for private(j)
> >    for (i=0; i<size; i++) {
> >       for (j=0; j<size; j++) {
> >          c[i] = c[i] + a[i][j];
> >       }
> >    }
> >    for (i=0; i<size; i++) {
> >       total = total + c[i];
> >    }
> >    clock_gettime(CLOCK_MONOTONIC, &ts_end);
> >    time_total = (ts_end.tv_sec - ts_start.tv_sec)*1000000000 + (ts_end.tv_nsec - ts_start.tv_nsec);
> >    printf("Total is %d, time is %f ms\n", total, time_total/1000000);
> > }
> > ~~~
> > {: .source}
> > 
> > This makes sure that every thread has their own private copy of `j` to be used for the inner for loop.
> {: .solution}
{: .challenge}

## Re-entrant functions

This leads to a very important idea, the concept of re-entrant functions. You need to be aware when writing multi-threaded programs whether functions you are using are reentrant or not. A classic example where this happens is when generating random numbers. These functions maintain state between calls to them. If they aren't written to be reentrant, then that internal state can get mixed up by multiple threads calling them at the same time.

> ## Time/threads?
> What happens to the run time as you change the number of threads available?
{: .challenge}

