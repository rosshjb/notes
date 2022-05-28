# Lexical

- identifier prefixed by `@` is reserved.
- decl order is insignificant. 

# Types

## reference types vs. value types

- val type var contains data directly. (impl detail : stack-allocated).
    - simple : `sbyte`, `short`, `int`, `long`, `byte`, `ushort`, `uint`, `ulong`, `char`, `float`, `double`, `decimal`, `bool`
    - enum : distinct type with named constants (underlying integral types)
    - struct
    - tuple : `(T1, T2, ...)`
    - and nullable value types
- ref type var contains reference to obj. (impl detail : heap-allocated).
    - class : `object`, `string`, ...
    - interface
    - array : `int[]`, `int[,]`, `int[][]`
    - delegate : `delegate int f(int)`
- user-definable : (record) class, (record) struct, interface, enum, delegate, tuple.

## root type

- All types, including primitive types, inherit from a single root `object` type.
- ref types can be treated as `object`s.
- val types can be treated as `objects` through (un)boxing; value-copied from/to object.

## class vs struct

- class is ref type; struct is value type.
- structs don't allow inheritance (i.e., sealed) => all implicitly inherits `object`.
- both can impl interfaces.

## delegate

- i.e., function types
- reference to methods

## generics

- support class, struct, interface, delegate

## array

- `int[]` : single dimen
- `int[,]` : two dimen
- `int[][]` : single of single (jagged)

## nullable vs. non-nullable

```cs
int? optionalInt = default;     // nullable default is null
optionalInt = 5;
string? optionalText = default; // nullable default is null
optionalText = "Hello World.";
```

- var of any type can be declared as non-nullable(`T`) or nullable(`T?`); nullable can hold additional value `null`
- nullable val types represented by `System.Nullable<T>`
- (non-)nullable ref types represented by underlying ref type.
- by diff (non-)nullable, metadata is diff by compilers/libraries; warning when nullable dereferenced without checking null, when non-nullable assigned null, etc.

# .NET

- programs run on .NET
- consists of CLR(common language runtime) + class libraries
- CLI(common language infrastructure) standard implementation by MS 
    -  basis for creating execution and development environments in which languages and libraries work together seamlessly.

## assembly

- C# executable code compiled into IL(intermediate language), conforming to CLI spec.
- IL + resources packaged into assembly `.dll`(lib) or `.exe`(app)
- there is manifest in assembly
    + symbolic info about assembly's type, version and culture

## program execution

1. CLR loads assembly
2. Before execution, IR code is JIT-compiled by CLR, to native inst.
3. CLR provides services related to automatic garbage collection, exception handling, and resource management.
4. code is executed by CLR => managed code

## IL

- IL conforms to CLS(common type specification) -> language interop.

# class

```cs
/* class header + class body : */
/* [attributes and modifiers]
 * class <name>
 * [: <base class/interface>[, <base class/interface>]]
 * { <body> }
 */
public class Point {
    public int X { get; }
    public int Y { get; }
    
    public Point(int x, int y) => (X, Y) = (x, y);
}
```

```cs
var p = new Point(0, 0);
```

- classes inherit `object` by default.

## inheritance

```cs
                    // base class
public class Point3D : Point {
    public int Z { get; set; }
    
                                       // call base's ctor 
    public Point3D(int x, int y, int z) : base(x, y) {
        Z = z;
    }
}
```

```cs
Point a = new(10, 20);
Point b = new Point3D(10, 20, 30);  // polymorph
```

- A class doesn't inherit the instance and static constructors, and the finalizer 

## members

- static vs. instance
- constant, field, method, property, indexer, event, operator, constructor, finalizer, (nested) type
- Specifically, *function members*(members that contain executable code) are constructors, properties, indexers, events, operators, and finalizers. 

### field

```cs
public class Color {
    // read-only(run-time constant) static fields
    public static readonly Color Black = new(0, 0, 0);
    public static readonly Color Red = new(255, 0, 0);
    public static readonly Color Blue = new(0, 0, 255);
    
    // instance fields
    public byte R;
    public byte G;
    public byte B;

    public Color(byte r, byte g, byte b) {
        R = r;
        G = g;
        B = b;
    }
}
```

- read-only fields can be assigned only in decl or ctor.

### methods

- signature consists of
    - name
    - number of type params
    - number, modifier, type of params
- method body can be single expression:
    ```cs
    public override string ToString() => "This is an object";
    ```

#### method parameters

- value params : value-copied from argument
    - can be optional(omitted) through default value
