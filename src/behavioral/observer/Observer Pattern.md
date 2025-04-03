**Genel Tanım**: Bir nesne, kendi içerisinde bir duruma (state) sahip olması ve bu state'deki değişikliklere bağlı olarak diğer nesnelere bildirecek bir mekanizmaya sahip olduğu için subject/publisher olarak adlandırılır. Bu nesnedeki state değişimi bu nesneyi dışarıdan izleyen (observe eden) nesnelere etki eder ve onlara haber verir, bu nesnelere ise observer/subscriber denir.   

> (Observer == Subscriber && Subject == Publisher) Ek olarak pub/sub şeklinde araştırılabilir. Pattern'in ismi ile anlatımın uyumlu olması için Observer ve Subject anahtar kelimeleri ile devam edilecektir.


Observer pattern, subject classlara observe olunmasını ve ardından subject class'larda oluşturulacak bir event ile bütün observer'lara notification (bildirim) göndermesi ile tanımlanır. Observer class'lar bir subject içerisinde attach/subscribe veya detach/unsubscribe edilir.

Örnek: Bir e-mail newsletter uygulamasına kayıt olan çeşitli kullanıcılar bulunmaktadır. Bu kullanıcıların her biri subject'e observe/attach veya daha sonrasında ignore/detach etmek isteyebilir. Ayrıca kullanıcılara gönderilecek her mail'in başında kullanıcının kendi isminin bulunmasını yani multi-tenant bir yapı kullanılması hedeflenmektedir.

# Default Implementation

```
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

```
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

```
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

```
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

Genel olarak özetlenmesi gerekirse Observer Pattern, observe edenlerin sürekli olarak bir nesneyi takip edip ona sürekli state değiştirdin mi diye sormadan (Poll), state değiştirdiği zaman bütün observer'ların subject tarafından notify edilmesi şeklinde çalışır (Push). 

# Kaynakça

- https://www.youtube.com/watch?v=_BpmfnqjgzQ
- https://refactoring.guru/design-patterns/observer
- https://www.tutorialspoint.com/design_pattern/observer_pattern.htm
- https://medium.com/kodcular/observer-design-pattern-nedir-671f61969c91
- https://www.geeksforgeeks.org/observer-pattern-set-1-introduction/
- https://medium.com/i%CC%87yi-programlama/observer-g%C3%B6zlemci-design-pattern-535df620b720