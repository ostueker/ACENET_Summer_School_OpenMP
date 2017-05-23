---
title: "Hello World"
teaching: 20
exercises: 20
questions:
- "How do you add OpenMP pragmas?"
- "What pragmas are available?"
- "How do you compile OpenMP programs?"
- "How do you affect the running of OpenMP programs?"
objectives:
- "Learning about pragmas"
- "Adding pragmas to your program"
- "Compiling an OpenMP program"
- "Running an OpenMP program"
keypoints:
- "Pragmas look like comments"
- "If the compiler doesn't recognize OpenMP pragmas, it will compile a single-threaded program"
---

> ## GCC on ACENET
> To use GCC on the ACENET clusters, you need to load the appropriate module.
> ~~~
> module purge
> module load gcc
> ~~~
> {: .bash}
{: .callout}

As with every programming course, we will start off by looking at a hello world program. The classic form for the C programming language is the following.

~~~
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv) {
   printf("Hello World\n");
}
~~~
{: .source}

To compile this program, you would use the following command.

~~~
gcc -o hello_world hello_world.c
~~~
{: .bash}

When running it, you should get the following results.

~~~
./hello_world
~~~
{: .bash}
~~~
Hello World
~~~
{: .output}

How do we add in parallelism to this basic program? The very first pragma that we will look at is the parallel pragma.

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
gcc -fopenmp -o hello_world hello_world.c
~~~
{: .source}


When you run this program, you should see the output "Hello World" multiple times. But how many? By default, OpenMP will look at the machine that it is running on and see how many computer cores there are. It will then launch a thread for each core. You can control the number of threads, however, with environment variables. If you wanted to only have 3 threads used, you could do the following:

~~~
export OMP_NUM_THREADS=3
./hello_world
~~~
{: .bash}

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

> What order do the threads write out their messages in?
{: .challenge}

You should see something interesting here.
