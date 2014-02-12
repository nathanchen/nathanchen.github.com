---
layout: post
title: "ThreadLocal 那点事儿"
category: Reading Notes
tags: ["读文章", "Java"]
---
{% include JB/setup %}

### ThreadLocal 那点事儿

	public class SequenceB implements Sequence {
	
	    private static ThreadLocal<Integer> numberContainer = new ThreadLocal<Integer>() {
	        @Override
	        protected Integer initialValue() {
	            return 0;
	        }
	    };
	
	    public int getNumber() {
	        numberContainer.set(numberContainer.get() + 1);
	        return numberContainer.get();
	    }
	
	    public static void main(String[] args) {
	        Sequence sequence = new SequenceB();
	
	        ClientThread thread1 = new ClientThread(sequence);
	        ClientThread thread2 = new ClientThread(sequence);
	        ClientThread thread3 = new ClientThread(sequence);
	
	        thread1.start();
	        thread2.start();
	        thread3.start();
	    }
	}
	
打印结果：

	Thread-0 => 1
	Thread-0 => 2
	Thread-0 => 3
	Thread-2 => 1
	Thread-2 => 2
	Thread-2 => 3
	Thread-1 => 1
	Thread-1 => 2
	Thread-1 => 3

> 用ThreadLocal<T>保证了一个Thread用一个T，Thread之间不会共享这个T

####  ThreadLocal 里面不就是封装了一个 Map

	public class MyThreadLocal<T> {
	
	    private Map<Thread, T> container = Collections.synchronizedMap(new HashMap<Thread, T>());
	
	    public void set(T value) {
	        container.put(Thread.currentThread(), value);
	    }
	
	    public T get() {
	        Thread thread = Thread.currentThread();
	        T value = container.get(thread);
	        if (value == null && !container.containsKey(thread)) {
	            value = initialValue();
	            container.put(thread, value);
	        }
	        return value;
	    }
	
	    public void remove() {
	        container.remove(Thread.currentThread());
	    }
	
	    protected T initialValue() {
	        return null;
	    }
	}





### Reference：

http://my.oschina.net/huangyong/blog/159489

