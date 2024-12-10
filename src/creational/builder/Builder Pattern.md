Builder Design Pattern, çok fazla property'si bulunan nesnelerin dolayısıyla bu property'lerin her zaman kullanılmadığı veya az sayıda constructor ile oluşturmanın yetersiz kaldığı durumlarda kullanılmaktadır. Özetle constructor çeşitliliğini azaltma amaçlı olarak ortaya çıkmış bir pattern'dir.

Bu pattern'e, unit testlerde, string işlemlerinde, kompleks nesnelerin oluşturulmasında, webAPI projelerinde servislerimizi register ettiğimiz, çeşitli konfigurasyonlar yaptığımız, uygulamamızı build ve run ettiğimiz program.cs dosyaları gibi yerlerde sıkça karşılaşılmaktadır. 

Unit Testlerde, test edilen class'tan döndürülen çıktı ile Builder pattern ile spesifik olarak oluşturulan nesnenin Assert bölümünde elde edilen çıktı ile karşılaştırılmasında [kullanılmaktadır.](https://ardalis.com/improve-tests-with-the-builder-pattern-for-test-data/)

Builder pattern kendi içerisinde farklı farklı implementasyonlara ayrılmaktadır:
- [Fluent Builder](#fluent-builder)
- [Inherited Builder](#inherited-builder)
- [Stepwise Builder](#stepwise-builder) 
- [Functional Builder](#functional-builder)
- [Faceted Builder](#faceted-builder)

---
# Fluent Builder 

Fluent Builder, bir builder class'ın fonksiyonunun kendi type'ını (tip) return (döndürmek) ederek farklı satırların yanı sıra aynı satırda üst üste işlemler yapılabilmesini sağlar.

örnek:
```
public class Product
{
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }

    public override string ToString()
    {
        return $"Name: {Name}, Description: {Description}, Price: {Price}";
    }
}
```
Yukarıda oluşturulan Product sınıfımızı öncelikle klasik Builder Pattern ile oluşturalım.
```
public class ProductBuilder
{
    private readonly Product _product = new Product();
    public void WithName(string name)
    {
        _product.Name = name;
    }
    public void WithDescription(string description)
    {
        _product.Description = description;
    }
    public void WithPrice(decimal price)
    {
        _product.Price = price;
    }
    public Product Build()
    {
        return _product;
    }
}
```
Görüldüğü üzere, `ProductBuilder` class'ı çeşitli fonksiyonlar ile içerisinde oluşturulan `Product` nesnesinin property'lerinin doldurulmasına yardımcı oluyor. Buralardaki amaç çeşitli koşullar veya validation'lar eklenerek oluşturulmayı sınırlandırmak ve en sonunda `Build()` fonksiyonu ile yine ihtiyaca bağlı olarak çeşitli koşullar ile `Product` nesnesinin oluşturulmasını sağlıyor.

```
internal class Program
{
    static void Main(string[] args)
    {
        var productBuiltWithBuilder = new ProductBuilder();
        
        productBuiltWithBuilder.WithName("Laptop");
        productBuiltWithBuilder.WithDescription("Dell Laptop");
        productBuiltWithBuilder.WithPrice(1000);
		
        Console.WriteLine(productBuiltWithBuilder.Build());
    }
}
```

`FluentBuilder` ise bahsedildiği gibi yukarıdaki her bir method çağırımında farklı satırlarda builder değişkeni ile çağırılmasına ihtiyaç kalmadan tek satırda üst üste kullanılabilmektedir.

```
public class ProductFluentBuilder
{
    private readonly Product _product = new Product();
    public ProductFluentBuilder WithName(string name)
    {
        _product.Name = name;
        return this;
    }
    public ProductFluentBuilder WithDescription(string description)
    {
        _product.Description = description;
        return this;
    }
    public ProductFluentBuilder WithPrice(decimal price)
    {
        _product.Price = price;
        return this;
    }
    public Product Build()
    {
        return _product;
    }
}
```
```
internal class Program
{
    static void Main(string[] args)
    {
        var builder = new ProductFluentBuilder();

        var productBuiltWithFluent = builder.WithName("Laptop")
                       .WithDescription("Dell Laptop")
                       .WithPrice(1000)
                       .Build();

        Console.WriteLine(productBuiltWithFluent);
    }
}
```

Yukarıdaki kod bloğunda, ilgili Builder class'ın `Build()` fonksiyonu hariç bütün fonksiyonları Builder class'ının kendisini döndürmektedir. Böylece `Main` içerisinde görüldüğü üzere builder her fonksiyon çağırımından sonra kendisini döndürdüğü için üzerine tekrardan kendi fonksiyonunu çağırabilir durumda oluyor.

---
# Inherited Builder

Inheritance ile oluşturulan builderlar'da karşılaşılan sıkıntı ise `child` bir builder `parent`'daki fonksiyonu kullandığı zaman FluentBuilder'daki gibi tek satırda tüm fonksiyonların kullanımına devam edemiyor çünkü builder'ın tipi anlık olarak parent tipine dönüşüyor.

Aşağıdaki örnekte görüldüğü üzere en tepede bulunan ve ``Build()`` işlemini yapan `ProductBuilder` class'ının altında birbirinden türeyen sırasıyla Name, Description ve Price için ayrı Builder class'lar oluşturulmuş ve inheritence uygulanmıştır. 

```
public abstract class ProductBuilderBase
{
    protected readonly Product _product = new Product();
    public Product Build()
    {
        return _product;
    }
}

public class ProductNameBuilder : ProductBuilderBase
{
    public ProductNameBuilder WithName(string name)
    {
        _product.Name = name;
        return this;
    }
}

public class ProductDescriptionBuilder : ProductNameBuilder
{
    public ProductDescriptionBuilder WithDescription(string description)
    {
        _product.Description = description;
        return this;
    }
}

public class ProductPriceBuilder : ProductDescriptionBuilder
{
    public ProductPriceBuilder WithPrice(decimal price)
    {
        _product.Price = price;
        return this;
    }
}
```

Bu sınıfların kullanımı aşağıdaki gibi olduğu durumlarda hata alınan kısımlar `Hata verir` yorum satırıyla belirtilmiştir. Bu satırlarda hata alınmasının sebebi ilk `product` build edilmesi anında `.WithName("Laptop")` fonksiyonunun çağırılması sonucunda `builder.WithName("Laptop")` işleminin tipi `ProductBuilder` olarak değişti ve bundan kaynaklı olarak bütün child class'lardaki fonksiyonlara erişimi iptal oldu. Aynı durum `product2` için de geçerlidir ancak onda ikinici işlemde hata vermesinin sebebi ise `.WithDescription("Dell Laptop")` ile işlemin tipi `ProductDescriptionBuilder`'a dönüştü ancak ``ProductNameBuilder`` bu class'ın parent'ı olduğu için `WithName` fonksiyonuna erişebildi ancak `WithPrice` fonksiyonu `child` olduğu için için erişmek mümkün olamadı.

```
internal class Program
{
    static void Main(string[] args)
    {
        var builder = new ProductPriceBuilder();
        var product = builder.WithName("Laptop")
                       .WithDescription("Dell Laptop") // Hata verir
                       .WithPrice(1000)
                       .Build();

        var product2 = builder.WithDescription("Dell Laptop")
                       .WithName("Laptop")
                       .WithPrice(1000) // Hata verir
                       .Build();
    }
}
```

Bu duruma çözüm olarak ``Recursive Generic`` tekniği kullanarak en üstteki base build class hariç herhangi bir fonksiyon kullanılsa bile en dipteki child class'a göre bir return yaptığı için bütün fonksiyonlar kullanılabilmektedir. En tepedeki base build class'ın bu durumu karşılamamasının sebebi ise `Build()` operasyonunun bulunduğu sınıf olması yani `Build()` edildikten sonra çıktımızın bir builder class değil build ettiğimiz nesne olarak çıkması beklenmektedir.

```
public class ProductNameBuilder<T> : ProductBuilderBase where T : ProductNameBuilder<T>
{
    public T WithName(string name)
    {
        _product.Name = name;
        return (T)this;
    }
}

public class ProductDescriptionBuilder<T> : ProductNameBuilder<ProductDescriptionBuilder<T>>
    where T : ProductDescriptionBuilder<T>
{
    public T WithDescription(string description)
    {
        _product.Description = description;
        return (T)this;
    }
}

public class ProductPriceBuilder<T> : ProductDescriptionBuilder<ProductPriceBuilder<T>>
    where T : ProductPriceBuilder<T>
{
    public T WithPrice(decimal price)
    {
        _product.Price = price;
        return (T)this;
    }
}
```

Bu dönüşümden sonra çok temiz olmayan bir çözüm ile Product class içerisinde bir değişikliğe gidilmesi gerekmektedir.

```
public class Product
{
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }

    public class Builder : ProductPriceBuilder<Builder>
    {
        internal Builder() { }
    }
    public static Builder InnerBuilder => new Builder();
    public override string ToString()
    {
        return $"Name: {Name}, Description: {Description}, Price: {Price}";
    }
}
```

Burada ``inner`` bir class oluşturuldu ve ``Product`` içerisine ``static`` bir şekilde erişilebilmesi için bir property oluşturuldu.

```
internal class Program
{
    static void Main(string[] args)
    {
        var product = Product.InnerBuilder.WithName("Laptop")
                       .WithDescription("Dell Laptop")
                       .WithPrice(1000)
                       .Build();

        Console.WriteLine(product);
        Console.WriteLine(Product.InnerBuilder.WithName("Laptop")
                            .GetType());
        Console.WriteLine(Product.InnerBuilder.WithName("Laptop")
                           .WithDescription("Dell Laptop")
                           .GetType());
        Console.WriteLine(Product.InnerBuilder.WithName("Laptop")
                           .WithDescription("Dell Laptop")
                           .WithPrice(1000)
                           .GetType());
    }
}
```

Görüldüğü üzere artık kullanım sırasının önemi kalmadan bir product build edilebilmektedir. Alttaki `.GetType()` içeren konsola yazdırılan satırlar ise her işlemin aslında en child class'ın tipini döndürdüğünün kontrolünü sağlamaktadır ve çıktı olarak konsolda hepsi en child (``Builder``) class'ı göstermektedir.

Çıktı:
```
Name: Laptop, Description: Dell Laptop, Price: 1000
InheritedBuilder.Product+Builder
InheritedBuilder.Product+Builder
InheritedBuilder.Product+Builder
```

Inner Class kullanımı, çeşitli prensiplerin göz ardı edilmesi ve code smell'e sebebiyet vermektedir. Bundan kaynaklı, farklı bir yöntem olarak aşağıdaki gibi InheritedBuilder class'ların en child'ından inherit alan bir Builder class oluşturup onu kullanarak ``Product`` Build etme işlemi yapılabilmektedir.

```
public class ProductBuilder : ProductPriceBuilder<ProductBuilder>
{
    public ProductBuilder() {}
}
```

ve ardından şu şekilde product oluşturulabilir:
```
internal class Program
{
    static void Main(string[] args)
    {
        var builder = new ProductBuilder();
        var product = builder.WithName("Laptop")
                       .WithDescription("Dell Laptop")
                       .WithPrice(1000)
                       .Build();
        Console.WriteLine(product);
    }
}
```

---

# Stepwise Builder

Bir diğer Builder çeşidi ise ``StepwiseBuilder``. Buradaki build işleminin yapılacağı class'ın propertylerini rastgele bir şekilde değil, bir sıra izleyerek yapılmasını developer'a şartlandıran bir builder'dır.

Aşağıdaki örnekte, eski ``Product`` class'a ek olarak `ProductType` enum eklendi ve ilerleyen süreçte bu enum ile şartlandırılacaktır.

```
public class Product
{
    public string Name { get; set; }
    public string Description { get; set; }
    public ProductType ProductType { get; set; }
    public decimal Price { get; set; }

    public override string ToString()
    {
        return $"Name: {Name}, Description: {Description}, Price: {Price}";
    }
}

public enum ProductType
{
    Laptop,
    Desktop,
    Mobile
}
```

Builder Pattern'e stepwise şekilde adım adım, developer'ı sınırlandıracak şekilde ayarlayabilmemiz için koşullu bir yapı da oluşturabiliriz ancak burada interface'lerden yararlanacağız.

```
public interface ISpecifyName
{
    public ISpecifyProductType WithNameAndDescription(string name,string description);
}

public interface ISpecifyProductType
{
    public ISpecifyPrice OfType(ProductType type);
}
public interface ISpecifyPrice
{
    public IBuildProduct WithPrice(decimal price);
}

public interface IBuildProduct
{
    public Product Build();
}
```

Yukarıdaki kod bloğundan anlaşılacağı üzere her işlemi ayrı ayrı interface'lere bölmüş durumdayız. Burada dikkat edilmesi gereken olay bir interface'in içerisindeki imzanın döndürdüğü tip bir sonraki interface olmaktadır ve bunun amacı sıralı yapıyı koşullandırmaktır çünkü kullanılacak fonksiyonun dönüş tipi sıradaki fonksiyonun interface'i olacak ve dolayısıyla `OfType()` fonksiyonu kullanıldıktan sonra tip olarak `WithPrice(decimal price)` içeren bir interface döndürdüğünden kaynaklı developer istese de istemese de sıradaki build edeceği fonksiyon `WithPrice` methodu olmak zorundadır.

Aşağıda ise anlatılanların implementasyonu bulunmaktadır.

```
public class ProductBuilder
{
    public ISpecifyName Create()
    {
        return new InnerBuildImplementation();
    }
    private class InnerBuildImplementation:
        ISpecifyName, ISpecifyProductType, ISpecifyPrice, IBuildProduct
    {
        private readonly Product _product = new Product();
        public ISpecifyProductType WithNameAndDescription(string name, string description)
        {
            _product.Name = name;
            _product.Description = description;
            return this;
        }
        public ISpecifyPrice OfType(ProductType type)
        {
            _product.ProductType = type;
            return this;
        }
        public IBuildProduct WithPrice(decimal price)
        {
            switch (_product.ProductType)
            {
                case ProductType.Laptop when price > 0 && price < 1000:
				    break;
				case ProductType.Desktop when price > 0 && price < 10000:
				    break;
				case ProductType.Mobile when price > 0 && price < 100:
				    break;
				default:
				    throw new Exception("Invalid price for the product type");
            }
            _product.Price = price;
            return this;
        }
        public Product Build()
        {
            return _product;
        }
    }
}
```

Yukarıda bir inner class kullanılarak implementasyonun yapıldığını görülmektedir. Bunun sebebi eğer `WithNameAndDescription`, `OfType`, `WithPrice`, `Build` fonksiyonlarının herhangi bir yerden erişilememesi gerekmektedir, eğer erişilebilirse StepwiseBuilder pattern'in sıralı bir şekilde build etme özelliği geçersiz olmaktadır. Böylece üstteki kod bloğunda, class'ın içerisine sadece bu inner class'ı oluşturacak kısmı public verdikten sonra işlem istenildiği gibi sıralı ve kurallı bir hal almaktadır ve aşağıdaki şekilde kullanılabilmektedir.

```
internal class Program
{
    static void Main(string[] args)
    {
        var builder = new ProductBuilder();

		var product = builder.Create().WithNameAndDescription("Laptop", "Dell Laptop")
		    .OfType(ProductType.Laptop)
		    .WithPrice(1000)
		    .Build();

        Console.WriteLine(product);
    }
}
```

Ancak bu şekilde Builder class oluşturmak fark edildiği üzere ekstra bir `InnerBuildImplementation` class'ına ihtiyaç doğurmaktadır ve ne kadar temiz bir implementasyon olduğu tartışılır. Bunun yerine :

```
public class Product
{
    public string Name { get; set; }
    public string Description { get; set; }
    public ProductType ProductType { get; set; }
    public decimal Price { get; set; }
    private Product() { }
    public override string ToString()
    {
        return $"Name: {Name}, Description: {Description}, Price: {Price}";
    }
    public static ProductBuilder Builder => new ProductBuilder();
    public class ProductBuilder : ISpecifyName, ISpecifyProductType, ISpecifyPrice, IBuildProduct
    {
        private static Product _product = new Product();
        internal ProductBuilder() { }
        public ISpecifyProductType WithNameAndDescription(string name, string description)
        {
            _product.Name = name;
            _product.Description = description;
            return this;
        }
        public ISpecifyPrice OfType(ProductType type)
        {
            _product.ProductType = type;
            return this;
        }
        public IBuildProduct WithPrice(decimal price)
        {
            switch (_product.ProductType)
            {
                case ProductType.Laptop when price > 0 && price < 1000:
                    break;
                case ProductType.Desktop when price > 0 && price < 10000:
                    break;
                case ProductType.Mobile when price > 0 && price < 100:
                    break;
                default:
                    throw new Exception("Invalid price for the product type");
            }
            _product.Price = price;
            return this;
        }
        public Product Build()
        {
            return _product;
        }
    }
}
```
Product sınıfımızın içerisine tekrar inner bir şekilde Builder class oluşturarak ve buna ek olarak Product sınıfımızı `new` anahtar kelimesi ile oluşturulmasını da private constructor ile sağlayarak sadece ilgili Builder static property'si ile erişilebilen bir Product nesnesi oluşturma şartı koyulabilmektedir. Tekrardan çok temiz bir yaklaşım değil ancak önceki yaklaşıma göre hem ekstra bir class implemente edilmiyor hem de ilgili nesnenin oluşturulması bu yöntem ile sınırlandırılabilmektedir (Herhangi bir yerde `new` ile initialize edilebilmemesi sağlanır).

```
internal class Program
{
    static void Main(string[] args)
    {
        var product = Product.Builder
            .WithNameAndDescription("Laptop2", "Dell Laptop2")
            .OfType(ProductType.Laptop)
            .WithPrice(500)
            .Build();

        Console.WriteLine(product);
    }
}
```

Böylece yukarıdaki şekilde eski haline göre hem daha fazla özelliğe sahip hem de daha temiz bir implementasyon sağlayabiliriz.

---

# Functional Builder

FunctionalBuilder aslında çok dinamik bir yapı ve actionlar kullanılarak yapıldığı için base bir class oluşturup bundan türeyen spesifik Builder class'lar ile devam ediyoruz.

```
public abstract class FunctionalBuilder<TObject, TBuilder>
    where TBuilder : FunctionalBuilder<TObject, TBuilder>
    where TObject : new()
{
    private readonly List<Func<TObject, TObject>> _actions = [];
    public TBuilder With(Action<TObject> action)
    {
        _actions.Add(obj =>
        {
            action(obj);
            return obj;
        });
        return (TBuilder)this;
    }
    public TObject Build()
    {
        return _actions.Aggregate((Func<TObject, TObject>)(obj => obj), (current, action) => obj => action(current(obj)))(Activator.CreateInstance<TObject>());
    }
}
```

Yukarıdaki kodda, abstract generic bir base class bulunmaktadır. Generic parametreler, `TObject` Build edilecek class'ın tipi ve TBuilder ise FluentBuilder gibi çalışması için fonksiyonelite eklettirilecek methodun dönüş tipi olacak olan Builder class'ın tipidir. Class içerisinde `With` fonksiyonu ile eklenecek olan `action`'ların listesi ve en sonunda `Build()` operasyonu içinde listede tutulan bütün `action`'ların teker teker ilgili nesneye uygulanması sağlanır.

```
public class ProductBuilder : FunctionalBuilder<Product, ProductBuilder>
{
    public ProductBuilder WithName(string name)
    {
        return With(p => p.Name = name);
    }
}
```

Yukarıdaki kullanımda ise eğer bu aşırı dinamik yapı sınırlandırılmak istenilirse spesifik fonksiyonlar eklenebilir ve hatta `With` methodu `protected` `access modifier` ile sınırlandırılırsa `With` methodunu sadece ``FunctionalBuilder`` class'ını inherit eden Builder class'lar kullanabilecek hale gelecektir.

```
internal class Program
{
    static void Main(string[] args)
    {
        var builder = new ProductBuilder();

        var product = builder.WithName("Laptop")
            .With(p => p.Description = "Dell Laptop")
            .With(p => p.Price = 10001)
            .Build();

        Console.WriteLine(product);
    }
}
```

---

# Faceted Builder

FacatedBuilder, birden çok builder'ın tek bir builder çatısı altında birleştirilmesi gibi düşünülebilir. Örneğin bir nesnenin çok fazla property'si bulunmakta ve o kadar çok ki bu property'leri kendi aralarında gruplaştırılabilir. Her grubun kendine özgü Builder class'ı oluşturulup daha sonrasında temel bir Builder class altında birleştirilerek FacatedBuilder class oluşturulmuş olur.

---

## Sonuç

Çeşitli ihtiyaçlar için çok fazla çeşit implementasyon bulunmaktadır. İhtiyaca göre belirli kuralları izlemek developer'ı hem de sonraki developer'ı çok daha iyi ve hızlı bir şekilde development yapmasını sağlar. Bu pattern, çeşitli nesneleri daha anlaşılır bir biçimde oluşturma konusunda ciddi oranda yardımcı olmakla birlikte, zengin çeşitliliği ile çoğu ihtiyacı karşılayacak implementasyonları mevcuttur.


---
# Kaynakça

- [Desing Patterns for Testing ~ NimblePros](https://www.youtube.com/watch?v=kB1bb7q7f0A)
- [Stepwise Builder (Wizard) in C#](https://www.youtube.com/watch?v=CY5hCx7QEAk)
- [Functional Builder in C#](https://www.youtube.com/watch?v=uK5SGBam0Uk)
- [Design Patterns in C# and .NET (udemy course)](https://www.udemy.com/course/design-patterns-csharp-dotnet/?referralCode=75592BB2072F94C0CDAE)
- [Builder Pattern | Design Patterns | Tasarım Kalıpları ~Tech Buddy](https://www.youtube.com/watch?v=Spq_ClIy-AM)
- [SOLID Wash Tunnel - Fluent Builder (part 1/3)](https://www.ledjonbehluli.com/posts/wash-tunnel/fluent_builder_part_1/)
- [SOLID Wash Tunnel - Fluent Builder (part 2/3)](https://www.ledjonbehluli.com/posts/wash-tunnel/fluent_builder_part_2/)
- [SOLID Wash Tunnel - Fluent Builder (part 3/3)](https://www.ledjonbehluli.com/posts/wash-tunnel/fluent_builder_part_3/)
- [Improve Tests with the Builder Pattern for Test Data](https://ardalis.com/improve-tests-with-the-builder-pattern-for-test-data/)
