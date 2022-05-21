# Writing an OS in Rust

This repo creates a small operating system in the Rust programming language. Each post is a small tutorial and includes all needed code, so you can follow along if you like. The source code is also available in the corresponding Github repository.

1. [Bare Bones](#bare-bones)
2. [Interrupts](#interrupts)
3. [Memory Management](#memory-management)
4. [Multitasking](#multitasking)

## Bare Bones

**[A Freestanding Rust Binary]**

The first step in creating our own operating system kernel is to create a Rust executable that does not link the standard library. This makes it possible to run Rust code on the [bare metal] without an underlying operating system.

[a freestanding rust binary]: 01-freestanding-rust-binary.md
[bare metal]: https://en.wikipedia.org/wiki/Bare_machine

**[A Minimal Rust Kernel]**

In this post we create a minimal 64-bit Rust kernel for the x86 architecture. We build upon the freestanding Rust binary from the previous post to create a bootable disk image, that prints something to the screen.

[a minimal rust kernel]: 02-minimal-rust-kernel.md

**[VGA Text Mode]**

The [VGA] text mode is a simple way to print text to the screen. In this post, we create an interface that makes its usage safe and simple, by encapsulating all unsafety in a separate module. We also implement support for [Rust’s formatting macros].

[vga text mode]: 03-vga-text-buffer.md
[vga]: https://en.wikipedia.org/wiki/VGA_text_mode
[rust’s formatting macros]: https://doc.rust-lang.org/std/fmt/#related-macros

**[Testing]**

This post explores unit and integration testing in `no_std` executables. We will use Rust’s support for custom test frameworks to execute test functions inside our kernel. To report the results out of QEMU, we will use different features of QEMU and the `bootimage` tool.

[testing]: 04-testing.md

## Interrupts

**[CPU Exceptions]**

CPU exceptions occur in various erroneous situations, for example when accessing an invalid memory address or when dividing by zero. To react to them we have to set up an _interrupt descriptor table_ that provides handler functions. At the end of this post, our kernel will be able to catch [breakpoint exceptions] and to resume normal execution afterwards.

[cpu exceptions]: 05-cpu-exceptions.md
[breakpoint exceptions]: https://wiki.osdev.org/Exceptions#Breakpoint

**[Double Faults]**

This post explores the double fault exception in detail, which occurs when the CPU fails to invoke an exception handler. By handling this exception we avoid fatal triple faults that cause a system reset. To prevent triple faults in all cases we also set up an Interrupt Stack Table to catch double faults on a separate kernel stack.

[double faults]: 06-double-faults.md

**[Hardware Interrupts]**

In this post we set up the programmable interrupt controller to correctly forward hardware interrupts to the CPU. To handle these interrupts we add new entries to our interrupt descriptor table, just like we did for our exception handlers. We will learn how to get periodic timer interrupts and how to get input from the keyboard.

[hardware interrupts]: 07-hardware-interrupts.md

## Memory Management

**[Introduction to Paging]**

This post introduces _paging_, a very common memory management scheme that we will also use for our operating system. It explains why memory isolation is needed, how _segmentation_ works, what _virtual memory_ is, and how paging solves memory fragmentation issues. It also explores the layout of multilevel page tables on the x86_64 architecture.

[introduction to paging]: 08-paging-introduction.md

**[Paging Implementation]**

This post shows how to implement paging support in our kernel. It first explores different techniques to make the physical page table frames accessible to the kernel and discusses their respective advantages and drawbacks. It then implements an address translation function and a function to create a new mapping.

[paging implementation]: 09-paging-implementation.md

**[Heap Allocation]**

This post adds support for heap allocation to our kernel. First, it gives an introduction to dynamic memory and shows how the borrow checker prevents common allocation errors. It then implements the basic allocation interface of Rust, creates a heap memory region, and sets up an allocator crate. At the end of this post all the allocation and collection types of the built-in `alloc` crate will be available to our kernel.

[heap allocation]: 10-heap-allocation.md

**[Allocator Designs]**

This post explains how to implement heap allocators from scratch. It presents and discusses different allocator designs, including bump allocation, linked list allocation, and fixed-size block allocation. For each of the three designs, we will create a basic implementation that can be used for our kernel.

[allocator designs]: 11-allocator-designs.md

## Multitasking

**[Async/Await]**

In this post we explore cooperative _multitasking_ and the _async/await_ feature of Rust. We take a detailed look how async/await works in Rust, including the design of the `Future` trait, the state machine transformation, and _pinning_. We then add basic support for async/await to our kernel by creating an asynchronous keyboard task and a basic executor.

[async/await]: 12-async-await.md
