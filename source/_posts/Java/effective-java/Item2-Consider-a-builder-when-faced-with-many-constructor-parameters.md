---
title: Effective Java-2. Consider a builder when faced with many constructor parameters
categories:
  - [Java, Effective Java]
---

# Item 2: Consider a builder when faced with many constructor parameters

## Background

Consider the case of a class representing the Nutrition Facts label that appears on packaged foods. These labels have a few required fields—serving size, servings per container, and calories per serving— and more than twenty optional fields—total fat, saturated fat, trans fat, cholesterol, sodium, and so on. Most products have nonzero values for only a few of these optional fields.

## Solutions

### 1. Telescoping Constructor

Provide a constructor with only the required parameters, another with a single optional parameter, a third with two optional parameters, and so on, culminating in a constructor with all the optional parameters.

```java
// Telescoping constructor pattern - does not scale well!
public class NutritionFacts {
  
  private final int servingSize;  // required
  private final int servings;	// required
  private final int calories; // optional
  private final int fat; // optional
  private final int sodium; // optional
  private final int carbohydrate; // optional
  
  public NutritionFacts(int servingSize, int servings) {
      this(servingSize, servings, 0);
  }
  public NutritionFacts(int servingSize, int servings,
          int calories) {
      this(servingSize, servings, calories, 0);
  }
  public NutritionFacts(int servingSize, int servings,
          int calories, int fat) {
      this(servingSize, servings, calories, fat, 0);
  }
  public NutritionFacts(int servingSize, int servings,
          int calories, int fat, int sodium) {
      this(servingSize, servings, calories, fat, sodium, 0);
  }
  
  public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
    this.servingSize = servingSize;
    this.servings = servings;
    this.calories = calories;
    this.fat = fat;
    this.sodium = sodium;
    this.carbohydrate = carbohydrate;
  }
}

// create an instance
NutritionFacts cocaCola =
       new NutritionFacts(240, 8, 100, 0, 35, 27);
```

**The telescoping constructor pattern works, but it is hard to write client code when there are many parameters, and harder still to read it.**

### 2. JavaBeans

Call a parameterless constructor to create the object and then call setter methods to set each required parameter and each optional parameter of interest.

```java
// JavaBeans Pattern - allows inconsistency, mandates mutability
public class NutritionFacts {
  // Parameters initialized to default values (if any)
  private int servingSize = -1; //Required; no default value
  private int servings = -1; //Required; no default value
  private int calories = 0;
  private int fat = 0;
  private int sodium = 0;
  private int carbohydrate = 0;
  
  public NutritionFacts() { }
  
  // Setters
  public void setServingSize(int val) { servingSize = val; }
  public void setServings(int val) { servings = val; }
  public void setCalories(int val) { calories = val; }
  public void setFat(int val) {fat = val;}
  public void setSodium(int val) { sodium = val; }
  public void setCarbohydrate(int val) { carbohydrate = val; }

}

// a bit wordy to create an instance
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

Disadvantages:

- a JavaBean may be in an inconsistent state partway through its construction.
- the JavaBeans pattern precludes the possibility of making a class immutable.

### 3. Builder

Instead of making the desired object directly, the client calls a constructor (or static factory) with all of the required parameters and gets a *builder object*. Then the client calls setter-like methods on the builder object to set each optional parameter of interest. Finally, the client calls a parameterless build method to generate the object, which is typically immutable.

```java
// Builder Pattern
public class NutritionFacts {
  private final int servingSize;
  private final int servings;
  private final int calories;
  private final int fat;
  private final int sodium;
  private final int carbohydrate;
  public static class Builder {
    // Required parameters
    private final int servingSize;
    private final int servings;
    // Optional parameters - initialized to default values
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;
    public Builder(int servingSize, int servings) {
      this.servingSize = servingSize;
      this.servings    = servings;
    }
    public Builder calories(int val)
        { calories = val;      return this; }
    public Builder fat(int val)
        { fat = val;           return this; }
    public Builder sodium(int val)
        { sodium = val;        return this; }
    public Builder carbohydrate(int val)
        { carbohydrate = val;  return this; }
    public NutritionFacts build() {
        return new NutritionFacts(this);
    }
  }
  
  private NutritionFacts(Builder builder) {
    servingSize = builder.servingSize;
    servings = builder.servings;
    calories = builder.calories;
    fat = builder.fat;
    sodium = builder.sodium;
    carbohydrate = builder.carbohydrate;
  }
}

// using a fluent API (chained) to create an instance
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
           .calories(100).sodium(35).carbohydrate(27).build();
```

Disadvantages:

- In order to create an object, you must **first create its builder**. While the cost of creating this builder is unlikely to be noticeable in practice, it could be a **problem in performance-critical situations**.
- Also, the Builder pattern is more verbose than the telescoping constructor pattern, so it should be used only if there are enough parameters to make it worthwhile, say four or more. 

## Summary

The Builder pattern is a good choice when designing classes whose constructors or static factories would have **more than a handful of parameters**.
