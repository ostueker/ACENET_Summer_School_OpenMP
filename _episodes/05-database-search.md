---
title: "Searching through data"
teaching: 20
exercises: 20
questions:
- "How to search in parallel"
objectives:
- "Use general parallel sections"
- "Have a single thread execute code"
keypoints:
- "OpenMP can manage general parallel sections"
- "You can use 'pragma omp single' to have a single thread execute something"
---

So far, we have looked at parallelizing loops. OpenMP also allows you to use general parallel sections where code blocks are executed by all of the available threads. In these cases, it is up to you as a programmer to manage what work gets done by each thread. A basic example would look like the following.

~~~
#include <stdio.h>
#include <stdlib.h>
#include <omp.h>

int main(int argc, char **argv) {
   int id, size;
   #pragma omp parallel private(id,size)
   {
      size = omp_get_num_threads();
      id = omp_get_thread_num();
      printf("This is thread %d of %d\n", id, size);
   }
}
~~~
{: .source}

> ## Private variables
> What happens if you forget the private keyword?
{: .challenge}

Using this as a starting point, we could use this code to have each available thread do something interesting. For example, we could write the text out to a series of individual files.

## Single threaded function

There are times when you may need to drop out of a parallel section in order to have a single one of the threads executing some code. There is a keyword available, 'single', that allows the first thread to see it to execute some single code chunk.

> ## Which thread runs first?
> How can you find out which thread gets to run first in a parallel section?
> > ## Pragma omp single
> > ~~~
> > #include <stdio.h>
> > #include <stdlib.h>
> > #include <omp.h>
> > 
> > int main(int argc, char **argv) {
> >    int id, size;
> >    #pragma omp parallel private(id,size)
> >    {
> >       size = omp_get_num_threads();
> >       id = omp_get_thread_num();
> >       #pragma omp single
> >       printf("This thread %d of %d is first\n", id, size);
> >    }
> > }
> > ~~~
> > {: .source}
> {: .solution}
{: .challenge}

