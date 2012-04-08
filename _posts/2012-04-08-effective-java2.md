---
layout: post
title: "Effective JAVA(2)"
category: 
tags: []
---
{% include JB/setup %}

##Item 3:
###Enforce the singleton property with a private constructor or an enum type

A singleton is simple a class that is instantiated exactly once.

	Approach 1:
		public static final Elvis INSTANCE = new Elvis();

	Approach 2:
		private static final Elvis INSTANCE = new Elvis();
		public static Elvis getInstance() { return INSTANCE; }

The main advantage of the public field approach is that the declarations make it clear that the class is a singleton.
One advantage of the factory-method approach is that it gives you the flexibility to change your mind about whether the class should be a singleton without changing its API. The factory method returns the sole instance but could easily be modified to return a unique instance for each thread that invokes it. A second advantage, concerning generic types.

To make a singleton class that is implemented using either of the previous approaches serializable. Otherwise, each time a serialized instance is deserialized, a new instance will be created. To prevent this, add this readResolve method to the Elvis class:

	// readResolve method to preserve singleton property
   private Object readResolve() {
        // Return the one true Elvis and let the garbage collector
        // take care of the Elvis impersonator.
       return INSTANCE;
   }

There is a third approach to implementing singletons. Simply make an enum type with one element:

    // Enum singleton - the preferred approach
   public enum Elvis {
       INSTANCE;
       public void leaveTheBuilding() { ... }
   }
**A single-element enum type is the best way to implement a singleton.**

##Item 4:
###Enforce noninstantiability with a private constructor

Attempting to enforce noninstantiability by making a class abstract does not work.

A class can be made noninstantiable by including a private constructor:
	
	// Noninstantiable utility class
   public class UtilityClass 
   {
       // Suppress default constructor for noninstantiability
       private UtilityClass() 
       {
           throw new AssertionError();
		}
       ...  // Remainder omitted
   }

It guarantees that the class will never be instantiated under any circumstances.


##Item 5:
###Avoid creating unnecessary objects

You can often avoid creating unnecessary objects by using static factory methods. For example, the static factory method Boolean.valueOf(String) is almost always preferable to the constructor Boolean(String)

class Person {
       private final Date birthDate;
       // Other fields, methods, and constructor omitted
       /**
        * The starting and ending dates of the baby boom.
        */
       private static final Date BOOM_START;
       private static final Date BOOM_END;

       **static {
           Calendar gmtCal =
               Calendar.getInstance(TimeZone.getTimeZone("GMT"));
           gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
           BOOM_START = gmtCal.getTime();
           gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
           BOOM_END = gmtCal.getTime();
		}**

       public boolean isBabyBoomer() {
           return birthDate.compareTo(BOOM_START) >= 0 &&
                  birthDate.compareTo(BOOM_END)   <  0;
		} 
	}

The Person class creates Calendar, TimeZone, and Date instances only once, when it is initialized, instead of creating them every time isBabyBoomer is invoked.

**Prefer primitives to boxed primitives, and watch out for unintentional autoboxing.**


