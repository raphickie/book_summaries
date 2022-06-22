# 2.4.4+

- Wow, iterators are IDisposable.
- Wow, foreach uses Using inside. Also it creates a state machine: Current, Finally, Dispose and other fun stuff. It saves state of the iterator, it has a special finalizer etc.

- Weird, but Partial classes can have partial classes inside them. I never needed to use that, but it's said that it is needed in the code generators

- Static class cannot be used as a generic parameter (never needed that though)
- :: is just an alias to a namespace
- extern alias is made to use classes of the same namespace, but different assemblies
- #pragma - just for warnings, not errors
- Expression q = (string x, string y) => x + y; - q.toString will return (x,y) => x+y. Cool.

- Nice thing with Dynamic:
  -- create overloads for different parameter types
  -- call method for dynamic type
  -- compiler chooses which method to use

# Dymamics

- ExpandoObject: additional object from Dynamic. When using statically, it has Dictionary inside.
- You cannot use extension methods for dynamic

# 7+

- Foreach can make smart Capture after c# 5: if you do the foreach with actions, it creates a new parameter every time (doesn't work in for loop):

```
List<string> names = new List<string> { "x", "y", "z" };
var actions = new List<Action>();
for (int i = 0; i < names.Count; i++)
{
actions.Add(() => Console.WriteLine(names[i]));
}
foreach (Action action in actions)
{
action();
}
```

This returns x,y,z instead of zzz;

# C# 5:

- CallerMemberName, ...line - nice thing, if you want to return calling class without System.Diagnostics (just put it into the method parameter)

```
static void ShowInfo(
[CallerMemberName] string member = null)
{
```

# С# 6

- Getter-setter: if you don't write special stuff for setter, it is "private readonly"
  Now you can write
  `public List<Friend> Friends {get;set;} = new List<Friends>();`

