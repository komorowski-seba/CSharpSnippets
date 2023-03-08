**.Net** is going more and more into performance, 
and **Span<>** is more often seen in the in code.
So I decided to use this in myself as well, 
I wanted used **span<>** in the methods
to validation the **Nip**, **Pessel** and **Regon**.
I wanted them to not allocate memory on the **heap**, and I wanted to
see how efficient they cud would be compared to the version of the 
operating an **array**.

So I carved something like this

### CheckSumNip

```c#
public static bool CheckSumNip(string value)
{
    var span = value.AsSpan();
    if (span.Length < 10)
        return false;
    
    var result = 0;
    var sum = 0;
    var resultIndex = 0;
    ref var search = ref MemoryMarshal.GetReference(span);
    for (var i = 0; i < span.Length; ++i)
    {
        var currentChar = Unsafe.Add(ref search, i);
        if (currentChar is < '0' or > '9')
            continue;

        var val = currentChar - '0';
        if (resultIndex >= 9)
        {
            result = val;
            break;
        }
        
        sum += resultIndex switch
        {
            0 => val * 6,
            1 => val * 5,
            2 => val * 7,
            3 => val * 2,
            4 => val * 3,
            5 => val * 4,
            6 => val * 5,
            7 => val * 6,
            8 => val * 7,
            _ => 0
        };
        ++resultIndex;
    }
    return (sum % 11) == result;
}
```

<br />

### CheckSumPesel

```c#
public static bool CheckSumPesel(string value)
{
    var span = value.AsSpan();
    if (span.Length < 11)
        return false;
    
    var result = 0;
    var sum = 0;
    var resultIndex = 0;
    ref var search = ref MemoryMarshal.GetReference(span);
    for (var i = 0; i < span.Length; ++i)
    {
        var currentChar = Unsafe.Add(ref search, i);
        if (currentChar is < '0' or > '9')
            continue;

        var val = currentChar - '0';
        if (resultIndex >= 10)
        {
            result = val;
            break;
        }
        
        sum += resultIndex switch
        {
            0 => val * 1,
            1 => val * 3,
            2 => val * 7,
            3 => val * 9,
            4 => val * 1,
            5 => val * 3,
            6 => val * 7,
            7 => val * 9,
            8 => val * 1,
            9 => val * 3,
            _ => 0
        };
        ++resultIndex;
    }

    sum %= 10;
    if (sum == 0) sum = 10;
    sum = 10 - sum;
    
    return sum == result;
}
```

<br />

### CheckSumRegon

```c#
public static bool CheckSumRegon(string value)
{
    var span = value.AsSpan();
    if (span.Length < 9)
        return false;
    
    var result = 0;
    var sum = 0;
    var resultIndex = 0;
    ref var search = ref MemoryMarshal.GetReference(span);
    for (var i = 0; i < span.Length; ++i)
    {
        var currentChar = Unsafe.Add(ref search, i);
        if (currentChar is < '0' or > '9')
            continue;

        var val = currentChar - '0';
        if (resultIndex >= 8)
        {
            result = val;
            break;
        }
        
        sum += resultIndex switch
        {
            0 => val * 8,
            1 => val * 9,
            2 => val * 2,
            3 => val * 3,
            4 => val * 4,
            5 => val * 5,
            6 => val * 6,
            7 => val * 7,
            _ => 0
        };
        ++resultIndex;
    }

    sum %= 11;
    if (sum == 10) sum = 0;
    
    return sum == result;
}
```

<br />

Using

```c#
var nipResult = CheckSumNip("9--489.0.8(25)46");
var peselResult = CheckSumPesel("0--2-240..968()157");
var regonResult = CheckSumRegon("51-(3137)1.3.0");

Console.WriteLine($"Nip ==>> {nipResult}");       // Nip ==>> true
Console.WriteLine($"Pesel ==>> {PeselResult}");   // Pesel ==>> true
Console.WriteLine($"Regon ==>> {regonResult}");   // Regon ==>> true
```


I decided to run tests with with version based on the Array

```c#
public class BenchmarkTest
{
    [Benchmark]
    public void CheckNipSpan1000()
    {
        for (var i = 0; i < 1000; ++i)
            CheckNipPesselRegon.CheckSumNip("9==--#$%$#@$#@$%^&**((4()(8==9.0.8....2..5~~~4ERTWEFSDGFDDFZARQ6");
    }

    [Benchmark]
    public void CheckNipArray1000()
    {
        for (var i = 0; i < 1000; ++i)
            CheckNipPesselRegon.CheckSumNipArray("9==--#$%$#@$#@$%^&**((4()(8==9.0.8....2..5~~~4ERTWEFSDGFDDFZARQ6");
    }
    ...
}
```

### CheckSumNipArray

```c#
public static bool CheckSumNipArray(string value)
{
    var array = value.ToArray();
    ...    
    for (var i = 0; i < array.Length; ++i)
    {
        var currentChar = array[i];
    ...
}
```

<br />

|              Method |        Mean |  Allocated |
|-------------------- |------------:|-----------:|
|       CheckNipSpan1 |    108.5 ns |          - |
|      CheckNipArray1 |    758.9 ns |      496 B |
|      CheckNipSpan10 |    978.3 ns |          - |
|     CheckNipArray10 |  8,127.2 ns |     4960 B |
|    CheckNipSpan1000 |    102.2 us |          - |
|   CheckNipArray1000 |    862.6 us |   496001 B |
|  CheckNipSpan100000 |  9,235.2 us |       10 B |
| CheckNipArray100000 | 76,312.8 us | 49600107 B |

<br />

As you can see, **8 times** more the speed :) <br />
This speaks for itself