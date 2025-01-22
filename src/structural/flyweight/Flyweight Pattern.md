<!-- TODO : pros cons eklenmeli, yazım hataları düzeltilmeli, ingilizce kelimelerin ilk görüldüğü yerde türkçelerinin parantez içerisinde belirtilmesi ve sonrasında hep ingilizce kullanılması.  -->
Flyweight Pattern, basitçe tanımlamak gerekirse uygulamada initialize edilen instance'ların (nesnelerin) cache mekanizması kullanılarak saklanması ve ihtiyaç dahilinde nesnenin tekrar tekrar initalize edilmeden bu cache'den edinilmesi tekniğiyle çalışmaktadır. 

Böylece duplication'dan kurtularak memory efficiency ve performansı sağlamaktadır.

## Hangi Durumlarda Kullanılır?

1. **Çok sayıda benzer nesne** oluşturmak zorunda olduğumuzda,
2. Her nesnenin içerisinde tekrar eden (çoğunlukla sabit) veriler varsa,
3. Bu sabit verileri merkezi bir yerde tutup paylaşmak, büyük bellek kazancı sağlıyorsa,
4. Nesnelerin farklı kullanımlara ilişkin küçük farklılıkları (dışsal durum) ayrı ayrı saklanıp yönetilebiliyorsa.

Bu pattern'i kullanırken extrinsic ve intrinsic property'lere dikkat edilmesi gerekmektedir. intrinsic property'ler ilgili flyweight nesnemizde ortaklaşa kullanılması potansiyel olan ve flyweight pattern ile duplicate olmaması için saklanacak olan nesnelerde bulunan property'ler extrinsic property'ler ise nesneden nesneye sıkça değişiklik gösteren property'lere denmektedir. 

Örneğin:

- Grafik uygulamalarında yüzlerce/dinamik olarak binlerce daire, kare nesneleri oluştururken renk, çizgi stili gibi ortak bilgileri paylaşmak benzer şekilde bellek kazancı sağlar.
- Oyunlarda oluşturulan karakterlerin aynı özellikte ve sık oluşturulanları bu pattern ile bellek kazancı sağlayarak tek bir instance ile oluşturulabilmeyi sağlar.
- Örneğin bir Metin editöründe binlerce karakter (harf) nesnesi oluşturulabilir. Harfin tipik özellikleri (font ailesi, stil, vb.) tekrar eden ortak veri olarak (intrinsic) Flyweight nesnelerinde saklanabilir. Her harfin konumu, rengi gibi değişken (extrinsic) durum dışarıdan sağlanır.

İkinci örnek üzerinden implementasyon yapılacak olursa öncelikle harf nesnesi oluşturulur:

```
public class Character
{
    private IntrinsicCharacterProperties IntrinsicCharacterProperties { get; set; }
    public int XCoordinate { get; set; }
    public int YCoordinate { get; set; }
    public string Color { get; set; }

    public Character(string font, string size, int xCoordinate, int yCoordinate, string color)
    {
        IntrinsicCharacterProperties = IntrinsicCharacterPropertiesFlyweight.GetOrSet(font, size);
        XCoordinate = xCoordinate;
        YCoordinate = yCoordinate;
        Color = color;
    }
}

public class IntrinsicCharacterProperties
{
    public string Font { get; set; }
    public string Style { get; set; }
}
```

Yukarıdaki kod bloğundan görüldüğü üzere `Character` sınıfının üç adet extrinsic ve iki adet intrinsic property'si bulunmakta ve bu intrinsic property'ler `IntrinsicCharacterProperties` sınıfı altında toplanmıştır. Burada dikkat edilmesi gereken kısım ``Character`` sınıfı initialize edildiği zaman parametre olarak `IntrinsicCharacterProperties` içerisindeki property'leri almış ve  `IntrinsicCharacterPropertiesFlyweight` isimli static sınıfın `GetOrSet` methodunu çağırarak `IntrinsicCharacterProperties`'i varsa getiriyor yoksa initialize ediyor.

