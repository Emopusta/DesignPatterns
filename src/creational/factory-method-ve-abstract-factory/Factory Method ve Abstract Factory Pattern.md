
Factory Method: Belirli bir interface'den (arayüz) türeyen birden çok sınıfın çeşitli parametreler ile bir factory sınıfının içerisinde instantiate (oluşturmak) edilerek kullanılacak yere return (döndürme) edilmesi olarak özetlenebilir.

Örnek:
Öncelikle hesaplama ile alakalı bir interface oluşturalım:

```csharp
public interface ICalculator
{
    int Calculate(int a, int b);
}
```

Ardından bu interface'i implemente eden üç adet toplama, çıkarma ve çarpma sınıflarının oluşturulması ve ilgili işlemleri yapan methodların doldurulması:

```csharp
public class Plus : ICalculator
{
    public int Calculate(int a, int b)
    {
        return a + b;
    }
}
public class Minus : ICalculator
{
    public int Calculate(int a, int b)
    {
        return a - b;
    }
}

public class Multiplication : ICalculator
{
    public int Calculate(int a, int b)
    {
        return a * b;
    }
}
```

En sonda ihtiyaca bağlı olarak içine verilen parametre değerleri ile oluşturacağımız sınıfları oluşturup bize getirecek olan factory sınıfımızı yazalım.

```csharp
public class CalculateFactory : ICalculateFactory
{
    public ICalculator GetCalculator(string type)
    {
        return type switch
        {
            "+" => new Plus(),
            "-" => new Minus(),
            "*" => new Multiplication(),
            _ => throw new NotImplementedException()
        };
    }
}
```

ve örneğini yapalım:

```csharp
using Microsoft.Extensions.DependencyInjection;

namespace FactoryMethod;

internal class Program
{
    static void Main(string[] args)
    {
        ServiceProvider serviceProvider = new ServiceCollection()
                                            .AddSingleton<ICalculateFactory, CalculateFactory>()
                                            .BuildServiceProvider();

        var factory = serviceProvider.GetService<ICalculateFactory>();

        var plusResult = factory.GetCalculator("+").Calculate(5, 3);

        Console.WriteLine(plusResult);

        var minusResult = factory.GetCalculator("-").Calculate(5, 3);
        Console.WriteLine(minusResult);

        var multiplicationResult = factory.GetCalculator("*").Calculate(5, 3);
        Console.WriteLine(multiplicationResult);
    }
}
```

Bu şekilde istenilen sınıflar ilgili parametrelere göre oluşturulmaktadır ancak bu şekilde kullanım developer için çeşitli belirsizlikler ve zorluklar çıkartabilmektedir. Bu yüzden aşağıdaki kod örneğindeki gibi bir factory method kullanılırsa:

```csharp
public class CalculateFactory : ICalculateFactory
{
    public ICalculator GetPlusCalculator()
    {
        return new Plus();
    }

    public ICalculator GetMinusCalculator()
    {
        return new Minus();
    }

    public ICalculator GetMultiplicationCalculator()
    {
        return new Multiplication();
    }
}

public interface ICalculateFactory
{
    ICalculator GetPlusCalculator();
    ICalculator GetMinusCalculator();
    ICalculator GetMultiplicationCalculator();
}
```

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        ServiceProvider serviceProvider = new ServiceCollection()
                                            .AddSingleton<ICalculateFactory, CalculateFactory>()
                                            .BuildServiceProvider();

        var factory = serviceProvider.GetService<ICalculateFactory>();

        var plusResult = factory.GetPlusCalculator().Calculate(5, 3);

        Console.WriteLine(plusResult);

        var minusResult = factory.GetMinusCalculator().Calculate(5, 3);
        Console.WriteLine(minusResult);

        var multiplicationResult = factory.GetMultiplicationCalculator().Calculate(5, 3);
        Console.WriteLine(multiplicationResult);
    }
}

