**Genel Tanım:** Singleton kullanılırken uygulanması gereken başlıca kurallardan birisi, ilgili class'ın (sınıf) constructor'ının (yapıcı) private (özel) olması ve en fazla bir kere çalıştırılmasıdır. Bununla birlikte bu class'ın instance'ına (örnek) her yerden ulaşılabilmesi gerekmektedir.

Singleton design pattern (tasarım kalıbı), şu şekilde özetlenebilir: Bir uygulamada singleton olan bir class'ın instance'ı oluşturulduktan itibaren, uygulama bitene kadar aynı instance'a sahip olur. Yani, herhangi bir yerde kullanıldığında, eğer değiştirilebilir değişkenleri bulunuyorsa, bu değişkenler değiştirildikten sonra uygulamanın başka bir yerinde tekrar bu class çağırıldığında değiştirilmiş halleri gelir.

Genelde loglama servislerinde, factory (fabrika) class'larda, uygulamaların her yerinde kullanılacak (global) class'larda kullanılmaktadır. Bazı kaynaklarda Veritabanı bağlantılarında da kullanılabildiğine rastlamak mümkün ancak tek bir bağlantı ile her ne kadar Thread-safe bir şekilde bağlantı yapılacak şekilde ayarlansa da `Bottle Neck` (Dar boğaz) problemiyle karşılaşıyoruz. Bu durumla karşılaşmamak için `Connection Pooling` kullanılmaktadır.

Aşağıdaki kod parçasında görüldüğü üzere static ve uygulama ayağa kalktığı gibi oluşturulan eager bir singleton loglama class'ı görülmektedir. Eager Initialization diye nitelendirilmesinin sebebi uygulama ayağa kalktığı gibi bu servisin instance'ı oluşturulup değişkene atanmaktadır. Bu koddan anlaşıldığı üzere SingletonLoggingService sadece bir kere `instantiate` (oluşturulmak) edilmekle birlikte sadece o instance'ın döndürülmesi sağlanmaktadır. Koşul içerisinde ise daha güvenli olsun diye bir koşul konmuş ve böylece `_instance`'ın oluşturulmama ihtimali ortadan kalkmaktadır.

```csharp
public class SingletonLoggingService
{

    private static SingletonLoggingService _instance = new SingletonLoggingService();

    public static SingletonLoggingService Instance
    {
        get
        {
            if (_instance is null) _instance = new SingletonLoggingService();
            return _instance;
        }
    }

    public void Information(string message)
    {
       Console.WriteLine(message);
    }
}
```

Tahmin edilebileceği üzere, singleton pattern ile oluşturulmuş bir class'a multi-thread bir uygulamada aynı anda farklı thread'lerden erişilmeye çalışılırsa sıkıntılı sonuçlar elde edilebilmektedir. Buna çözüm olarak thread-safe implementasyonlar yapılması gerekmektedir. Buna çözüm olarak lock kullanabiliriz.

```csharp
public sealed class ThreadSafeLoggingService
{

    private static ThreadSafeLoggingService _instance;
    private static readonly object _lock = new();

    public static ThreadSafeLoggingService Instance
    {
        get
        {
            if (_instance is not null) return _instance;

            lock (_lock)
            {
                _instance ??= new ThreadSafeLoggingService();
            }

            return _instance;
        }
    }

    public void Information(string message)
    {
       Console.WriteLine(message);
    }
}

```

Görüldüğü üzere yukarıdaki kod parçası ilgili servis ne zaman çağırılırsa o zaman initialize edilmekle birlikte lock ile kullanımda olduğu zamanlarda thread safe halini almakta. Bu şekilde kullanım sağlandığı zaman initialize edilmesine Lazy initialization denir ve Lazy\<T> kullanarak da implemente edilebilir. `get` bloğunu tamamen lock'a alabilirdik ancak `_instance` yaratılmış ise direkt bunu dönerek ve bu bloğu lock'a almayarak performans sağlamaktayız çünkü lock pahalı bir mekanizmadır. Eğer her return işleminde lock içerisine almamamız thread-safe çalışmasını engelleyeceğini düşünüyorsanız yanılıyorsunuz. Bir kere lock içerisinde instance oluşturmak onu zaten thread-safe yapacağı için tekrar tekrar bu şekilde return etmemize gerek bulunmamaktadır. Bu yöntem `double check locking` ismiyle nitelendirilmektedir. Ek olarak class'ı deklare ederken sealed anahtar kelimesini kullandık. Bunun sayesinde bu class herhangi bir class'a inherit (miras almak) yapamaz. Bunu yapma sebebimiz aslında ek bir koruma. Normal şartlar altında constructor private olduğundan kaynaklı inheritence (miras)(kalıtım) yapıldığı zaman hata verecek ancak inner (iç) class şeklinde oluşturulursa hata vermeyecek ve o oluşturulan inner class' da kendine ait base class yani singleton class'ımızdan instantiate edilmiş bir instance'ı olacak ve singleton class'ımız singleton'lıktan çıkacak. 

