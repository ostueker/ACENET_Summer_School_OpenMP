---
title: "Introduction"
teaching: 15
exercises: 0
questions:
- "What is shared-memory programming?"
- "What is OpenMP?"
objectives:
- "Understand how shared-memory programs work."
keypoints:
- "OpenMP programs can process data very quickly"
- "Programs are limited to a single physical machine"
---

Parallel programs come in two broad flavors: shared-memory and message-passing. In this workshop, we will be looking at shared-memory programming, with a focus on OpenMP programming. But, what is shared-memory programming? In any parallel program, the general idea is to have multiple threads of execution so that you can break up your problem and have each thread handle one part. These multiple threads need to be able to communicate with each other as your program runs. In a shared-memory program, this communication happens through the use of global variables stored in the global memory of the local machine. This means that communication between the various threads is extremely fast, as it happens at the speed of RAM access. The major downfall is that your program will be limited to a single physical machine.
