# Classes

D provides support for classes and interfaces like in Java.

All `class` types implicitly inherit from [`Object`](https://dlang.org/phobos/object.html).

    class Foo { } // inherits from Object
    class Bar : Foo { } // Bar is a Foo too

Classes in D are generally instantiated on the heap using `new`:

    auto bar = new Bar;

Class objects are always reference types and unlike structs, they aren't
copied by value.

    Bar bar = foo; // bar points to foo

The garbage collector will make sure the memory is freed
when no references to an object exist anymore.

### Inheritance

Like [structs](basics/structs), a class can have member functions (methods).
By default, they are virtual and can be overridden by a child class.

If a member function of a base class is overridden, the keyword
`override` must be used. This prevents unintentional
overriding of functions.

    class Foo {
        void functionFromFoo() {}
    }
    class Bar : Foo {
        override void functionFromFoo() {}
    }

In D, each class may only directly inherit from one other class.
This prevents the diamond inheritance problem, and isn't a limitation
in practice because D (like Java), supports interfaces.

### Final and abstract member functions

- A function can be marked `final` in a base class to disallow overriding
it.
- A function can be declared as `abstract` to force a derived class to override
it.
- A whole class can be declared as `abstract` to make sure
that it isn't instantiated.

### Constructors

Similar to structs, a class can define one or more constructors. The compiler calls
a matching constructor when creating an instance of a class `C`, based on arguments
given e.g. `new C(args)`.

- A constructor is declared as `this(Parameters) { /* setup code */ }`.
- A constructor body can call another constructor with `this(args)`.
- `super(args)` can be used to explicitly call a base constructor.

### Runtime type information

    class Foo {}
    class Bar : Foo {}

    void main()
    {
        Foo foo = new Bar;
        assert(typeid(foo) == typeid(Bar));

        // dynamic cast
        Bar bar = cast(Bar) foo;
        assert(bar);
    }

- `typeid(foo)` is a runtime expression which gives the *TypeInfo* object for *Bar*.
- `cast(Bar) foo` will check `foo` is an instance of `Bar`, casting `foo` if it is,
or yielding `null` otherwise.

### Checking for identity

For class objects, the `==` and `!=` operators compare the _contents_ of the objects.
Therefore, comparing against `null` is invalid, as `null` has no contents.
The `is` compares for identity. To compare for nonidentity, use `e1 !is e2`.

```d
MyClass c;
if (c == null)  // error
    ...
if (c is null)  // ok
    ...
```

For `struct` objects all bits are compared,
for other operand types, identity is the same as equality.

### In-depth

- [Classes in _Programming in D_](http://ddili.org/ders/d.en/class.html)
- [Inheritance in _Programming in D_](http://ddili.org/ders/d.en/inheritance.html)
- [Object class in _Programming in D_](http://ddili.org/ders/d.en/object.html)
- [Classes in D spec](https://dlang.org/spec/class.html)

## {SourceCode}

```d
import std.stdio : writeln;

/*
Fancy type which can be used for
anything...
*/
class Any {
    // protected is just seen by inheriting
    // classes
    protected string type;

    // constructor
    this(string type) {
        this.type = type;
    }

    // public is implicit by the way
    final string getType() {
        return type;
    }

    // This needs to be implemented!
    abstract string convertToString();
}

class Integer : Any {
    // just seen by Integer
    private {
        int number;
    }

    this(int number) {
        // call base class constructor
        super("integer");
        this.number = number;
    }

    // This is implicit. And another way
    // to specify the protection level
    public:

    override string convertToString() {
        import std.conv : to;
        // The swiss army knife of conversion.
        return to!string(number);
    }
}

class Float : Any {
    private float number;

    this(float number) {
        super("float");
        this.number = number;
    }

    override string convertToString() {
        import std.string : format;
        // We want to control precision
        return format("%.1f", number);
    }
}

void main()
{
    Any[] anys = [
        new Integer(10),
        new Float(3.1415f)
    ];

    foreach (any; anys) {
        writeln("any's type = ", any.getType());
        writeln("Content = ",
            any.convertToString());
    }
}
```
