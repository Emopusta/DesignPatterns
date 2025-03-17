Tanım: Decouple Abstraction from its Implementation.

Bridge Pattern, bir implementasyonu iki soyut class'a ayırarak hem implementasyonu Open-Closed Principle doğrultusunda enine genişletilebilmesini sağlamakta hem de her koşulun sağlanması için gereken gereksiz kodlamalardan kaçınılmasını sağlar. Sağladığı etkilerden biri ise plugin implementasyonlarında kullanılabilmektedir. Böylece source code daha ``Orthononal`` olur.

Örnek: ErrorConsoleLogger isimli uygulamadaki oluşan hataları loglayan bir class ve bu class logları konsola yazmaktadır:

```

public class ErrorConsoleLogger
{
    public void Log(string message)
    {
        Console.WriteLine($"Error: {message}");
    }
}

```

Aynı şekilde yeni class'lar eklenmesi durumunda her Error Log'u için bütün loglama tiplerini ayrı ayrı class'lara yazılıp implemente edilmesi gerekmektedir.

Örneğin `Error`, `Info`, `Debug` tiplerinde Loggerlar oluşturuldu. Bunlara da loglama tipleri olarak `Console`, `File`, ``Cloud`` oluşturuldu. Tahmin edilebileceği üzere her logger için bütün tiplerin implementasyonunu yapmamız gerektiği worst case senaryoda:
- ErrorConsoleLogger
- ErrorFileLogger
- ErrorCloudLogger
- InfoConsoleLogger
- InfoFileLogger
- InfoCloudLogger
- DebugConsoleLogger
- DebugFileLogger
- DebugCloudLogger

<!-- TODO : Buraya iki farklı uml class diagramı şeklinde görsel eklenebilir Bridge pattern ile ve patternsiz örnek oluşturulması açısından. -->

Şeklinde tam 3\*3 = 9 adet class tanımlanması gerekmektedir ve implementasyonel olarak DRY prensibine uyulmamaktadır çünkü her ``Console`` tip implementasyonu bütün `Logger` lar içerisinde aynı implementasyona sahip olmaktadır. Ek olarak da yeni bir tip eklenmesi gerektiğinde en az Logger class miktarı kadar implementasyon yapılması gerekmektedir.

Bahsedilen kod tekrarları ve bağımlılıkların source code'dan arındırılması için Bridge Pattern kullanılmaktadır. Bridge Pattern bu ``Logger`` class'lar ile `Loglama Tipleri` class'larını birbirinden kısmi bir şekilde ayırarak daha temiz bir implementasyon sunmaktadır.

Öncelikle bu iki birbirinden ayrılabileceği öngörülen tiplere Abstraction ve Implementor şeklinde base class ve interface implementasyonları yapılıyor. Abstraction implementasyonu kullanılacak sistemin servisi olacak şekilde oluşturulmalı ve Implementor ise servisin kullanacağı işlemleri sağlayacak yapıyı içermesi gerekmektedir.

Yukarıdaki Logger örneğinden yola çıkarak `Error`, `Info`, `Debug` class'ları developerın kullanacağı servis görevi görecek class'lar yani Abstraction'un uygulanması gereken class'lar ve bu abstraction ile imzalanan class'lar ise ConcreteAbstract olarak tanımlanacaktır. Bu class'ların içerisinde loglama işlemlerinin hangi tipte (ör. Console) olacağına karar verecek olan implementor class'lar olarak tanımlanmaktadır.

LoggerAbstraction.cs
```
public abstract class LoggerAbstraction
{
    protected ILoggerImplementor _implementor;

    public LoggerAbstraction(ILoggerImplementor writer)
    {
        _implementor = writer;
    }

    public abstract void Log(string message);
}
```

ILoggerImplementor.cs
```
public interface ILoggerImplementor
{
    void WriteLog(string message);
}
```

Yukarıdaki kodlardan görüldüğü üzere Implementor interface'inden imzalanacak implementor class'lar bütün loglama tipi class'larda olacak olan `WriteLog` fonksiyonuna sahip olacak ve Abstraction class'ı ise bu implementoru constructor'ı veya başka yöntemlerle içine alıp kullanılabilecek halde bulundurması gerekmektedir. ``Abstraction`` implementasyonu farklı şekillerde de kodlanabilir. Burada böyle kodlanmasının sebebi Logger class'larının kendi Log fonksiyonlarını zorla implemente etmelerini sağlamak.

`Abstraction` ve `Implementor`'ler ayrıldığına ve base abstract class ve interface şeklinde yazıldığına göre tipler ve servisler implemente edilebilir.