Aşağıdaki kod bloğu ise `IntrinsicCharacterPropertiesFlyweight` sınıfımızın içeriği. Burada basit bir şekilde nesneden üretilecek olan unique (eşsiz) bir hash veya key ile oluşturulan dictionary veri yapısına ilgili nesne eklenmektedir veya hali hazırda içerisinde saklanıyor ise ilgili methodun çağırıldığı yere ilgili instance döndürülmektedir. Böylece, font ve style'ı aynı olan nesneler tekrar tekrar initialize edilmek yerine tek bir sınıf oluşturulup her yerde o kullanılmaktadır. 

```
public static class IntrinsicCharacterPropertiesFlyweight
{
    private static Dictionary<string, IntrinsicCharacterProperties> _cache = [];
    public static IntrinsicCharacterProperties GetOrSet(string font, string style)
    {
        var key = font + style;
        if (_cache.TryGetValue(key, out var value))
        {
            return value;
        }
        var properties = new IntrinsicCharacterProperties
        {
            Font = font,
            Style = style
        };
        _cache.TryAdd(key, properties);
        return properties;
    }
}
```

Önemli Not: sınıflar referans değer olduğundan kaynaklı içeriğindeki herhangi bir değişim bütün kullanıldığı yerleri etkileyecektir ondan dolayı içeriği değiştirilmeyecek property'lerin flyweight objelere dahil edilmesi sağlıklı olmaktadır.

## Generic Flyweight Pattern

Flyweight pattern'i kullanabilmek için her sınıfın içerisinde intrinsic property'lerin ayrıldığı sınıflara özel flyweight sınıfların oluşturulması ve içerisinde yukarıdaki gibi bir implementasyon yapılması çok fazla kod tekrarına sebebiyet vermektedir. Buna çözüm olarak, `Generic Flyweight Pattern` geliştirilebilir.

Aşağıdaki kod parçacığı buna basit bir örnek:

```
public static class EmopFlyweight<EObject> where EObject : class
{
    private static ConcurrentDictionary<string, EObject> _cache = [];

    public static EObject GetOrSet(params object[] parameters)
    {
        string hash = string.Join("", parameters);
        hash += "-" + typeof(EObject).FullName;
        hash = hash.GetHashCode().ToString();
        if (_cache.TryGetValue(hash, out var value))
        {
            return _cache[hash];
        }

        var @object = (EObject)Activator.CreateInstance(typeof(EObject), parameters);
        _cache.TryAdd(hash, @object);
        return @object;
    }

    public static EObject GetOrSet(object parameter)
    {
        var hash = parameter.ToString() + "-" + typeof(EObject).FullName;
        hash = hash.GetHashCode().ToString();
        if (_cache.TryGetValue(hash, out var value))
        {
            return _cache[hash];
        }

        var @object = (EObject)Activator.CreateInstance(typeof(EObject), parameter);
        _cache.TryAdd(hash, @object);
        return @object;
    }
}
```

Burada ``EmopFlyweight`` isimli sınıfa alınan generic parametre ile hangi obje sınıfının flyweight pattern'ine tabi tutulacağı belirtilmektedir. Ardından bu sınıfın önceki implementasyondaki gibi hash veya unique key'i oluşturulmasıyla birlikte cache'den çekilmeye çalışılmaktadır. Burada gözle görülen farklılığın `Activator` kullanılarak ilgili generic parametre tipinin oluşturulması gözlemlenmektedir. Bu işlem ilk implementasyondaki `new IntrinsicCharacterProperties()` satırıyla aynı işleme denk düşmektedir, sadece generic parametreden gelen tipin initialize edilebilmesi için bu şekilde implemente edilmesi gerekmektedir. 

Tekrar hatırlatılması gerekirse, intrinsic property'lerinin erişiminin sınırlandırılması ve değiştirilmelerinin engellenmesi ilgili implementasyonda referans değer olan class kullanılmasından kaynaklı ciddi önem taşımaktadır. Yani, `private` access modifier ile ilgili property'lerin dışarıdan erişime kapatılmalıdır!

# Kaynakça


https://www.tutorialspoint.com/design_pattern/flyweight_pattern.htm

https://www.youtube.com/watch?v=qscOsQV-K14

https://www.youtube.com/watch?v=KtwdU2Vzde8

https://www.youtube.com/watch?v=RDV0ioVTF4g

https://medium.com/kodcular/flyweight-design-pattern-nedir-deabe3c2b93d

https://refactoring.guru/design-patterns/flyweight