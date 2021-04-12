---
title: Required Data Annotations where data is NULL
date: "2021-04-12"
description: Understanding the scope of Required in newer versions of EF Core
---
## Data Annotations

Data annotations are great for quickly and easily changing data display formats or specifying required properties on a model. However, you should be careful when applying annotations to an existing model where it may contradict the state of your data.

```cs
[DisplayName("First Name")] 
[Required] 
public string firstName { get; set; }
```
What the example above really means is this string property is ***never null***. If you apply this annotation ***after*** your data is collected then <code>Entity Framework Core</code> will throw a Null Reference Exception when attempting to read this property if it is empty.

EF Core used to be more tolerant of this, alas in 3.1.1 it is much stricter.