- reference params : ref-copied from argument; arg must be initialized. param declared with `ref` modifier.
    ```cs
    static void Main(String[] args) {
        int i = 1, j = 2;
        Console.WriteLine($"{i}, {j}");  // 1, 2
        Swap(ref i, ref j);
        Console.WriteLine($"{i}, {j}");  // 2, 1
    }

    static void Swap(ref int x, ref int y) {
        int temp = x;
        x = y;
        y = temp;
    }
    ```
- output params : ref-copied from argument; arg needn't be initialized. param declared with `out` modifier. output param must be assigned before return.
    ```cs
    static void Main(String[] args) {
        int i;
        Initialize(out i);
        Console.WriteLine($"{i}");  // 10
    }

    static void Initialize(out int result) {
        result = 10;
    }
    ```
- params array params : allows a variable number of arguments to be passed as single-dimensional array param. param declared with `param` modifier. should be only the last param. allows passing arguments as single argument array, or as individual element type arguments(array param auto created/initialized). 
    ```cs
    static void Main(String[] args) {
        PassVariableArguments("<", ">", 1, 2, 3);  // < 1 2 3 >
        PassVariableArguments("[", "]", new int[]{4, 5, 6});  // [ 4 5 6 ]
    }

    static void PassVariableArguments(string start, string end, params int[] args) {
        Console.Write(start + " ");
        foreach (int i in args)
            Console.Write($"{i} ");
        Console.WriteLine(end);
    }
    ```

#### local var

- local vars should be definitely assigned before used.

#### this

- In instance method (and constructor), `this` refers to invoked current object.
    - can refer both instance and static fields
- In static method, `this` can't be referred.
    - cannot refer instance fields.

#### virtual

- declared and implemented as `virtual` in base; derived provide more specific impl.
- allow run-time type is considered, not compile-time type, to determine the called member.
- can be overriden in derived's method with `override`, to provide new impl.

#### abstract

```code
public abstract class Expression {
    public abstract double Evaluate(Dictionary<string, object> vars);
}

public class Constant : Expression {
    double _value;
    
    public Constant(double value) {
        _value = value;
    }
    
    public override double Evaluate(Dictionary<string, object> vars) {
        return _value;
    }
} 
```

- virtual with no impl; declared with `abstract`.
- permiited only in `abstract` classes.
- must be overriden in non-abstract derived class

#### overloading

- same name methods with unique signature => overload resolution => find best match



## accessibility

- `public` : no limit
- `private` : this cls
- `protected` : this cls, derived cls
- `internal` : current assembly
- `protected internal` : this cls, derived cls, other cls within same assembly
- `private protected` : this cls, derived cls within same assembly

# struct

```cs
public struct Point {
    public double X { get; }
    public double Y { get; }
    
    public Point(double x, double y) => (X, Y) = (x, y);
}
```

- structs inherit `System.ValueType` implicitly.

# interfaces

```cs
interface IControl {
    void Paint();
}
```

## inheritance

```cs
interface ITextBox : IControl {
    void SetText(string text);
}

interface IListBox : IControl {
    void SetItems(string[] items);
}

// multiple inheritance
interface IComboBox : ITextBox, IListBox { }
```

- interface can contain methods, properties, events, indexers.
- members typically don't provide impls.

## implementation

```cs
interface IDataBound {
    void Bind(Binder b);
}

public class EditBox : IControl, IDataBound {
    public void Paint() { }
    public void Bind(Binder b) { }
}
```

# enums

```cs
public enum SomeRootVegetable {
    HorseRadish,
    Radish,
    Turnip
}
```

```cs
var turnip = SomeRootVegetable.Turnip;
```

## enum, treated as bitfield

```cs
[Flags]                                     // thanks to
public enum Seasons {
    None = 0,
    Summer = 1,
    Autumn = 2,
    Winter = 4,
    Spring = 8,
    All = Summer | Autumn | Winter | Spring // here
}
```

```cs
var spring = Seasons.Spring;
var startingOnEquinox = Seasons.Spring | Seasons.Autumn;  // here
var theYear = Seasons.All;
```

# tuple

```cs
(double Sum, int Count) t2 = (4.5, 3);

Console.WriteLine($"Sum of {t2.Count} elements is {t2.Sum}.");
```

# generic

```cs
public class Pair<TFirst, TSecond> {
    public TFirst First { get; }
    public TSecond Second { get; }
    
    public Pair(TFirst first, TSecond second) => 
        (First, Second) = (first, second);
}
```

when generic type is provided type arguments, it is called constructed type: 

```cs
var pair = new Pair<int, string>(1, "two");
int i = pair.First;
string s = pair.Second;
```

for generic method's type parameters, type argument can often be inferred from the call. 