ErrorLogger.cs
```
public class ErrorLogger : LoggerAbstraction
{
    public ErrorLogger(ILoggerImplementor implementor) : base(implementor) { }

    public override void Log(string message)
    {
        _implementor.WriteLog($"Error: {message}");
    }
}
```

Constructor'da parent'ına implementoru gönderdi. Implementor class'ı base abstract içerisinde setlendi ve böylece ``Log`` methodu içerisinde \_implementor olarak çağırılıp setlenen implementor class'a göre Error message basılacaktır.

ConsoleLog.cs
```
public class ConsoleLog : ILoggerImplementor       
{
    public void WriteLog(string message)
    {
        Console.WriteLine($"Writing to Console: {message}");
    }
}
```

ConsoleLog implementor'u verildiğinde yapılacak işlem.

FileLog.cs
```
public class FileLog : ILoggerImplementor
{
    public void WriteLog(string message)
    {
        Console.WriteLine($"Writing to File: {message}");
    }
}
```

FileLog implementor'u verildiğinde yapılacak işlem.

Bunların kullanım örnekleri ise örnek konsol uygulamasında aşağıdaki gibi gösterilmektedir:

```
namespace Bridge;

internal class Program
{
    static void Main(string[] args)
{
    var consoleLogger = new ErrorLogger(new ConsoleLog());
    var fileLogger = new ErrorLogger(new FileLog());

    consoleLogger.Log("Console log entry");
    fileLogger.Log("File log entry");
}
}

```

Çıktı olarak:

```
Writing to Console: Error: Console log entry
Writing to File: Error: File log entry
```

Böylece en baştaki gibi 9 adet farklı class'a benzer kodların implemente edilmesi yerine aynı durum karşılanırsa 6 adet (`ErrorLogger, InfoLogger, DebugLogger` ve `ConsoleLog, FileLog, CloudLog`) class ile hem daha temiz hem de daha az miktarda kod ile implemente edilmektedir.

# Bonus Generic Implementation

Bu ConcreteAbstract class'ları generic bir yapıyla daha temiz bir şekilde servis olarak kullanabilir miyiz?

Aşağıdaki implementasyon bu ihtiyacı karşılamaktadır.

LoggerAbstraction.cs
```
public abstract class LoggerAbstraction<T> where T : ILoggerImplementor
{
    protected ILoggerImplementor _implementor;

    public LoggerAbstraction(IServiceProvider serviceProvider)
    {
        _implementor = (ILoggerImplementor)serviceProvider.GetService(typeof(T));
    }

    public abstract void Log(string message);
}
```
Burada test edilebilmesi örneğinin konsol uygulamasında canlandırılabilmesi için ServiceProvider ile implementasyonu gözlenmektedir. Buna ek olarak aynı implementasyonu Activator ile nesne oluşturularak da yapılabilir:

```
_implementor = (ILoggerImplementor)Activator.CreateInstance(typeof(T));
```


ErrorLogger.cs
```
public class ErrorLogger<T> : LoggerAbstraction<T> where T : ILoggerImplementor
{
    public ErrorLogger(IServiceProvider serviceProvider) : base(serviceProvider)
    {
    }

    public override void Log(string message)
    {
        _implementor.WriteLog($"Error: {message}");
    }
}
```

GenericBase class ile oluşturulan `ErrorLogger` yukarıdaki şekilde oluşturulabilir ve böylece aşağıdaki gibi kullanım sağlanabilir.

```
namespace Bridge;

internal class Program
{
    static void Main(string[] args)
    {
        ServiceProvider serviceProvider = new ServiceCollection()
                                    .AddScoped<ConsoleLog>()
                                    .AddScoped<FileLog>()
                                    .BuildServiceProvider();

        var consoleLoggerGeneric = new ErrorLogger<FileLog>(serviceProvider);
        var fileLoggerGeneric = new ErrorLogger<FileLog>(serviceProvider);
        consoleLoggerGeneric.Log("console log entry generic");
        fileLoggerGeneric.Log("File log entry generic");
    }
}
```

Bu kullanım daha da iyileştirilebilir.

# Kaynakça

- https://refactoring.guru/design-patterns/bridge
- https://www.geeksforgeeks.org/bridge-design-pattern/
- https://medium.com/design-patterns/design-patterns-7-bridge-7c5c74801c70
- https://www.tutorialspoint.com/design_pattern/bridge_pattern.htm
- https://www.youtube.com/watch?v=xhhZzx2SD70
- https://www.youtube.com/watch?v=F1YQ7YRjttI
- https://www.youtube.com/watch?v=9jIgSsIfh_8
- https://www.youtube.com/watch?v=Q3R0zZfXit0
- https://www.youtube.com/watch?v=88kAIisOiYs