---
title: "Matrix Multiplication"
teaching: 20
exercises: 20
questions:
- "How can you do matrix multiplication?"
objectives:
- "Learn the parallel for pragma"
- "Use it to break work up across multiple threads"
keypoints:
- "The parallel for pragma automatically parallelizes code"
---

One of the classic problems in parallel programming is linear algebra, in all of its beauty and complexity. We will be using a couple of simple problems in order to see how we can spread work out around whatever compute resources you may have available, and how OpenMP can make this task easier.

The simplest problem is applying some function to an array of numbers. An example would be multiplying each value by some constant number. In C, you would use a for loop to do this multiplication for each entry. A single threaded example would look like the following.

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
   printf("Total time is %f ms\n", time_total/1000000);
}
~~~
{: .source}

> ## Compiling with GCC
> Don't forget to add the option '-fopenmp' to your GCC compiler command.
{: .callout}

Here, we added a line to include the header file 'omp.h' and a line to include the 'parallel for' pragma.

> ## Using more threads
> What happens to the runtime when you change the number of threads to be used?
{: .challenge}

Moving up to a 2D matrix adds a new layer of looping. The basic code looks like the following.

~~~
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <omp.h>

int main(int argc, char **argv) {
   struct timespec ts_start, ts_end;
   int size = 1000;
   int multiplier = 2;
   int a[size][size];
   int c[size][size];
   int i,j;
   float time_total;
   clock_gettime(CLOCK_MONOTONIC, &ts_start);
   #pragma omp parallel for
   for (i = 0; i<size; i++) {
      for (j=0; j<size; j++) {
         c[i,j] = multiplier * a[i,j];
      }
   }
   clock_gettime(CLOCK_MONOTONIC, &ts_end);
   time_total = (ts_end.tv_sec - ts_start.tv_sec)*1000000000 + (ts_end.tv_nsec - ts_start.tv_nsec);
   printf("Total time is %f ms\n", time_total/1000000);
}
~~~
{: .source}

> ## Time/threads?
> What happens to the run time as you change the number of threads available?
{: .challenge}

We have been setting the number of threads to be used with an environment variable. There is also a function available to do this from within your code.

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
   omp_set_num_threads(10);
   clock_gettime(CLOCK_MONOTONIC, &ts_start);
   #pragma omp parallel for
   for (i = 0; i<size; i++) {
      c[i] = multiplier * a[i];
   }
   clock_gettime(CLOCK_MONOTONIC, &ts_end);
   time_total = (ts_end.tv_sec - ts_start.tv_sec)*1000000000 + (ts_end.tv_nsec - ts_start.tv_nsec);
   printf("Total time is %f ms\n", time_total/1000000);
}
~~~
{: .source}

Here we added a call to 'omp_set_num_threads()' in order to explicitly set the number of threads requested to 10.

> ## Compiler gotchas
> This call is one of the optional ones. You particular compiler/version may or may not support it.
> If it doesn't support the call, it will simply fail silently, allowing your program to finish.
{: .callout}
