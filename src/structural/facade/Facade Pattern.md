Facade Design Pattern, karmaşık işlemlerin, implementasyonların, işlevlerin, kullanımların, vb... basit bir arayüz sayesinde, gerek aşırı konfigurasyona gerekse alt sistemlerin işleyişinin bilinmesine ihtiyaç duyulmadan, kullanımını kolaylaştırmaya yarayan bir pattern'dir. 

"Provide a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use." Design Patterns Book by Gang of Four.

Facade'ın en basit örneği bir console uygulamasında tek satır kod ile arayüzde mesaj gösterilmesini sağlayan aşağıdaki örnek kod bloğu.

```csharp
internal class Program
{
	private static void Main(string[] args)
	{
		Console.WriteLine("Hello, World!");
	}
}
```

Buradaki kod bloğu, basit bir şekilde ekrana `Hello, World!` basmaktadır. Bu işlemin yapılması sırasında `Console`'un buffer'ından', viewport'undan, style'larından, texture'undan, ... Kısacası neredeyse arkada çalışan hiçbir işlemden haberdar olunmadan en high-level (yüksek seviye) fonksiyonlardan birini kullanarak ekrana yazı yazdırılmaktadır.

# Kaynakça

Ücretsiz:<br>
- https://www.youtube.com/watch?v=K4FkHVO5iac <br>
- https://refactoring.guru/design-patterns/facade <br>

Ücretli:
- [Desing Patterns: Elements of Reusable Object-Oriented Software Book](https://www.amazon.com.tr/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612/ref=sr_1_2?__mk_tr_TR=%C3%85M%C3%85%C5%BD%C3%95%C3%91&crid=WHUMS0DHJBKO&dib=eyJ2IjoiMSJ9.mTRaTOPYqsPcUsGD8azntQBwoQYmLa7486oAF-n21naeCMl-cWRy6Tc4xyGXPHzIe4pgk3yyBBQ5xXEXy_yChPa8_t7-ZEiWFDxX6xRvYtws2SsECY5g6_L03uQXeOL8hFzn00c2Ccjiq1EKQHmZEb4mUS1O4esM4UrdgbgWi_EB92UbzYH7rBFb5SJsRLxTch6rUKNqSfxO9I9FBaaZQoJbC04f4JZKGyaf1G6QW5xcHb7AJ4gMh3peaP8xz24u7sXUMLs7M8RIAByW4YO97lxJNs2AjFfzRyJTMtZlxpY.xPLI_w471Dn2oGOGQVdfmRuoMEX8cetRTg0iYLmadDo&dib_tag=se&keywords=design+patterns%2C&qid=1752059924&sprefix=design+pattern%2Caps%2C762&sr=8-2)
- https://www.udemy.com/course/design-patterns-csharp-dotnet/