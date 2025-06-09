Bu bir creational pattern'dir. 
Amaç: bir nesne oluşturulurken, bu nesnenin başka bir nesneden kopyalanarak oluşturulmasını sağlamaktır.

- [Shallow Copy](#shallow-copy)
- [Deep Copy](#deep-copy)
    - [ICloneable](#icloneable)
    - [Explicit Deep Copy](#explicit-deep-copy)
    - [Deep Copy Through Serialization](#deep-copy-through-serialization)
- [EK1 Constructor Copy](#ek1-constructor-copy)
- [Kaynakça](#kaynakça)
## Shallow Copy

Aşağıda .Net ile gelen ICloneable interface'ini implemente eden bir Order class ve property'sinde bulunan Address class'ının implementasyonu gözlemlenmektedir.

```
public class Order : ICloneable
{
    public string OrderId { get; set; }
    public string ProductName { get; set; }
    public Address Address { get; set; }
    public Order(string orderId, string productName, Address address)
    {
        OrderId = orderId;
        ProductName = productName;
        Address = address;
    }

    public object Clone()
    {
        return new Order(OrderId, ProductName, Address);
    }

    public override string ToString()
    {
        return $"OrderId: {OrderId}, ProductName: {ProductName}, Address: {Address.City}, {Address.ZipCode}";
    }
}

public class Address
{

    public string City { get; set; }
    public string ZipCode { get; set; }
    public Address(string city, string zipCode)
    {
        City = city;
        ZipCode = zipCode;
    }
}
```

Bu class'da gözlemlendiği üzere ICloneable interface'inden gelen object return eden bir Clone methodu bulunmaktadır. Bu methodun içerisinde Order class'ının bütün propertyleri direkt kullanılarak yeni bir Order nesnesi yaratılmıştır. Bu şekilde kullanım teknik olarak nesnenin tamamen kopyalanmamasına sebebiyet vermektedir ve bu kullanıma Shallow Copy denir.

```
internal class Program
{
    private static void Main(string[] args)
    {
        #region Prototype Pattern ICloneable Example
		var order1 = new CloneableOrder("123", "Laptop", new Address("Bursa", "16000"));
		var order2 = (CloneableOrder)order1.Clone();
		
		Console.WriteLine("Before change:");
		Console.WriteLine(order1);
		Console.WriteLine(order2);
		
		order2.OrderId = "456"; 
		order2.Address.City = "Istanbul";
		order2.Address.ZipCode = "34000";
		
		Console.WriteLine("After change:");
		Console.WriteLine(order1);
		Console.WriteLine(order2);
		#endregion Prototype Pattern ICloneable Example
	}
}
```

Yukarıdaki kullanım incelendiğinde öncelikle `order1` initialize edilmektedir. ardından `order2`
isimli değişkenin içerisine `order1`in kopyası/klonu verilmektedir. Clone order2 üzerinde yapılan değişikliklerin order1'i etkilememesini bekleriz ancak `Address` property'si reference type olduğundan kodun çıktısından da görüleceği üzere yapılan değişiklikler `order1`i de etkilemektedir.

```
Before change:
OrderId: 123, ProductName: Laptop, Address: Bursa, 16000
OrderId: 123, ProductName: Laptop, Address: Bursa, 16000
After change:
OrderId: 123, ProductName: Laptop, Address: Istanbul, 34000
OrderId: 456, ProductName: Laptop, Address: Istanbul, 34000
```

Görüldüğü üzere `Address` property'si iki nesnede de güncellendi ancak `OrderId` value type olduğundan kaynaklı değişmedi. Bu duruma Shallow Copy denmektedir.

# Deep Copy

Deep copy işlemi bir nesnenin bütün propertylerinin reference veya value type farketmeden kopyalanması işlemidir. Buna farklı farklı çözümler ile yaklaşılmaktadır.

## ICloneable

Shallow Copy'de bahsedilen işlemin iç içe tekrarlı bir şekilde implemente edilmesiyle Deep Copy sağlanabilmektedir.

Shallow Copy'deki örnekteki Order class'ının içerisindeki `Clone` methodunda `Address` propertysi ICloneable interface'ini implemente ederse ve Order class'ının Clone methodu `Address` in clone'unu kullanımında işlem tamamlanmış oluyor. Ancak fazla karmaşık nesnelerde, bu implementasyon da karmaşıklaşmaktadır.

```
public class CloneableOrder : ICloneable
{
    public string OrderId { get; set; }
    public string ProductName { get; set; }
    public CloneableAddress Address { get; set; }
    public CloneableOrder(string orderId, string productName, CloneableAddress address)
    {
        OrderId = orderId;
        ProductName = productName;
        Address = address;
    }

    public object Clone()
    {
        return new CloneableOrder(OrderId, ProductName, (CloneableAddress)Address.Clone());
    }

    public override string ToString()
    {
        return $"OrderId: {OrderId}, ProductName: {ProductName}, Address: {Address.City}, {Address.ZipCode}";
    }
}

public class CloneableAddress : ICloneable
{
    public string City { get; set; }
    public string ZipCode { get; set; }
    public CloneableAddress(string city, string zipCode)
    {
        City = city;
        ZipCode = zipCode;
    }

    public object Clone()
    {
        return new CloneableAddress(City, ZipCode);
    }
}
```

program.cs
```
internal class Program
{
    private static void Main(string[] args)
    {
        #region Prototype Pattern ICloneable CloneableOrder Example
        var order5 = new CloneableOrder("123", "Laptop", new CloneableAddress("Bursa", "16000"));
        var order6 = (CloneableOrder)order5.Clone();

        Console.WriteLine("Before change:");
        Console.WriteLine(order5);
        Console.WriteLine(order6);

        order6.OrderId = "456";
        order6.Address.City = "Istanbul";
        order6.Address.ZipCode = "34000";

        Console.WriteLine("After change:");
        Console.WriteLine(order5);
        Console.WriteLine(order6);
        #endregion Prototype Pattern ICloneable CloneableOrder Example
    }
}
```

Yukarıda bahsedildiği gibi kod güncellendikten sonra çıktı Deep Copy yapıldığını kanıtlar biçimde düzelmektedir.

Çıktı:
```
Before change:
OrderId: 123, ProductName: Laptop, Address: Bursa, 16000
OrderId: 123, ProductName: Laptop, Address: Bursa, 16000
After change:
OrderId: 123, ProductName: Laptop, Address: Bursa, 16000
OrderId: 456, ProductName: Laptop, Address: Istanbul, 34000
```

Not: ICloneable interface'inin implementasyonu gereği return edilen nesneler object tipindedir. Bundan kaynaklı olarak her `copy` edilen nesneyi olması gereken tipine `cast` işlemi yapılması gerekmektedir. 

Ek: [Sadece Constructor'lar ile de clone işlemi yapılabilmektedir.](#ek1-constructor-copy)

## Explicit Deep Copy

Yukarıdaki `ICloneable` interface'iyle yapılan işlemlerdeki object tipinde geri dönüşün sebebiyet verdiği zorunlu casting operasyonunun önüne geçebilmek için custom interface kullanımı gerekmektedir.

```
public interface ICustomCloneable<T>
{
    T Clone();
}
```

Aşağıdaki kod örneklerinde gözlemlendiği üzere yukarıdaki eski implementasyonlardakinden farklı olarak `Clone()` işleminden sonra `Casting` yapılmamaktadır.

```
public class CustomCloneableOrder : ICustomCloneable<CustomCloneableOrder>
{
    public string OrderId { get; set; }
    public string ProductName { get; set; }
    public CustomCloneableAddress Address { get; set; }
    public CustomCloneableOrder(string orderId, string productName, CustomCloneableAddress address)
    {
        OrderId = orderId;
        ProductName = productName;
        Address = address;
    }

    public CustomCloneableOrder Clone()
    {
        return new CustomCloneableOrder(OrderId, ProductName, Address.Clone());
    }

    public override string ToString()
    {
        return $"OrderId: {OrderId}, ProductName: {ProductName}, Address: {Address.City}, {Address.ZipCode}";
    }
}

public class CustomCloneableAddress : ICustomCloneable<CustomCloneableAddress>
{
    public string City { get; set; }
    public string ZipCode { get; set; }
    public CustomCloneableAddress(string city, string zipCode)
    {
        City = city;
        ZipCode = zipCode;
    }

    public CustomCloneableAddress Clone()
    {
        return new CustomCloneableAddress(City, ZipCode);
    }
}

internal class Program
{
    private static void Main(string[] args)
    {
        #region Prototype Pattern ICustomCloneable Example
        var order7 = new CustomCloneableOrder("123", "Laptop", new CustomCloneableAddress("Bursa", "16000"));
        var order8 = order7.Clone();

        Console.WriteLine("Before change:");
        Console.WriteLine(order7);
        Console.WriteLine(order8);

        order8.OrderId = "456";
        order8.Address.City = "Istanbul";
        order8.Address.ZipCode = "34000";

        Console.WriteLine("After change:");
        Console.WriteLine(order7);
        Console.WriteLine(order8);
        #endregion Prototype Pattern ICustomCloneable Example
    }
}
```

Çıktı aşağıda görüldüğü üzere olması gerektiği gibi gelmektedir.
```
Before change:
OrderId: 123, ProductName: Laptop, Address: Bursa, 16000
OrderId: 123, ProductName: Laptop, Address: Bursa, 16000
After change:
OrderId: 123, ProductName: Laptop, Address: Bursa, 16000
OrderId: 456, ProductName: Laptop, Address: Istanbul, 34000
```


## Deep Copy Through Serialization

Aşağıdaki extension method ile herhangi bir objeyi serialize ettikten sonra deserialize işlemiyle deep copy yapılabilmektedir.

```
public static class SerializationHelper
{
    public static T DeepClone<T>(this T obj)
    {
        var json = JsonSerializer.Serialize(obj);
        return JsonSerializer.Deserialize<T>(json);
    }
}
```

```
internal class Program
{
    private static void Main(string[] args)
    {
        #region Prototype Pattern Serialization Copy Example
        var order9 = new Order("123", "Tablet", new Address("Bursa", "16000"));
        var order10 = order9.DeepClone();
        
        Console.WriteLine("Before change:");
        Console.WriteLine(order9);
        Console.WriteLine(order10);
        
        order10.OrderId = "789";
        order10.Address.City = "Izmir";
        order10.Address.ZipCode = "35000";
        
        Console.WriteLine("After change:");
        Console.WriteLine(order9);
        Console.WriteLine(order10);
        #endregion Prototype Pattern Serialization Copy Example
    }
}

```

```
Before change:
OrderId: 123, ProductName: Tablet, Address: Bursa, 16000
OrderId: 123, ProductName: Tablet, Address: Bursa, 16000
After change:
OrderId: 123, ProductName: Tablet, Address: Bursa, 16000
OrderId: 789, ProductName: Tablet, Address: Izmir, 35000
```

 Görüldüğü üzere, diğer implementasyonlara göre çok daha basit bir implementasyon sunar.
 
# EK1 Constructor Copy

```
    public Order(Order order)
    {
        OrderId = order.OrderId;
        ProductName = order.ProductName;
        Address = new Address(order.Address);
    }
```

```
    public Address(Address address)    
    {
        City = address.City;
        ZipCode = address.ZipCode;
    }
```

Order ve Address sınıfına yukarıdaki constructorların eklenmesi ile şu şekilde bir clonelama işlemi yapılabilmektedir:
```
internal class Program
{
    private static void Main(string[] args)
    {
        #region Prototype Pattern Constructor Copy Example
        var order3 = new ConstructorCopyOrder("123", "Smart Phone", new ConstructorCopyAddress("Bursa", "16000"));
        var order4 = new ConstructorCopyOrder(order3);

        Console.WriteLine("Before change:");
        Console.WriteLine(order3);
        Console.WriteLine(order4);

        order4.OrderId = "101112";
        order4.Address.City = "Ankara";

        Console.WriteLine("After change:");
        Console.WriteLine(order3);
        Console.WriteLine(order4);
        #endregion Prototype Pattern Constructor Copy Example

    }
}

```

```
Before change:
OrderId: 123, ProductName: Smart Phone, Address: Bursa, 16000
OrderId: 123, ProductName: Smart Phone, Address: Bursa, 16000
After change:
OrderId: 123, ProductName: Smart Phone, Address: Bursa, 16000
OrderId: 101112, ProductName: Smart Phone, Address: Ankara, 16000
```




# Kaynakça

https://www.youtube.com/watch?v=fqaoCDyxb1w
https://refactoring.guru/design-patterns/prototype
https://medium.com/kodcular/prototype-design-pattern-prototip-kal%C4%B1b%C4%B1-nedir-76d011e01fec
https://www.udemy.com/course/design-patterns-csharp-dotnet
