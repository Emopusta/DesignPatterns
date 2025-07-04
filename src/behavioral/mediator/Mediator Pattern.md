Mediator pattern, nesneler arasındaki doğrudan bağımlılıkları ortadan kaldırarak, iletişimlerini merkezi bir arabulucu üzerinden gerçekleştirmelerini sağlar. Bu sayede nesneler birbirlerine bağımlı olmadan etkileşime geçebilir ve sistem daha esnek, modüler ve kolay yönetilebilir hale gelir.

Bu durum, nesneler birbirlerine referans verilmediği için bir message'ın hangi nesneye gönderildiğini bulmakta zorluklar yaratmaktadır. Ancak dependency'leri ciddi oranda azalttığı için bu yöntem çeşitli alanlarda tercih edilmektedir.

Mediator türkçesiyle arabulucu, tam anlamıyla nesneler arası arabuluculuk yaparak yönlendirmeler sağlar ve birbirlerinin varlığından bile haberdar olmayan nesneler kendi aralarında haberleşebilecek şekilde onları yönlendirir.

Bu yazıda iki adet örnek üzerinden anlatım yapılacaktır 
1. Bir "Chat Room" uygulaması üzerinden kullanıcılar kendi aralarında veya ortak bir alanda mesajlaşması sağlanacaktır. 
2. Backend uygulamalarda sıkça karşılaşılan API/Controller development sırasında Business Logic ile Controller'ların aynı yerde yazılmaması gerektiğinden, tamamen dependency'lerinden arındırılmış bir implementasyon yapılacaktır.