```csharp
public class ThreadSafeLoggingService
{
    public int counter = 0;

    private static ThreadSafeLoggingService _instance;
    private static readonly object _lock = new();

    private ThreadSafeLoggingService()
    {
        counter++;
    }
    public static ThreadSafeLoggingService Instance
    {
        get
        {
            if (_instance is not null) return _instance;

            lock (_lock)
            {
                _instance ??= new ThreadSafeLoggingService();
            }

            return _instance;
        }
    }

    public void Information(string message)
    {
        Console.WriteLine(message);
    }

    public class DerivedSingletonClass : ThreadSafeLoggingService { }
}
```
Yukarıdaki kod parçacığında bahsedilen problemi görebilirsiniz. Program içerisinde bunu test etmek için ise counter değerini kullanabiliriz.
```csharp
static void Main(string[] args)
{
    var x = new DerivedSingletonClass();
	var y = ThreadSafeLoggingService.Instance;
	x.counter += 10;
	y.counter += 20;
	Console.WriteLine($"Derived {x.counter}");
	Console.WriteLine("Base " + ThreadSafeLoggingService.Instance.counter);
}
```
Yukarıdaki denemede çıktı olarak `DerivedSingletonClass`'ın counter'ı 11 olacak ve `SingletonLoggingService`'in `counter`'ı ise 21 olacak ancak ikisi de `SingletonLoggingService`'in constructor'unu kullandıkları için singleton olma kuralını çiğniyor ve uygulamamızda bize sıkıntı çıkarabilme potansiyeline sahip bir hal almış oluyor.

`Lazy<\T>` kullanarak yapılan implementasyonuna bir örnek:
```csharp
public sealed class ThreadSafeLoggingService
{
    private static readonly Lazy<ThreadSafeLoggingService> _instance = new Lazy<ThreadSafeLoggingService>();

    public static ThreadSafeLoggingService Instance
    {
        get
        {
            return _instance.Value;
        }
    }

    public void Information(string message)
    {
       Console.WriteLine(message);
    }
}
```

Burada `lock` kullanılmamasının sebebi Lazy class'ı default olarak Thread-safe bir instance üretmektedir.

Yukarıdaki şekilde yapılan static implementasyon güzel bir şekilde çalışıyor ancak test yazmak aşırı zor ve neredeyse mümkün değil. Bu yüzden Dependency Injection için Singleton service lifetime'da (servis yaşam döngüsü) servis collection'umuza register (kayıt) edeceğimiz bir servisimiz ile test edilebilir bir şekilde Singleton servis oluşturabiliriz.

Örnek:

```csharp
public interface IMySingleton
{
    string GetSingletonName(string name);
}

public class MySingleton : IMySingleton
{
    public string GetSingletonName(string name)
    {
        return $"Hard-named\\{name}";
    }
}
```

Yukarıdaki örnekte IMySingleton interface'inden türemiş bir MySingleton concrete class'ımız bulunmaktadır. Burada GetSingletonName fonksiyonu içerisine gönderilen herhangi bir name parametresinden başına `Hard-named` stringi eklenerek bir çıktı almaktayız. Bu sınıfı Singleton Service Lifetime ile Collection'umuza kaydettikten sonra herhangi bir sınıfta Dependency Injection ile kullanabiliriz anlamına gelmektedir.

Aşağıdaki kod parçacığında ise yukarıda oluşturduğumuz sınıfımızın Dependency Injection ile kullanımını görmekteyiz:

```csharp
public class Something
{
    private readonly IMySingleton _mySingleton;

    public Something(IMySingleton mySingleton)
    {
        _mySingleton = mySingleton;
    }

    public string GetSingletonNameInSomething(string name)
    {
        return _mySingleton.GetSingletonName(name);
    }
}
```

