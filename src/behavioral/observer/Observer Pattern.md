**Genel Tanım**: Bir nesne, kendi içerisinde bir state'e (durum) sahip olması ve bu state'de oluşan değişikliklere bağlı olarak diğer nesnelere bildirecek bir mekanizmaya sahip olmasına subject/publisher denir. Bu nesnedeki state değişimi bu nesneyi dışarıdan izleyen (observe eden) nesnelere etki eder ve onlara haber verir, bu nesnelere ise observer/subscriber denir.   

> (Observer == Subscriber && Subject == Publisher) Ek olarak pub/sub şeklinde araştırılabilir. Pattern'in ismi ile anlatımın uyumlu olması için Observer ve Subject anahtar kelimeleri ile devam edilecektir.


Observer pattern, subject classların bazı observer class'lar ile observe edilmesini ve buna bağlı olarak, subject class'larda oluşturulacak bir event ile bütün observer'lara notification (bildirim) göndermesi ile tanımlanır. Observer class'lar bir subject içerisinde attach/subscribe veya detach/unsubscribe edilir. Böylece Subscribe olan observer nesnesinin, subject state'inde meydana gelecek değişiklikleri izlediği anlaşılır ve Unsubscribe olan observer nesnesinin, subject state'inde gelecek değişiklikleri izlemediği anlaşılır.

### Örnek Senaryo
Bir newsletter (e-posta haber bülteni) uygulamasına çeşitli kullanıcılar kayıt olur. Bu kullanıcılar her biri subject'e observe/attach veya daha sonrasında ignore/detach edebilir. Ayrıca kullanıcılara gönderilecek her mail'in başında kullanıcının kendi isminin bulunmasını yani multi-tenant bir yapı kullanılması hedeflenmektedir.

# Default Implementation

```csharp
public interface IObserver
{
    void Update(string message);
}

public interface ISubject
{
    void Attach(IObserver observer);
    void Detach(IObserver observer);
    void Notify();
}
```

Observer class'ların ve Subject class'ların implement edeceği interfaceler yukarıdaki gibidir. `IObserver` içerisindeki Update methodu, ISubject içerisindeki Notify methodu ile Subject'e Attach/Subscribe olmuş bütün observer'larda tetiklenecek olan methoddur. `ISubject` içerisindeki Attach ve Detach methodları observer'ları kendi içerisinde saklama ve silme işlemlerini gerçekleştirecek olup Notify methodu ise kayıtlı tüm observerların update methodunu triggerlamak için implemente edilecektir.

```csharp
public class MailSendingSubject : ISubject
{
    private HashSet<IObserver> _observers = new HashSet<IObserver>();
    private string _state;
    public string State
    {
        get { return _state; }
        set
        {
            _state = value;
            Notify();
            Console.WriteLine("********");
        }
    }
    public void Attach(IObserver observer)
    {
        _observers.Add(observer);
    }
    public void Detach(IObserver observer)
    {
        _observers.Remove(observer);
    }
    public void Notify()
    {
        foreach (var observer in _observers)
        {
            observer.Update(_state);
        }
    }
}
```

Yukarıda bahsedildiği gibi implementasyon gerçekleştirilmiştir. Ek olarak `State` property'si herhangi bir manipulasyon/değişiklik işlemine maruz kaldığında bütün observer'ların Notify edilmesi için ek bir implementasyon yapılmıştır.

İçine gönderilecek mesajın bütün bu Subjecte bağlı olanlarda aynı olacağı düşünülerek aşağıdaki gibi sadece  `name` kısmı observera bağlı olacak şekilde oluşturulmuştur.

```csharp
public class MailSenderObserver : IObserver
{
    private string _name;
    public MailSenderObserver(string name)
    {
        _name = name;
    }
    public void Update(string message)
    {
        Console.WriteLine($"Hello {_name}, {message}");
    }
}
```

Yukarıdaki implementasyonların konsol uygulamasında örneği:

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        var subject = new MailSendingSubject();
        var observer1 = new MailSenderObserver("Emre");
        var observer2 = new MailSenderObserver("Semih");
        var observer3 = new MailSenderObserver("Emopusta");
        subject.Attach(observer1);
        subject.Attach(observer2);
        subject.Attach(observer3);
        subject.State = "Tebrikler, şöyle böyle indirimimizden yararlanabilirsiniz aşağıdaki linke tıklayın";
        subject.State = "Naber?";
        subject.Detach(observer1);
        subject.State = "Teşekkürler beni mailinizde unsubscribe yapmadığınız için.";
    }
}
```

Çıktı olarak ise:

```
Hello Emre, Tebrikler, şöyle böyle indirimimizden yararlanabilirsiniz aşağıdaki linke tıklayın
Hello Semih, Tebrikler, şöyle böyle indirimimizden yararlanabilirsiniz aşağıdaki linke tıklayın
Hello Emopusta, Tebrikler, şöyle böyle indirimimizden yararlanabilirsiniz aşağıdaki linke tıklayın
********
Hello Emre, Naber?
Hello Semih, Naber?
Hello Emopusta, Naber?
********
Hello Semih, Teşekkürler beni mailinizde unsubscribe yapmadığınız için.
Hello Emopusta, Teşekkürler beni mailinizde unsubscribe yapmadığınız için.
********
```

Bu pattern kullanılarak bir çok ihtiyaç karşılanabilir ve nihai implementasyon yukarıdaki gibi değildir. İhtiyaçlara göre değişiklik gösterebilir. Örneğin her Notify işleminden sonra observer listesi temizlenebilir veya Yukarıdaki gibi sadece mail değil buna ek olarak bir de başka bir platformdan mesaj gönderimi yapılabilir... Böylece farklı ihtiyaçlara ekstra implementasyonlar yapılması gerekir. 

Genel olarak özetlenmesi gerekirse Observer Pattern, observe edenlerin sürekli olarak bir nesneyi izleyip ona sürekli state değiştirdin mi diye sormadan (Poll), state değiştirdiği zaman bütün observer'ların subject tarafından notify edilmesi şeklinde çalışır (Push). 

# Kaynakça

- https://www.youtube.com/watch?v=_BpmfnqjgzQ
- https://refactoring.guru/design-patterns/observer
- https://www.tutorialspoint.com/design_pattern/observer_pattern.htm
- https://medium.com/kodcular/observer-design-pattern-nedir-671f61969c91
- https://www.geeksforgeeks.org/observer-pattern-set-1-introduction/
- https://medium.com/i%CC%87yi-programlama/observer-g%C3%B6zlemci-design-pattern-535df620b720