```

Bu şekilde kullandığımız zaman hem developer'ın kafası karışmıyor yani ne tür bir parametre girmesi gerektiğini bilmek zorunda kalmıyor hem de daha temiz bir implementasyon yapılmış olunuyor. ancak dinamik olan factory sınıfımız ilgili methodlarla sınırlandırılmış oluyor. İhtiyaçlara göre kullanımlar değişmektedir ve mühendislikte yaygın olarak dile getirildiği gibi "problemlere tek bir çözüm bulunmamaktadır".

---

Abstract Factory ise birden çok factory class oluşturularak polymorphism durumu yaratır ve farklı farklı sınıfların construct işlemlerini farklı farklı şekillerde yapılabilmesini sağlamaktadır. Ancak aynı interface'i implemente etmelidir çünkü Abstract factory'nin amacı birbirine benzer yapıların farklı factory'ler ile aynı interface çatısı altında üretilmesi ve böylelikle interface'in koyduğu şart ile polymorphism kullanımı gerçekleştirilebilir bir hal almaktadır. Aşağıdaki örnekteki `SqlDatabaseConnection` içerisindeki methodların aynı interface'den türediği için bugün Sql veri tabanı kullanılırken yarın Oracle'a geçilmesi istendiğinde tek yapılması gereken IoC container'dan ilgili IDatabaseConnectionFactory interface'ine karşılık verilen concrete sınıfın değiştirilmesi olacaktır.

```csharp
public interface IDatabaseConnectionFactory
{
    IDatabaseConnection CreateConnection();
}
public class SqlDatabaseConnectionFactory : IDatabaseConnectionFactory
{
    public IDatabaseConnection CreateConnection()
    {
        return new SqlDatabaseConnection();
    }
}

public class OracleDatabaseConnectionFactory : IDatabaseConnectionFactory
{
    public IDatabaseConnection CreateConnection()
    {
        return new OracleDatabaseConnection();
    }
}
```

Ancak bahsedildiği şekilde ortak bir interface kullanılmadan yapılan implementasyonda construct edilen sınıfların içeriğinin aynı olmaması durumunda bir sınıfta olmayan method ötekinde varsa kodda bu değişiklik sonucunda bu methodların kullanıldığı yerler ötekinde olmayacağı için ciddi sıkıntılar yaşanabilir. O yüzden factory methodların dönüş tiplerinin eğer polymorphism kullanılacaksa aynı olması gerekmektedir. bu dönüş tipini ilgili sınıfları implement eden bir interface olarak seçerek daha sağlıklı bir sınırlandırma getirmiş oluruz.
bahsettiğim worst case ise aşağıdaki gibidir:

```csharp

class SqlDatabaseConnectionFactory
{
    public static SqlDatabaseConnection CreateConnection(string connectionString)
    {
        return new SqlDatabaseConnection(connectionString);
    }
}

class OracleDatabaseConnectionFactory
{
    public static OracleDatabaseConnection CreateConnection(string connectionString)
    {
        return new OracleDatabaseConnection(connectionString);
    }
}
```

fark edildiği üzere factory class'larda herhangi bir interface ile ortak imzalama bulunmadığı için bir factory'yi ötekisinin yerine kullanmak imkansızlaşıyor. Yukarıdaki kodun biraz iyileştirilmiş versiyonu ile polymorphism odaklı bir değişim yapmasak da kodumuzu strictly coupled halden loosely coupled'a çekebiliriz.

```csharp
public interface IOracleDatabaseConnectionFactory
{
    OracleDatabaseConnection CreateConnection(string connectionString);
}

public class OracleDatabaseConnectionFactory : IOracleDatabaseConnectionFactory
{
    public static OracleDatabaseConnection CreateConnection(string connectionString)
    {
        return new OracleDatabaseConnection(connectionString);
    }
}
```


Böylelikle Dependency Injection kullanarak test edilebilir ve polymorphism'den uzak bir implementasyona sahip oluruz.

bir console uygulamasında basitleştirilmiş örneğine aşağıdan ulaşabilirsiniz:

```csharp
static void Main(string[] args)
{
    ServiceProvider serviceProvider = new ServiceCollection()
                                        .AddSingleton<IOracleDatabaseConnectionFactory, OracleDatabaseConnectionFactory>()
                                        .BuildServiceProvider();

    var factory = serviceProvider.GetService<IOracleDatabaseConnectionFactory>();

    var connection = factory.CreateConnection("connection string from secrets");

    Console.WriteLine(connection.Con);
}