- [Chat Room](#chat-room)
- [API ve Business Logic Mediator ile Dependency Azaltımı](#api-ve-business-logic-mediator-ile-dependency-azaltımı)

### Chat Room 

Bu örnekte oluşturulan bir chat room class'ı mediator görevi görerek içerisine kaydolan `User`'ların mesajlarını birbirleri arasında iletimini veya bir `User` ın diğer bütün `User`'lara mesajı Broadcast (yayınlamak) yapmasını sağlıyor. Bu `ChatRoom` mediator sınıfı sayesinde kullanıcılar hiçbir şekilde birbirleriyle strictly coupled (sıkı bağlı) bir ilişki kurmadan mediator sınıfı üzerinden haberleşiyor.

ChatRoom.cs
```
public class ChatRoom
{
    private readonly Dictionary<string, User> _users = new();

    public void Register(User user)
    {
        if (!_users.ContainsKey(user.Name))
        {
            _users[user.Name] = user;
            user.SetChatRoom(this);
        }
    }

    public void SendMessage(string from, string to, string message)
    {
        if (_users.TryGetValue(to, out var user))
        {
            user.Receive(from, message);
        }
        else
        {
            Console.WriteLine($"User '{to}' not found in chat room.");
        }
    }

    public void BroadcastMessage(string from, string message)
    {
        foreach (var user in _users.Values)
        {
            if (user.Name != from)
            {
                user.Receive(from, message);
            }
        }
    }
}
```

Yukarıdaki `ChatRoom` mediator class'ında görüldüğü üzere `User`'ların saklandığı bir `Dictionary` veri tipi, `User`'ları bu property'ye kayıt eden `Register` methodu, `User`'lar arası özel mesaj ve tüm `User`'lara broadcast yapılmasını sağlayacak olan methodlar bulunmaktadır.

User.cs
```
public class User
{
    public string Name { get; }
    private ChatRoom? _chatRoom;

    public User(string name)
    {
        Name = name;
    }

    public void SetChatRoom(ChatRoom chatRoom)
    {
        _chatRoom = chatRoom;
    }

    public void Send(string to, string message)
    {
        if (to.Equals("Broadcast", StringComparison.OrdinalIgnoreCase))
        {
            Console.WriteLine($"{Name} broadcasts message: {message}");
            _chatRoom?.BroadcastMessage(Name, message);
            return;
        }
        Console.WriteLine($"{Name} sends to {to}: {message}");
        _chatRoom?.SendMessage(Name, to, message);
    }

    public void Receive(string from, string message)
    {
        Console.WriteLine($"{Name} receives from {from}: {message}");
    }
}
```

Yukarıda ise `User` class'ı tanımlanmıştır. Bu class'ın içerisinde mesajın gönderilmesi ve alınması için `Send` ve `Receive` methodlarının yanı sıra `User`'ın `ChatRoom`'a kayıt olmasını sağlayacak işlemler sağlanmaktadır.

Burada iyileştirmeler ve ek feature'lar eklenebilir. Örneğin ``Chat`` geçmişi (ChatLog) ve benzeri işlemler yapılabilir.

Aşağıda program.cs dosyasında yapılan örnek case'de (durum), bir chatRoom oluşturulur ardından bu chatroom'a kayıt olacak kullanıcılar oluşturulur. Oluşturulan chatRoom'a bu kullanıcılar `Register` edilir. Ardından `User`lar kendileri arasında mesajlaşma yapar ve en sonda bir `User` broadcast yapar.

Program.cs
```
internal class Program
{
    private static void Main(string[] args)
    {
        var chatRoom = new ChatRoom();

        var emre = new User("Emre");
        var semih = new User("Semih");
        var emopusta = new User("Emopusta");

        chatRoom.Register(emre);
        chatRoom.Register(semih);
        chatRoom.Register(emopusta);

        emre.Send("Semih", "Selam Semih!");
        semih.Send("Emopusta", "Emopusta nasıl bir nickname?");
        emopusta.Send("Semih", "Gayet iyi bir nickname.");
        emopusta.Send("Broadcast", "Benim nickname'im çok iyidir.");
    }
}
```

Çıktı
```
Emre sends to Semih: Selam Semih!
Semih receives from Emre: Selam Semih!
Semih sends to Emopusta: Emopusta nasıl bir nickname?
Emopusta receives from Semih: Emopusta nasıl bir nickname?
Emopusta sends to Semih: Gayet iyi bir nickname.
Semih receives from Emopusta: Gayet iyi bir nickname.
Emopusta broadcasts message: Benim nickname'im çok iyidir.
Emre receives from Emopusta: Benim nickname'im çok iyidir.
Semih receives from Emopusta: Benim nickname'im çok iyidir.
```


### API ve Business Logic Mediator ile Dependency Azaltımı

Bu örnekte bire-bir mesaj iletimi ile business logic'lerin API'lardan ayrılmasını sağlanacaktır. Aşağıda kullanılacak mediator class'ın sadece Send methodundan oluştuğunu ve içerisine gelen `Request` `Response` çiftiyle `Handler` class'ının aranıp bulunması ve içerisinde default bulunan Handle methodunun gerekli parametreler ile çalıştırılmasını sağlamaktadır.

```
public interface IMediator
{
    TResponse Send<TResponse>(IRequest<TResponse> request);
}

public class APIBusinessMediator : IMediator
{
    private readonly IServiceProvider _serviceProvider;
    public APIBusinessMediator(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    public TResponse Send<TResponse>(IRequest<TResponse> request)
    {
        var handlerType = typeof(IRequestHandler<,>).MakeGenericType(request.GetType(), typeof(TResponse));
        dynamic handler = _serviceProvider.GetService(handlerType);
        var handleMethod = handlerType.GetMethod("Handle");
        return (TResponse)handleMethod.Invoke(handler, new object[] { request });
    }
}
```

Aşağıda oluşturulup mediator ile arabuluculuğun sağlanacağı sınıfların interfaceleri implement edilmiştir. Handle methodu mediator içerisinde static tanımlandığından kaynaklı interface ile ekstra kural ile sınırlandırılmıştır. 

```
public interface IRequest<out TResponse>
{
}

public interface IRequestHandler<in TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    TResponse Handle(TRequest request);
}
```

```
public class CreateProductCommand : IRequest<CreateProductResponse>
{
    public string Name { get; set; }
    public decimal Price { get; set; }
    public CreateProductCommand(string name, decimal price)
    {
        Name = name;
        Price = price;
    }
}

public class CreateProductResponse
{
    public string Name { get; set; }
    public decimal Price { get; set; }
    public CreateProductResponse(string name, decimal price)
    {
        Name = name;
        Price = price;
    }
}
```

Oluşturulacak mesajın içerisindeki argument'lar ile `Handle` sonucunda dönülecek response yukarıdaki şekilde oluşturulmuştur.

Ardından mesajın gönderileceği handler sınıfı aşağıdaki gibi oluşturulmuştur.

```
public class CreateProductCommandHandler : IRequestHandler<CreateProductCommand, CreateProductResponse>
{
    private readonly IMediator _mediator;
    public CreateProductCommandHandler(IMediator mediator)
    {
        _mediator = mediator;
    }
    public CreateProductResponse Handle(CreateProductCommand request)
    {

        Console.WriteLine($"Creating product with given Name: {request.Name}, and Price: {request.Price}");

        return new CreateProductResponse(request.Name, request.Price);
    }
}
```

Gerçekçi bir ortam kurulması için container oluşturularak Dependency Injection implemente edilmiştir.

```
internal class Program
{
    private static void Main(string[] args)
    {
        ServiceProvider serviceProvider = new ServiceCollection()
                                    .AddSingleton<IMediator, APIBusinessMediator>()
                                    .AddScoped<IRequestHandler<CreateProductCommand, CreateProductResponse>, CreateProductCommandHandler>()
                                    .BuildServiceProvider();

        var mediator = serviceProvider.GetService<IMediator>();
        var response = mediator.Send(new CreateProductCommand("Laptop", 1500.00m));
        var response2 = mediator.Send(new CreateProductCommand("Smartphone", 800.00m));
    }
}
```

Yukarıda görüldüğü üzere servisler container'a register ediliyor ve ardından WebAPI üzerinde oluşturulacak requestler ve responseların örnekleri gösteriliyor.

Çıktı
```
Creating product with given Name: Laptop, and Price: 1500,00
Creating product with given Name: Smartphone, and Price: 800,00
```

Yukarıda console uygulaması üzerinden örnekler gerçekleştirildiğinden WebAPI uygulaması için ufak değişiklikler yapılması gerekmektedir. Bunlar servislerinizin program.cs içerisinde register edildikten sonra oluşturulacak Controller'lar içerisine `IMediator` inject edilip yukarıdaki örnekteki gibi kullanılırsa API katmanı ile Business Logic arasındaki bağlantı ciddi oranda loose olmuş olur. 

MediatR kütüphanesi ile bu işlemler basit bir şekilde yapılabilmektedir ve ek olarak bire-bir mesajlaşma dışında bire-çok gibi çeşitli mesajlaşmalar bulunmaktadır.

# Kaynakça

Ücretsiz:
https://refactoring.guru/design-patterns/mediator
https://www.gencayyildiz.com/blog/c-mediator-design-patternmediator-tasarim-deseni/
https://medium.com/bili%C5%9Fim-hareketi/mediator-tasar%C4%B1m-kal%C4%B1b%C4%B1-881ee987ea72
https://medium.com/kodcular/mediator-design-pattern-nedir-20585adf0e98
https://www.geeksforgeeks.org/system-design/mediator-design-pattern/
https://www.youtube.com/watch?v=B32xXS9YE7c
https://www.youtube.com/watch?v=5tyRwBWGjQk

Ücretli:
https://www.udemy.com/course/design-patterns-csharp-dotnet/