---
layout: post
title: "Effective JAVA(3)"
category: 
tags: []
---
{% include JB/setup %}

##Item 6
###Eliminate obsolete object references
 
	public class Stack 
	{
		private Object[] elements;
		private int size = 0;
		private static final int DEFAULT_INITIAL_CAPACITY = 16;
		public Stack() 
		{
			elements = new Object[DEFAULT_INITIAL_CAPACITY];
		}
		public void push(Object e) 
		{
			ensureCapacity();
			elements[size++] = e;
		}
		public Object pop() 
		{
			if (size == 0)
			throw new EmptyStackException();
			return elements[--size];
		}
		/**
		* Ensure space for at least one more element, roughly
		* doubling the capacity each time the array needs to grow.
		*/
		private void ensureCapacity() 
		{
			if (elements.length == size)
			elements = Arrays.copyOf(elements, 2 * size + 1);
		}
	}
 
If a stack grows and then shrinks, the object that were popped off the stack will not be garbage collected.
 
The stack maintains obsolete references to these objects.
 
The corrected version of the pop method looks like this:

	public Object pop() 
	{
		if (size == 0)
		throw new EmptyStackException();
		Object result = elements[--size];
		elements[size] = null; // Eliminate obsolete reference
		return result;
	}
 
When programmers are first stung by this problem, they may overcompensate by nulling out every object references as soon as the program is finished using it. This is neither necessary nor desirable, as it clutters up the program unnecessarily.
 
**Nulling out object references should be the exception rather than the norm.**
 
Whenever a class manages its own memory, the programmer should be alert for memory leaks. Whenever an element is freed,any object references contained in the element should be nulled out.
 
Another common source of memory leaks is caches.
 
A third common source of memory leaks is listeners and other callbacks.
 
 
##Item 7:(不是非常明白，待读)
###Avoid finalizers
Finalizers are unpredictable, often dangerous, and generally unnecessary.
 
**YOU should avoid finalizers**
 
There is a severe performance penalty for using finalizers. It is about 430 times slower to create and destroy objects with finalizers.
 
Just provide an explicit termination method, and require clients of the class to invoke this method one each instance when it is no longer needed.
 
Typical examples of explicit termination methods are the close methods on InputStream, OutputStream, and java.sql.Connection.
 
Explicit termination methods are typically used in combination with the try-finally construct to ensure termination.
 
Things that finalizers are good for:
    - One is to act as a "safety net" in case the owner of an object forgets to call its explicit termination method. While there's no guarantee that the finalizer will be invoked promptly, it may be better to free the resource later than never. But the finalier should log a warning if it finds that the resource has not been terminated.
    - A second legitimate use of finalizers concerns objects with native peers. A native peer is a native object to which a normal object delegates via native methods.