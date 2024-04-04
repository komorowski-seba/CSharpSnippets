# Why I am not a fan of automapper, this is a few reasons

- Automapper causes increased api startup time
- Slower mapping time compared to manual mapping
- Method of using

## 1 Automapper causes increased api startup time
This is caused becouse the automapper must  run the configuration before use.
When he building a mapping, uses reflection and allocates memory which involves consuming resources.

```c#
cfg =>
{
    cfg.CreateMap<User, UserDto>();
    cfg.CreateMap<Address, AddressDto>();
});
```

```c#
[Benchmark]
public void ConfigurationMapperTest_2Map()
{
    var config = new MapperConfiguration(
        cfg =>
        {
            cfg.CreateMap<User, UserDto>();
            cfg.CreateMap<Address, AddressDto>();
        });
    var mapper = config.CreateMapper();
}
```

|                                            Method |          Mean | Allocated |
|-------------------------------------------------- |--------------:|----------:|
|                      ConfigurationMapperTest_1Map | 141,132.11 ns |  39,332 B |
|                      ConfigurationMapperTest_2Map | 149,893.70 ns |  51,743 B |
|                      ConfigurationMapperTest_3Map | 174,131.30 ns |  68,252 B |
|                      ConfigurationMapperTest_4Map | 192,303.10 ns |  80,444 B |

As you can see, the average allocation for a simple object can be about **10 KB**.
They can guess that with quite a lot of objects, this can consume a lot of resources.

## 2 Slower mapping time compared to manual mapping
as shown by benchmarks (https://mapperly.riok.app/docs/intro/)

|Method	       | Mean           | Allocated |
| ------------ | -------------: | --------: |
|AutoMapper    |	1,203.9 ns  | 1,904 B   |
|ManualMapping |	529.6 ns    | 1,160 B   |
|Mapperly      |	338.5 ns    | 920 B     |

The **Automaper** is slower than manual mapping or the 
excellent **Mapperly**.

## 3 Method of using
How useing automapper can seen below

```c#
var dto = mapper.Map<UserDto>(User);
```

For me, is more readable using extension method
In the case of many mappers, IDLE can will tell you what you can do with the object. 

```c#
var abc = object.ToABC();
var xyz = object.ToXYZ();
```

Mapperly also can you to create mappers in the form of extension method

```c#
[Mapper]
public static partial class UserMapper
{
    public static partial UserDto ToUserDto(this User user);
}
```

Another thing that is not apparent at first look is the use of the **IoC** for automapper, 
this is a hidden cost beyond the mapping
ManualMapping or Mapperly do not generate of that cost.

I think these 3 points, pretty much speak for manual mapping or Mapperly