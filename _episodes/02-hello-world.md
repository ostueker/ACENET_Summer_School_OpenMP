---
title: "Hello World"
teaching: 20
exercises: 20
questions:
- "How do you compile and run an OpenMP program?"
- "What are OpenMP pragmas?"
objectives:
- "Learning about pragmas"
- "Compiling and running an OpenMP program"
keypoints:
- "Pragmas are directives to the compiler to parallelize something"
- "If the compiler doesn't recognize OpenMP pragmas, it will compile a single-threaded program"
---

Since OpenMP is an extension to the compiler, you need to be able to tell the compiler when and where to add the code necessary to create and use threads for the parallel sections. This is handled through special statements called pragmas. To a compiler that doesn't understand OpenMP, pragmas look like comments. The basic forms are:

C/C++
~~~
#pragma omp ...
~~~
{: .source}

FORTRAN
~~~
!OMP ...
~~~
{: .source}

## Hello World

How do we add in parallelism to the basic hello world program? The very first pragma that we will look at is the parallel pragma.

~~~
#include <stdio.h>
#include <stdlib.h>

#include <omp.h>

int main(int argc, char **argv) {
   #pragma omp parallel
   printf("Hello World\n");
}
~~~
{: .source}

To compile it, you'll need to add an extra flag to tell the compiler to treat the source code as an OpenMP program.

~~~
gcc -fopenmp -o hello hello.c
~~~
{: .source}

When you run this program, you should see the output "Hello World" multiple times. But how many? By default, OpenMP will look at the machine that it is running on and see how many computer cores there are. It will then launch a thread for each core. You can control the number of threads, however, with environment variables. If you wanted to only have 3 threads used, you could do the following:

~~~
export OMP_NUM_THREADS=3
./hello_world
~~~
{: .bash}

> ## Using multiple cores
> Try running the hello world program with a number of different threads? Can you use more threads than the cores on the machine?
{: .challenge}

> ## OpenMP with Grid Engine
> When you wish to submit an OpenMP job to Grid Engine, you can use the following boilerplate.
> ~~~
> #$ -l h_rt=1:0:0
> #$ -pe openmp 3
> export OMP_NUM_THREADS=$NSLOTS
> ./hello_world
> ~~~
> {: .bash}
{: .callout}

How can you tell which thread is doing what? The OpenMP specification includes a number of functions that are made available through the included header file "omp.h". One of them is the function "omp_get_thread_num()", to get an ID of the thread running the code.

~~~
#include <stdio.h>
#include <stdlib.h>

#include <omp.h>

int main(int argc, char **argv) {
   int id;
   #pragma omp parallel
   id = omp_get_thread_num();
   printf("Hello World from thread %d\n", id);
}
~~~
{: .source}

Here, you will get each thread tagging their output with their unique ID, a number between 0 and NUM_THREADS-1.

> ## Thread ordering
> What order do the threads write out their messages in?
{: .challenge}

You should see something interesting here.

> ## Conditional compilation
> Try compiling the code without the -fopenmp flag. What happens? Can you figure out how to fix it?
>
> Hint: The compiler defines a preprocessor variable \_OPENMP
> > ## Solution
> > ~~~
> >
> > #include <stdio.h>
> > #include <stdlib.h>
> > #include <omp.h>
> > 
> > int main(int argc, char **argv) {
> >    int id = 0;
> >    #pragma omp parallel
> >    {
> > #ifdef _OPENMP
> >    id = omp_get_thread_num();
> > #endif
> >    printf("Hello World from thread %d\n", id);
> >    }
> > }
> > ~~~
> > {: .source}
> {: .solution}
{: .challenge}
