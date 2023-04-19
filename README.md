# AnyStruct
Attempting to Create a Struct Base Class to Solve Boxing

> *Author: snoopyuj*  
> *Created: August 21, 2021 5:00 AM*  
> *Tags: C#*

[[中文版]](#嘗試製作-struct-基底類解決轉型-boxing)

# Attempting to Create a Struct Base Class to Solve Boxing

Writing C# for a long time, I always feel frustrated when boxing occurs during object-to-struct conversions. Recently, I spent some time googling and found this discussion:

> [How to store structs of different types without boxing](https://stackoverflow.com/questions/6163335/how-to-store-structs-of-different-types-without-boxing)

After reading through the discussion, I found Alternative 3 quite interesting. It proposes creating a struct base class and implementing various type conversions to solve the boxing issue. However, the code in the article contained some minor syntax errors, so I made some modifications:

```csharp
using System.Runtime.InteropServices;

/// <summary>
/// For storing any struct to avoid boxing
/// </summary>
[StructLayout(LayoutKind.Explicit)]
public partial struct AnyStruct
{
    public static readonly AnyStruct empty = new AnyStruct();

    [FieldOffset(0)] public int intVal;
    [FieldOffset(0)] public float floatVal;
    [FieldOffset(0)] public byte byteVal;

    public static implicit operator int(AnyStruct _obj) { return _obj.intVal; }
    public static implicit operator float(AnyStruct _obj) { return _obj.floatVal; }
    public static implicit operator byte(AnyStruct _obj) { return _obj.byteVal; }

    public static implicit operator AnyStruct(int _val)
    {
        var r = empty;
        r.intVal = _val;

        return r;
    }

    public static implicit operator AnyStruct(float _val)
    {
        var r = empty;
        r.floatVal = _val;

        return r;
    }

    public static implicit operator AnyStruct(byte _val)
    {
        var r = empty;
        r.byteVal = _val;

        return r;
    }
}
```

Creating a base class for type conversion to solve Boxing is nothing special, and everyone should have thought of doing this. However, what's special is that the author used `[StructLayout(LayoutKind.Explicit)]`. StructLayout allows us to control the layout of a struct in memory, and then cleverly paired it with `[FieldOffset(0)]`, so that all variables start from the beginning. This approach ensures that the size of AnyStruct is only the size of the largest variable among these variables, regardless of how many type conversions we implement in AnyStruct.

For example, the above code implements type conversions for int, float, and byte, so the size of AnyStruct is only 4. If we implement type conversion for long, the size of AnyStruct will increase to 8. This way, we don't have to worry about AnyStruct becoming larger as we implement more type conversions. (It will still get larger, but only to the largest variable in it.)

Now let's take a look at the usage and results of AnyStruct:

```csharp
private static void Main()
{
    Console.WriteLine($"AnyStruct Size = {Marshal.SizeOf(AnyStruct.empty)}\n");
    // AnyStruct Size = 4

    AnyStruct any = AnyStruct.empty;
    Console.WriteLine($"=== Original === \n{PrintAnyStruct(ref any)}\n");
    // === Original ===
    // int     = 0
    // float   = 0
    // byte    = 0

    any = 1234;
    Console.WriteLine($"=== Set Int 1234 === \n{PrintAnyStruct(ref any)}\n");
    // === Set Int 1234 ===
    // int     = 1234
    // float   = 1.729E-42
    // byte    = 210

    any = 4.56f;
    Console.WriteLine($"=== Set Float 4.56f === \n{PrintAnyStruct(ref any)}\n");
    // === Set Float 4.56f ===
    // int     = 1083304837
    // float   = 4.56
    // byte    = 133

    any = (byte)78;
    Console.WriteLine($"=== Set Byte 78 === \n{PrintAnyStruct(ref any)}\n");
    // === Set Byte 78 ===
    // int     = 78
    // float   = 1.1E-43
    // byte    = 78
}

private static string PrintAnyStruct(ref AnyStruct _anyStruct)
{
    return $"int \t= {(int)_anyStruct} \nfloat \t= {(float)_anyStruct} \nbyte \t= {(byte)_anyStruct}";
}
```

1. At first, we used `Marshal.SizeOf()` to confirm the size of AnyStruct. Those who are interested can implement other types to see the changes.
2. Using AnyStruct is quite simple because we have implemented implicit casting. We can simply assign values, but we need to convert them to the correct type when retrieving the value.
3. In the example, I printed the values of each variable inside AnyStruct when different types of values were assigned.

From the results, it can be seen that since all variables start from 0, no matter what type of value is assigned, all variables will have a value. However, values of the wrong type will not make sense, so be sure to convert to the correct type when retrieving the value. However, this is something that needs to be taken care of when using object casting as well :P

In addition, I used `partial` in the implementation of AnyStruct for convenience. For example, at work we use Unity, and we have written the conversion of Unity's structs, such as `Vector2`, `Vector3`, `Quaternion`, etc., in another cs file. We even separated our own custom structs into another file. In future projects, we can simply copy the first cs file :)

<br/>
<br/>

# 嘗試製作 struct 基底類，解決轉型 Boxing

寫 C# 久了總是對 object 轉任何 struct 會產生 boxing 感到苦惱，最近花點時間 google 一下發現這篇討論:

[How to store structs of different types without boxing](https://stackoverflow.com/questions/6163335/how-to-store-structs-of-different-types-without-boxing)

文章中的 Alternative 3 解法相當有趣，製作了 struct 基底類並實作各類轉型解決 Boxing。由於文章中的程式碼有些微的語法錯誤，自己有做些修改:

```csharp
using System.Runtime.InteropServices;

/// <summary>
/// 用於儲存任何 struct 避免 boxing
/// </summary>
[StructLayout(LayoutKind.Explicit)]
public partial struct AnyStruct
{
    public static readonly AnyStruct empty = new AnyStruct();

    [FieldOffset(0)] public int intVal;
    [FieldOffset(0)] public float floatVal;
    [FieldOffset(0)] public byte byteVal;

    public static implicit operator int(AnyStruct _obj) { return _obj.intVal; }
    public static implicit operator float(AnyStruct _obj) { return _obj.floatVal; }
    public static implicit operator byte(AnyStruct _obj) { return _obj.byteVal; }

    public static implicit operator AnyStruct(int _val)
    {
        var r = empty;
        r.intVal = _val;

        return r;
    }

    public static implicit operator AnyStruct(float _val)
    {
        var r = empty;
        r.floatVal = _val;

        return r;
    }

    public static implicit operator AnyStruct(byte _val)
    {
        var r = empty;
        r.byteVal = _val;

        return r;
    }
}
```

做一個轉換型別的基底類來解決 Boxing 其實沒什麼特別，大家應該都想過這麼做。但特別的是作者用了 `[StructLayout(LayoutKind.Explicit)]`，StructLayout 允許我們控制 struct 在記憶體中的排列方式，然後巧妙地搭配 `[FieldOffset(0)]`，讓所有變數都從頭開始。這作法不管我們之後在 AnyStruct 裡實作多少類別轉型，其大小僅會是這些變數裡最大的那個。舉例來說，上述程式碼實作出 int、float、byte 的轉型，所以 AnyStruct 大小僅會是 4。如果我們再實作 long 的轉型，AnyStruct 大小就會來到 8。這樣我們就不用擔心實作越多，AnyStruct 越胖。(還是會胖啦，但只會挑變數裡，最胖的那個)

現在我們來看 AnyStruct 的使用與結果:

```csharp
private static void Main()
{
    Console.WriteLine($"AnyStruct Size = {Marshal.SizeOf(AnyStruct.empty)}\n");
    // AnyStruct Size = 4

    AnyStruct any = AnyStruct.empty;
    Console.WriteLine($"=== Original === \n{PrintAnyStruct(ref any)}\n");
    // === Original ===
    // int     = 0
    // float   = 0
    // byte    = 0

    any = 1234;
    Console.WriteLine($"=== Set Int 1234 === \n{PrintAnyStruct(ref any)}\n");
    // === Set Int 1234 ===
    // int     = 1234
    // float   = 1.729E-42
    // byte    = 210

    any = 4.56f;
    Console.WriteLine($"=== Set Float 4.56f === \n{PrintAnyStruct(ref any)}\n");
    // === Set Float 4.56f ===
    // int     = 1083304837
    // float   = 4.56
    // byte    = 133

    any = (byte)78;
    Console.WriteLine($"=== Set Byte 78 === \n{PrintAnyStruct(ref any)}\n");
    // === Set Byte 78 ===
    // int     = 78
    // float   = 1.1E-43
    // byte    = 78
}

private static string PrintAnyStruct(ref AnyStruct _anyStruct)
{
    return $"int \t= {(int)_anyStruct} \nfloat \t= {(float)_anyStruct} \nbyte \t= {(byte)_anyStruct}";
}
```

1. 最開始利用 `Marshal.SizeOf()`確認 AnyStruct 的大小，有興趣的人可以實作其他型別看看變化
2. 使用 AnyStruct 相當簡單，因為我們已經實作了隱式轉型，所以可以直接塞值。要注意的地方只有取值時要轉成對的型別
3. 範例中，我列印出存入各種值時，其 AnyStruct 內部各變數的值有何變化

從結果可看出，由於各變數都是從 0 開始，不論我們塞什麼型別的值進去，所有變數都會有值。而錯的型別的值會是沒意義的，所以取值時記得轉為對的型別。不過這件事在使用 object 轉型時本來就要注意 :P

另外，我在 AnyStruct 的實作上使用了 `partial`，原因是方便抽取。例如工作上會用到 Unity，我們將 Unity 的 struct 諸如 `Vector2`、`Vector3`、`Quaternion` 等轉換寫在另一份 cs 檔中。甚至，我們將自己寫的 struct 又另外拆出一份。往後的專案只要移植第一份 cs 檔即可 :)
