# SED505 - Assignment 3 - Investigation of Design Patterns

The Lesson Plan:

[1. Introduction](https://github.com/Wadood3/SED505-ASG3/blob/main/README.md#introduction)

[2. Videos](https://github.com/Wadood3/SED505-ASG3/blob/main/README.md#videos)

[3. Rationale](https://github.com/Wadood3/SED505-ASG3/blob/main/README.md#rationale)

[4. UML Diagram](https://github.com/Wadood3/SED505-ASG3/blob/main/README.md#uml-diagram)

[5. Example Code (Simple)](https://github.com/Wadood3/SED505-ASG3/blob/main/README.md#example-code-simple-c)

[6. Common Usage](https://github.com/Wadood3/SED505-ASG3/blob/main/README.md#common-usage)

[7. Example Code (Complex)](https://github.com/Wadood3/SED505-ASG3/blob/main/README.md#example-code-complex-c)

The Flyweight Design Pattern

# Introduction

Flyweight is a structural design pattern that lets you fit more objects into the available amount of RAM by sharing common parts of state between multiple objects instead of keeping all of the data in each object. It separates the object’s data into two types:

1. Intrinsic state: Shared, immutable data common across many objects.
2. Extrinsic state: Unique, context-dependent data supplied at runtime.

This pattern is especially useful when an application needs to create a large number of similar objects that would otherwise consume significant memory.


# Videos: 
1. https://youtu.be/6cj-VxC8CxI?si=k5WB6DWubhq7BQi0 (Java Example)
2. https://youtu.be/8cL9KbHS5kE?si=z_bf9CfDU_3atlSB (Explanation through Scratch)

# Rationale: 
In applications such as games, or GUIs, thousands of objects may share similar data. Creating a separate object for each instance wastes memory. The Flyweight pattern helps by sharing objects with common states, reducing the memory usage and improving efficiency.

Use Case: Instead of creating 1,000 separate objects with the same color, we share a single 'Color_Flyweight' object representing that color.

# UML Diagram:

<img width="800" height="401" alt="image" src="https://github.com/user-attachments/assets/c124e3fd-9cb1-41bd-bf76-a7780d158d66" />

1. Flyweight (Interface)

Role:
Defines the common interface for all flyweight objects.

Responsibilities: Declares the operation(extrinsicData : Object) method, which accepts external (context-specific) data that varies between objects. Ensures that all concrete flyweights can be used interchangeably by the client.

2. ConcreteFlyweight

Role: Implements the Flyweight interface and represents the shared object that can be reused.

Attributes:
intrinsicState : Object — internal, shared state that doesn’t change across instances (e.g., color, texture, font).

Methods:
operation(extrinsicData : Object) — performs the operation using both intrinsic and provided extrinsic data.

3. FlyweightFactory

Role: Responsible for creating and managing flyweight objects. It ensures that flyweights are reused instead of being created repeatedly.

Methods:
get(key : Object) : Flyweight — Returns an existing flyweight if one exists for the given key; otherwise, creates a new one and stores it.

4. Client

Role:
The Client interacts with the system using the Flyweight objects.

Responsibilities:
Requests flyweights from the FlyweightFactory.

# Example Code (Simple) (C#)



```
// Flyweight interface
public interface IShape
{
    void Draw(string color);
}

// Concrete Flyweight
public class Circle : IShape
{
    public void Draw(string color)
    {
        Console.WriteLine($"Drawing a {color} circle.");
    }
}

// Flyweight Factory
public class ShapeFactory
{
    private Dictionary<string, IShape> shapes = new Dictionary<string, IShape>();

    public IShape GetCircle()
    {
        if (!shapes.ContainsKey("circle"))
        {
            shapes["circle"] = new Circle();
        }
        return shapes["circle"];
    }
}

// Client code
class Program
{
    static void Main()
    {
        ShapeFactory factory = new ShapeFactory();

        IShape circle1 = factory.GetCircle();
        circle1.Draw("Red");

        IShape circle2 = factory.GetCircle();
        circle2.Draw("Blue");

        // Both circle1 and circle2 refer to the same object
        Console.WriteLine($"Is Red and Blue same object? {object.ReferenceEquals(circle1, circle2)}"); // True
    }
}
```

# Common Usage
1. Game Development: Sharing textures, sprites, or particles in a game to reduce memory load.
2. Text Editors: Reusing character glyphs or font styles instead of creating new objects for every letter.
3. Graphics Rendering: Managing shared visual elements like icons or shapes.
4. Cache Systems: Reusing data objects that represent immutable shared data.

# Example code (Complex) (C#)

Problem: You are building a forest rendering system for a game that displays thousands of trees.

Each tree has: 
1. Species name
2. color
3. texture
4. position (x and y)

Issue: Creating an individual object for every tree (with all properties) would waste memory.

Solution: By introducing a shared TreeType flyweight, only one object per tree species is created.
Each Tree stores only its position, and references a TreeType from a TreeFactory that manages reuse.

```

// Flyweight: Shared state for tree types
class TreeType // Intrinsic state: shared across all instances
{
    private readonly string name;    // Species name
    private readonly string color;   // Color
    private readonly string texture; // Texture

    // Constructor
    public TreeType(string name, string color, string texture)
    {
        this.name = name;
        this.color = color;
        this.texture = texture;
    }

    // Draw method uses values provided by extrinsic state object
    public void Draw(int x, int y)
    {
        Console.WriteLine($"Drawing {name} tree with color {color} and texture {texture} at ({x}, {y})");
    }
}

// Flyweight Factory: Manages shared TreeType instances
class TreeFactory
{
    // Dictionary to store shared TreeType objects connected to unique keys
    private static readonly Dictionary<string, TreeType> treeTypes = new Dictionary<string, TreeType>();

    // Returns existing TreeType if available, else creates new one
    public static TreeType GetTreeType(string name, string color, string texture)
    {
        string key = $"{name}-{color}-{texture}"; // Unique key for each type

        if (!treeTypes.ContainsKey(key))
        {
            // Create and cache new TreeType if not found
            treeTypes[key] = new TreeType(name, color, texture);
            Console.WriteLine($"Creating new TreeType: {key}");
        }

        return treeTypes[key]; // Return shared instance
    }
}

// Context: Unique tree with position and shared type
class Tree // Extrinsic state: unique per instance
{
    private readonly int x;         // Unique x position
    private readonly int y;         // Unique y position
    private readonly TreeType type; // Shared tree type

    // Constructor
    public Tree(int x, int y, TreeType type)
    {
        this.x = x;
        this.y = y;
        this.type = type;
    }

    // Uses the Intrinsic state to draw itself
    public void Draw()
    {
        type.Draw(x, y);
    }
}


// Forest manages all Tree objects
// Uses TreeFactory to share TreeType objects
class Forest
{
    private readonly List<Tree> tree_forest = new List<Tree>(); // List of all trees

    // Plants a tree at (x, y) with given type
    public void PlantTree(int x, int y, string name, string color, string texture)
    {
        // Get shared TreeType from factory
        TreeType type = TreeFactory.GetTreeType(name, color, texture);
        
        // Create Tree with unique position and shared type then add to forest
        tree_forest.Add(new Tree(x, y, type));
    }

    // Draws all trees in the forest
    public void Draw()
    {
        foreach (var tree in tree_forest)
        {
            tree.Draw();
        }
    }
}

// Demo: Shows Flyweight pattern in action
class FlyweightForestDemo
{
    static void Main()
    {
        Forest forest = new Forest();

        // Plant trees with shared types but unique positions
        forest.PlantTree(10, 20, "Oak", "Green", "Rough");      // New TreeType created
        forest.PlantTree(30, 40, "Oak", "Green", "Rough");      // Reuses existing TreeType
        forest.PlantTree(50, 60, "Pine", "Dark Green", "Smooth"); // New TreeType created

        // Draw all trees
        forest.Draw();
    }
}
```

