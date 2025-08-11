---
dg-publish: true
---
```csharp
class A
{
  public virtual void Show()
  {
        Console.WriteLine("Class A");
  }
}
class B : A
{
  public override void Show()
  {
    Console.WriteLine("Class B");
    base.Show();
  }
}
class Program
{
  public static void Main()
  {
    B ob = new B();
    ob.Show();
  }
}
```




roslyn почти всегда эмитит callvirt для всех instance методов за исключением вызванных через base. есть еще пару корнер кейсов

Он полиморфно вызывает метод из объекта this

callvirt для обычных методов юзают исключительно как хак - для нуллчека, по идее это не нужно

У тебя тип в calvirt резолвится в момент вызова, так что там будет реальный тип В всегда

Его хорошо поработал и иногда calvirt  резолвится в прямой вызов, но это детали реализации jit


 резолвится - это значит, что по указателю на объект (в данном случае это this) выполнение пойдет в таблицу методов, найдет там метод show и вызовет его


https://blog.washi.dev/posts/confusing-decompilers-with-call/

https://blog.washi.dev/posts/confusing-decompilers-with-callvirt/

https://stackoverflow.com/questions/193939/call-and-callvirt

Name resolve — это когда имени сопоставляется конкретная сущность из языка программирования.


```csharp
	public override void Show()
	{
		Console.WriteLine("Class B");
		base.Show();
	}
```
base.Show()-там прямой вызов - значит вызываем прямо A.Show, хочешь вритуальный - убери слово base.



При таком коде:
```csharp
object a = 1;
object b = a;
object c = b;
```

У нас в 'c' содержится ссылка на 'b', в 'b' ссылка на 'a", в 'a' ссылка на 1?


Не совсем так.

На первой строчке у тебя число боксится.

А потом все три переменные являются одинаковыми ссылками на один и тот же объект.

Не ссылки на переменные, а ссылки на объект — это важно.


Привет. У тебя несколько нарушены причины и следствия. Поколения отслеживаются GC и он сам делает ранжирование, что куда разместить по определённым правилам. И сам же GC запускает различные вариации сборки мусора.

Если хочешь деталей и разобраться, можешь например почитать вот этот документ (https://github.com/Maoni0/mem-doc/blob/master/doc/.NETMemoryPerformanceAnalysis.md#GC-Fundamentals) (полностью или тот раздел, ссылку на который я кидаю). Либо поищи книгу Konrad Kokosa - Pro .NET Memory Management, где всё максимально подробно (1000 стр) :) На русском книга называлась Управление памятью в .NET для профессионалов | Кокоса Конрад

[[Управление памятью в .NET для профессионалов (Конрад Кокоса) (Z-Library).pdf]]

GitHub (https://github.com/Maoni0/mem-doc/blob/master/doc/.NETMemoryPerformanceAnalysis.md)
mem-doc/doc/.NETMemoryPerformanceAnalysis.md at master · Maoni0/mem-doc
This is a document to help with .NET memory analysis and diagnostics. - Maoni0/mem-doc


На русском книга называлась Управление памятью в .NET для профессионалов | Кокоса Конрад

Если отвечать конкретно на твой вопрос, то он в разделе Understanding how GCs are triggered статьи, что я сбросил, но GC fundamentals все же советую первым изучить

Это один из подходов к "сборке мусора", в принципе тут можешь почитать базовые подходы: https://en.wikipedia.org/wiki/Tracing_garbage_collection (не зря ли я тебя этим гружу, смотри сам :) )

Поколения используются при Generational GC (ephemeral GC) реализациях. 

Если простыми словами, то это способ деления объектов по возрасту и затратам на отслеживание/освобождение. В 0м поколении это как правило практически бесплатно, чем дальше, тем дороже (с точки зрения самого GC) сделать освобождение и воспользоваться освобожденной памятью.

Где и как это хранится, как устроены различные режимы (он далеко не один в .NET) - за этим отправлю к книге или статье. Лучше книга, там максимально подробно.

Но в целом можешь все-таки изучить и статью для начала. Как хранится в разделе - Physical representation of the GC heap