- It's made easier to use getter-setter: you don't have to set everything in the constructor before using.
- Expression=bodied stuff: "=>" appeared in properties (by the way, in cases of properties it's not lambda: there are no delegates created)
- => also appeared in operators

# Strings.

- Alignment in Console.WriteLine:
  `"Price: {0,9:C}"`
  where 0=index, 9 - minimum width, and where to put (right if positive, left if negative), С = format

- You can get the currency position from localization!
- InvariantCulture - general culture for machine (computer,server etc)
- Using interpolation, parameters like alignment and others are set after the comma
- If you want first to create a string, and format it afterwards, there is FormattableString for that
- Interpolated stuff is created _before_ method call: nice fact
- If you make nameof(alias) - it returns you alias name you created.

# Other stuff

- using static appeared

```using static SomeEnum
var a = SomeEnum.SomeEnumValue // can be replaced with SomeEnumValue (without SomeEnum)
```

- You can create indexer right in object initializer:
  `new Dictionary<string,int>(){["Test"]=1}; `
  It also removes duplicates if set that way!

- You can use extension methods inside initialization:
  (seems like `(int? a).Equals(q) ?? false` is really cleaner that the way I used to write

# C# 7 and more

- Deconstruction of tuples: just like in JS
  `public void Deconstruct(out double x, out double y) => (x, y) = (X, Y);`
- You can put deconstruction in extensions, which is even nicer
- Pattern matching: switch case Rectangle rect: ...
- Patterns:
  -- constant (input is "hello"), (decimalnumber is 3L), (objectnumber is 2)
  -- type
  -- var (mostly useless)

# REF

- Ref local:
- You can also adress struct in an array like a class
  ` ref var element = ref array[i].`
- ref can be put in a return of a method, in parameters - everywhere
- Ref readonly is a thing
- In parameter works as ref readonly. If you write some apis, you _should_ put methods with in, ref and others only in private methods, so you won't get sick figuring out what's going wrong: it's error prone

- `public ref struct MyStruct{}` - is a thing. It is ref-like struct, you cannot even use it in methods.
- You can use this
  `Span<char> = stackalloc char[30]`
  it looks like unsafe but is not. It's made for the sake of speed and optimization (not to copy variables to many times)
- Writing binary-styled variables in a nicer way:
  `var q = 0b101101 // = 101101`
- Underscore separators created here
- Unmanaged type: it's a non-nullable value type that doesn't contain anything reference-like (example: double, int, ...)
- You can add field attribute for a property:

```
[Demo]
private string name;
public string Name
{
get { return name; }
set { name = value; }
}
```

can be replaced by

```
[field: Demo]
public string Name { get; set; }
```

# Book ends here. No my reading of latter releases of C#

- Readonly properties in struct

```
struct MyStruct
{
  public readonly double Distance => Math.Sqrt(X * X + Y * Y);
  }
```

- Default interface methods (just like abstract methods)

```
public static RGBColor FromRainbow(Rainbow colorBand) =>
    colorBand switch
    {
        Rainbow.Red    => new RGBColor(0xFF, 0x00, 0x00),
        Rainbow.Orange => new RGBColor(0xFF, 0x7F, 0x00),
        Rainbow.Yellow => new RGBColor(0xFF, 0xFF, 0x00),
        Rainbow.Green  => new RGBColor(0x00, 0xFF, 0x00),
        Rainbow.Blue   => new RGBColor(0x00, 0x00, 0xFF),
        Rainbow.Indigo => new RGBColor(0x4B, 0x00, 0x82),
        Rainbow.Violet => new RGBColor(0x94, 0x00, 0xD3),
        _              => throw new ArgumentException(message: "invalid enum value", paramName: nameof(colorBand)),
    };
```

- Almost same thing for properties:

```
public static decimal ComputeSalesTax(Address location, decimal salePrice) =>
    location switch
    {
        { State: "WA" } => salePrice * 0.06M,
    };
```

- Same for tuples (but more interesting):

```
var (x, y) when x > 0 && y > 0 => Quadrant.One
```

- static local functions added
- now if you create ref struct, it _should_ implement IDisposable
- IAsyncEnumerable. They are created in a captured context (otherwise TaskAsyncEnumerableExtensions.ConfigureAwait should be added)
- IAsyncDisposable: can be used with await using (has IAsyncEnumerator). Also captured context!
- ^ operator: array[^1] = array[array.Length]
- .. operator: [1..4].
  words[1..4] copies elements 1-3 to another array (no element 4)
- ??= is really cool
- struct now can be unmanaged. Now can use it in span, stackalloc and stuff

# C# 9

- Records. You can create _init;_ modifier for the record property for it to be set only on initialization
  - You can make them mutable, but _shouldn't_
- Positional properties

```
public record Person(string FirstName, string LastName);

public static void Main()
{
    Person person = new("Nancy", "Davolio");
    Console.WriteLine(person);
    // output: Person { FirstName = Nancy, LastName = Davolio }
}
```

Also adds "Deconstruct" with _out_ parameter for every positional property

- ReferenceEquals for records return different for similar records, but Equals works like with Struct
- with:

```
Person person2 = Person with Name='Vasily'
```

- Also good nice ToString is implemented inside record by default
- Also you can derive record _from another record_. Equals then doesn't show that the derived equals parent, even if all the fields/properties are the same
- Init setters are also available for classes
- class Program with namespace can be skipped now
- Nicer pattern matching:

  - more fun with _is_

    ```
    public static bool IsLetterOrSeparator(this char c) =>
        c is (>= 'a' and <= 'z') or (>= 'A' and <= 'Z') or '.' or ',';
    ```

  - nicer null check:
    ```
    if(e is not null)
    ```

- nint and nuint (has different size depending on bitness of machine it runs on)
- new() can be used as a parameter in a method call, constructor
- static modifier to lambda and anonymous
- also some stuff added for code generators

# C# 10

- record structs (readonly record structs)
- you can write your own interpolation now
- global using added (can do this with aliases)
  - can be automatically added to whole assembly with <ImplicitUsings>)
  - can remove specific implicit usings if you write
  ```
  <ItemGroup>
  <Using Remove (add)="System.Threading.Tasks" />
  </ItemGroup>
  ```
- File-scoped namespaces now available (without brackets for whole file)
- Natural types for lambdas:

```
var parse = (string s) => int.Parse(s); // shouldn't use Func<string,int>
```

- Nice thing (works only for patterns):
  ```
  { Prop1.Prop2: pattern }
  ```
  is now equal to
  ```
  { Prop1: { Prop2: pattern } }
  ```
- Attributes now can be added to lambdas
- Can seal ToString in a record
-
