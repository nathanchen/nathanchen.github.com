---
layout: post
title: "Effective JAVA(1)"
category: 
tags: ["读书", "Java"]
---
{% include JB/setup %}

##Item 1: 
###Consider static factory methods instead of constructors

**The normal way for a class to allow a client to obtain an instance of itself is to provide a public constructor**. A class can provide a public static factory method, which is simply a static method that returns an instance of the class.

A class can provide its clients with static factory methods instead of, or in addition to, constructors.

	One advantage of static factory methods is that, unlike constructors, they have names.

A static factory with a well-chosen name is easier to use and the resulting client code easier to read. **For example, the constructor BigInteger(int, int, Randon)m which returns a BigInteger that is probably prime, would have been better expressed as a static factory method named BigInteger.probablePrime**.

##### In case where a class seems to require multiple constructors with the same signature, replace the constructors with static factory methods and carefully chosen names to highlight their differences.

	A second advantage of static factory methods is that, unlike constructors, they are not required to create a new object each time they're invoked.

The Boolean.valueOf(boolean) method illustrates this technique: it never creates an object. Instance control allows a class to guarantee that it is a singleton or noninstantiable. It allows an immutable class to make the guarantee that no two equal instances exist: a.equals(b) if and only if a==b

	A third advantage of static factory methods is that, unlike constructors, they can return an object of any subtype of their return type.

	A fourth advantage of static factory methods is that they reduce the verbosity of creating parameterized type instances.

	Map<String, List<String>> m = new HashMap<String, List<String>>();
	
	OR

	public static <K, V> HashMap<K, V> newInstance(){
		return new HashMap<K, V>();
	} 

	Map<String, List<String>> m = HashMap.newInstance();

##### The main disadvantage of providing only static factory methods is that classes without public or protected constructors cannot be subclassed.

	A second disadvantage of static factory methods is that they are not readily distinguishable from other static methods (names of static methods for different classes could be identical).



##Item 2: 
###Consider a builder when faced with many constructor parameters

Static factories and constructors share a limitation: they do not scale well to large numbers of optional parameters.

Traditionally, programmers have used the telescoping constructor pattern, in which you provide a constructor with only the required parameters, another with a single optional parameter, a third with two optional parameters, and so on.

	NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);

The telecoping constructor pattern works, but it is hard to write client code when there are many parameters, and harder still to read it.

A second alternative when you are faced with many constructor parameters is the JavaBeans pattern, in which you call a parameterless constructor to create the object and then call setter methods to set each required parameter and each optional parameter of interest.

The JavaBeans pattern has serious disadvantages of its own. Because construction is split across multiple calls, a JavaBean may be in an inconsistent state partway through its construction.

##### A third alternative is BUILDER Pattern.

Instead of making the desired object directly, the client calls a constructor (or static factory) with all of the required parameters and gets a builder object. Then the client calls setter-like methods on the builder object to set each optional parameter of interest. Finally, the client calls a parameterless build method to generate the object, which is immutable.

	public class NutritionFacts 
	{
	    private final int servingSize;
	    private final int servings;
	    private final int calories;
	    private final int fat;
	    private final int sodium;
	    private final int carbohydrate;
	    
	    public static class Builder 
	    {
	        // Required parameters
	        private final int servingSize;
	        private final int servings;
	        // Optional parameters - initialized to default values
	        private int calories      = 0;
	        private int fat           = 0;
	        private int carbohydrate  = 0;
	        private int sodium        = 0;
	        
	        public Builder(int servingSize, int servings) 
	        {
	            this.servingSize = servingSize;
	            this.servings    = servings;
			}
			
	        public Builder calories(int val)
	        { calories = val;      return this; }
	        
	        public Builder fat(int val)
	        { fat = val;           return this; }
	        
	        public Builder carbohydrate(int val)
	        { carbohydrate = val;  return this; }
	        
	        public Builder sodium(int val)
	        { sodium = val;        return this; }
	        
	        public NutritionFacts build() 
	        {
	            return new NutritionFacts(this);
			} 
		}

	    private NutritionFacts(Builder builder) 
	    {
	        servingSize  = builder.servingSize;
	        servings     = builder.servings;
	        calories     = builder.calories;
	        fat          = builder.fat;
	        sodium       = builder.sodium;
	        carbohydrate = builder.carbohydrate;
		} 
	}

##### NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).calories(100).sodium(35).carbohydrate(27).build();

###The builder pattern is a good choice when designing classes whose constructor or static factories would have more than a handful of parameters.
