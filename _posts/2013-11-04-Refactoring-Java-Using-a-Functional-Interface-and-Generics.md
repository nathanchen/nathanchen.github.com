---
layout: post
title: "Refactoring Java Using a Functional Interface and Generics"
category: Reading Notes 
tags: ["读文章", "Java", "代码重构"]
---
{% include JB/setup %}

### Original Code 

	public V get(final K key)
	{
	  Session s;
	  try {
	      s = oGrid.getSession();
	      ObjectMap map = s.getMap(cacheName);
	      return (V) map.get(key);
	  }
	  catch (ObjectGridException oge)
	  {
	      throw new RuntimeException("Error performing cache operation", oge);
	  }
	  finally
	  {
	      if (s != null)
	          s.close();         
	  }    
	  return null;
	}      
	
	public void put(final K key, final V value)
	{
	  Session s;
	  try {
	      s = oGrid.getSession();
	      ObjectMap map = s.getMap(cacheName);
	      map.upsert(key, value);
	  }
	  catch (ObjectGridException oge)
	  {
	      throw new RuntimeException("Error performing cache operation", oge);
	  }
	  finally
	  {
	      if (s != null)
	          s.close();             
	  }            
	}
	
	public Map<K, V> getAll(Set<? extends K> keys)
	{
	  final List<V> valueList = new ArrayList<V>();
	  final List<K> keyList = new ArrayList<K>();
	  keyList.addAll(keys);
	
	  Session s;
	  try {
	      s = oGrid.getSession();
	      ObjectMap map = s.getMap(cacheName);
	      valueList = map.getAll(keyList);
	  }
	  catch (ObjectGridException oge)
	  {
	      throw new RuntimeException("Error performing cache operation", oge);
	  }
	  finally
	  {
	      if (s != null)
	          s.close();             
	  }
	                              
	  Map<K, V> map = new HashMap<K, V>();
	  for(int i = 0; i < keyList.size(); i++) {
	      map.put(keyList.get(i), valueList.get(i));
	  }
	  return map;
	}

### The problem：this block of code is duplicate

	Session s;
	try {
	  s = oGrid.getSession();
	  ObjectMap map = s.getMap(cacheName);
	  // Some small bit of business logic goes here
	}
	catch (ObjectGridException oge)
	{
	  throw new RuntimeException("Error performing cache operation", oge);
	}
	finally
	{
	  if (s != null)
	      s.close();             
	}

### The refactored code:

	interface Executable<T> {
	  public T execute(ObjectMap map) throws ObjectGridException;
	}
	
	private <T> T executeWithMap(Executable<T> ex)
	{
	  Session;
	  try {
	      s = oGrid.getSession();
	      ObjectMap map = s.getMap(cacheName);
	      // Execute our business logic
	      return ex.execute(map);
	  }
	  catch (ObjectGridException oge)
	  {
	      throw new RuntimeException("Error performing cache operation", oge);
	  }
	  finally
	  {
	      if (s != null)
	          s.close();             
	  }
	}
	
	public V get(final K key)
	{
	  return executeWithMap(new Executable<V>() {
	      public V execute(ObjectMap map) throws ObjectGridException
	      {
	          return (V) map.get(key);
	      }
	  });              
	}      
	
	public void put(final K key, final V value)
	{
	  executeWithMap(new Executable<Void>() {
	      public Void execute(ObjectMap map) throws ObjectGridException
	      {
	          map.upsert(key, value);
	          return null;
	      }
	  });              
	}
	
### Reference:

http://www.michaelbrameld.com/blog/2013/11/02/refacoring-java-generic-functional-interface/