Eğer bu kodu bir konsol uygulamasında çalıştırmak istersek aşağıdaki örnek ile bunu başarabiliriz:

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        ServiceProvider serviceProvider = new ServiceCollection()
                                            .AddSingleton<IMySingleton, MySingleton>()
                                            .BuildServiceProvider();

        var something = new Something(serviceProvider.GetService<IMySingleton>());
        Console.WriteLine(something.GetSingletonNameInSomething("Emre"));
    }
}
```

Bu işlemlerin bize en büyük faydası, artık unit test yazımı çok daha basit bir hale geldi. 
```csharp
public class Test
{
    [Fact]
    public void TestMethod()
    {
        // Arrange
        var mockSingleton = new Mock<IMySingleton>();

        mockSingleton.Setup(x => x.GetSingletonName("Emre")).Returns("Mock\\Emre");

        // Act
        var concrete = new Something(mockSingleton.Object);
        var result = concrete.GetSingletonNameInSomething("Emre");

        // Assert

        Assert.Equal("Mock\\Emre", result);
    }
}
```

Yukarıdaki örnekten görüldüğü üzere `MySingleton` sınıfının interface'inin mock kütüphanesi ile mocklandıktan sonra Setup ile eğer `GetSingletonName` fonksiyonuna `Emre` parametresi ile bir istek atıldığında fonksiyonun kendi içeriğini umursamadan çıktı olarak `Mock\Emre` dönmesini sağlıyoruz ve testimizi istediğimiz şekilde şekillendirebiliyoruz.

Statik servisin test edilmesi gerekliyse bir preprocessor yardımı veya direkt manuel olarak bu servisi kullanacağımız yerlerde ilgili using'i başka bir yerde oluşturduğumuz test için mock bir class'ın using'iyle değiştirebiliriz. Bu mock class'ın ismi gerçek olanla aynı olmalı ki kodda bir değişikliğe veya using'de yapıldığı gibi preprocessor veya her kullanılan yerde koşul koyma ihtiyacı ortadan kaldırılmış olur.  

### Ek

Ek olarak çoğu kaynak (Erich Gamma'da dahil) Singleton Pattern'i code smell olarak nitelendirmektedir. Bu kişiden kişiye değişmekle birlikte benim düşüncem bütün projede kullanılması gereken global runtime konfigurasyon içeren class'lar ve Factory Pattern içeren class'larda kullanılması pek smell olarak düşündürtmüyor. Factory Pattern içerenlerin neden smell olarak düşünmememe Factory Pattern kısmında anlatacağım (tabi DI kullanılarak oluşturulacak olan Singleton service lifetime'ından bahsediyorum static olan için geçerli değil bu düşüncem). Ancak dediğim gibi çoğu kişi bu global access olayının yönetilmesi konusunda zorluklar çıkacağının ve ambiguity (belirsizlik) oluşturma ihtimalinin yüksek olacağını savunsa da Runtime konfigurasyonlara globalda erişilebilmek gibi ihtiyacımız olan yerler de bulunmaktadır. 

Not: Runtime konfigurasyonlardan bahsedecek olursam örneğin üç adet farklı katmanlarda servislerimiz var. Bu servislerin bunlardan ayrı bir katmanda kaydedilmesini istiyorum ve uygulamamızın sadece seçeceğimiz servisleri bu ayrı katmana kaydetmesini ve kullanmasını istiyoruz. Runtime'da bu servislerin hangilerinin kullanılacağı belli olacağı için kullanılacak katmanlar kendi içerisinde global singleton bir collector class'a kendi servislerinin uygulamada kullanılacağını belirtir ve böylece ayrı katmanımız hangi servislerin kullanıldığından haberdar olup işlemlerini ona göre yapabilir.

# Kaynaklar

[Singleton Pattern - Design Patterns (ep 6) Youtube Video ~Christopher Okhravi](https://www.youtube.com/watch?v=hUE_j6q0LTQ)

[Singleton Pattern Nedir ? | Design Patterns | Tasarım Kalıpları - YouTube ~Tech Buddy](https://www.youtube.com/watch?v=vISy3_0mxT8)

[Singleton Design Pattern in C# – Part 1 – Code Teddy](https://codeteddy.com/2018/01/12/singleton-design-pattern-in-c-part-1/)

[Singleton Design Pattern In C# – Part 2 (Eager and Lazy Initialization in Singleton) – Code Teddy](https://codeteddy.com/2018/01/12/singleton-design-pattern-in-c-part-2-eager-and-lazy-initialization-in-singleton/)

[Singleton Design Pattern In C# – Part Three (Static vs Singleton) – Code Teddy](https://codeteddy.com/2018/01/12/singleton-design-pattern-in-c-part-three-static-vs-singleton/)

[Is This Madness? Singleton Pattern to Manage Database Connections](https://medium.com/@eyupcanarslan/is-this-madness-singleton-pattern-to-manage-database-connections-a0baa01c1f3)