public class OracleDatabaseConnection
{
    public string Con { get; set; }
    public OracleDatabaseConnection(string con)
    {
        Con = con;
    }
}
```

Bu kullanım uygulamalarımızda herhangi bir kısımda `new OracleDatabaseConnection("connstring");` kullanımlarının test edilemez bir durum oluşturmasının önüne geçilmesine yardımcı olabilir. 

Hatta bazı durumlarda herhangi bir yerde ilgili sınıfın `new` anahtar kelimesi ile strictly initialize edilmesinin kural koyularak önüne geçilmesi istendiğinde iki farklı şekilde buna ulaşılabilmektedir.

İlki eğer kullanacağınız sınıf bir kütüphane şeklinde business vb. kodlarınızın bulunduğu uygulamanıza başka bir katmandan geliyor ise sınıfınızı internal access modifier ile tanımlayıp bulunduğu katmanda oluşturulan factory sınıfıyla yaratılabilmektedir. Ancak bütün sınıflar aynı projede ise buna ayrı bir çözüm bulmamız gerekiyor.

ikinci ve çok temiz olmayan bir çözüm olan inner class tekniği ile, ilgili sınıfın constructorlarını private tanımladıktan sonra sınıfın içerisine public bir factory sınıf oluşturarak aynı katman içerisinde hem bu sınıfın herhangi bir yerde oluşturulması engellenmiş olur hem de factory ile istenildiği gibi construct işlemleri sınırlanmış olur.

ör:

```csharp
public class OracleDatabaseConnection
{
    public string Con { get; set; }
    private OracleDatabaseConnection(string con)
    {
        Con = con;
    }
    private OracleDatabaseConnection()
    {
        
    }

    public class OracleDatabaseConnectionFactory : IOracleDatabaseConnectionFactory
    {
        public OracleDatabaseConnection CreateConnection(string connectionString)
        {
            return new OracleDatabaseConnection(connectionString);
        }
    }
    public interface IOracleDatabaseConnectionFactory
    {
        OracleDatabaseConnection CreateConnection(string connectionString);
    }
}
```

Evet inner class'lar code smell olduğu düşünülebilir ancak böyle bir şart konulmuş ise ve koşullar bu şekildeyse bu implementasyon pratik olmakla birlikte isterleri karşılamaktadır.

# Ek

Factory Method ile Abstract Factory arasındaki en büyük fark abstract factory'de birden çok factory ile polymorphism sağlanarak open-closed prensibini uygulanması sağlanıyor. Factory Method'da ise tek bir factory'den ayrı methodlarla aynı interface'den türeyen sınıfların oluşturulmasını sağlıyor.

---

Constructoruna parametre alan class'larda çok işe yarayan bu teknik hem developera bir sınıfı nasıl oluşturup kullanabileceği konusunda destek sağlıyor hem de eğer DI kullanırsak kodumuzu loosely coupled a çekiyor ve test edilebilir bir şekilde sınıflarımızı construct etmiş ve çağırmış oluyoruz.

# Kaynakça

https://www.youtube.com/watch?v=EcFVTgRHJLM

https://www.youtube.com/watch?v=l0rzdmnsyZ0

https://refactoring.guru/design-patterns/factory-method

https://refactoring.guru/design-patterns/abstract-factory

https://www.yasinatilkan.com/factory-method-tasarim-deseninin-kullanimi/

https://www.patreon.com/techworld_with_milan/shop/design-patterns-in-use-e-book-312304

https://www.davidguida.net/di-friendly-factory-pattern/

https://chaitanyasuvarna.wordpress.com/2021/03/21/factory-pattern-di-in-net-core/

https://www.ledjonbehluli.com/posts/wash-tunnel/simple_factory/

https://www.tutorialspoint.com/design_pattern/abstract_factory_pattern.htm

https://medium.com/kodcular/factory-design-pattern-nedir-2d656bd7f429