Всё сказанное относится и к виртуальным свойствам и событиям, поскольку они, как показано далее, на самом деле реализуются методами.

```csharp
internal class Employee
{
	// Невиртуальный экземплярный метод
	public Int32 GetYearsEmployed() { return 1; }
	// Виртуальный метод (виртуальный - значит, экземплярный)
	public virtual String GetProgressRepor() { return ""; }
	// Статический метод
	public static Employee Lookup(String name) { return new Employee(); }
}
```

![Pasted image 20230311153424.png](/img/user/Files/Image/Pasted%20image%2020230311153424.png)

При компиляции этого кода компилятор помещает три записи в таблицу определений методов сборки. Каждая запись содержит флаги, указывающие, является ли метод экземплярным, виртуальным или статическим. 


```csharp
public sealed class Program
{
	public static void Main()
	{
		Console.WriteLine(); // Вызов статического метода
		Object o = new Object();
		o.GetHashCode(); // Вызов виртуального экземплярного метода
		o.GetType(); // Вызов невиртуального экземплярного метода
	}
}
```

При компиляции кода, ссылающегося на эти методы, компилятор проверяет флаги в определении методов, чтобы выяснить, какой IL-код нужно вставить для корректного вызова методов. В CLR есть две инструкции для вызова метода call и callvirt:


### Вызов виртуальных методов невиртуально

Если JIT видит, что виртуальный вызов происходит на запечатанном `sealed` классе, он может его **девиртуализовать**, или даже **встроить**(inline) в вызывающий код.

###### Кратко:
1. **call** не проверяет вызывающий объект на значение null и используется только тогда , когда JIT компилятор 100% может гарантировать, что ссылка никогда не будет null.
2. **callvirt** проверяет и в случает когда вызывающий объект равен null выдаёт исключение NullReferenceException

По факту JIT компилятор не делает сложного анализа кода и заменяет **callvirt** на **call** только в редких и очевидных случаях (например, вызов ToString на значимых типах), а так везде пихает **callvirt**, даже если имеет возможность проанализировать метод и убедиться, что ссылка не будет null.

Невиртуальные методы вызываются быстрее виртуальных, поскольку для виртуальных CLR во время выполнения проверяет настоящий тип объекта, чтобы выяснить, где находится метод. Под нахождением метода понимается поиск ближайшей реализации виртуального метода обнаруживаемого вверх по иерархии, а для невиртуального экземплярного метода сама переменная однозначно показывает, где определен необходимый метод.



### Полно:

- Инструкция **call** используется для вызова статических, экземплярных и виртуальных методов. Если с помощью этой инструкции вызывается статический метод, необходимо указать тип, в котором определяется метод. При вызове экземплярного или виртуального метода необходимо указать переменную, ссылающуюся на объект, причем в **call** подразумевается, что эта переменная не равна **null**. Иначе говоря, сам тип переменной указывает, в каком типе определен необходимый метод. Если в типе переменной метод не определен, проверяются базовые типы. Инструкция **call** часто служит для невиртуального вызова виртуального метода. 

- Инструкция **callvirt** используется только для вызова **экземплярных** и **виртуальных** (но не статических) методов. При вызове необходимо указать переменную, ссылающуюся на объект. Если с помощью этой инструкции вызывается невиртуальный экземплярный метод, тип переменной показывает, где определен необходимый метод. При использовании callvirt для вызова виртуального экземплярного метода CLR определяет настоящий тип объекта, на который ссылается переменная, и вызывает метод **полиморфно**. При компиляции такого вызова JIT-компилятор генерирует код для проверки значения переменной — если оно равно **null**, CLR сгенерирует исключение NullReferenceException. Из-за этой дополнительной проверки инструкция **callvirt выполняется немного медленнее, чем call**. Проверка на null выполняется даже при вызове невиртуального экземплярного метода.

```csharp
.method public hidebysig static void  Main() cil managed
{
  .entrypoint
  // Размер кода:       24 (0x18)
  .maxstack  8
  IL_0000:  call       void [mscorlib]System.Console::WriteLine()
  IL_0005:  newobj     instance void [mscorlib]System.Object::.ctor()
  IL_000a:  dup
  IL_000b:  callvirt   instance int32 [mscorlib]System.Object::GetHashCode()
  IL_0010:  pop
  IL_0011:  callvirt   instance class [mscorlib]System.Type [mscorlib]System.Object::GetType()
  IL_0016:  pop
  IL_0017:  ret
} // end of method Program::Main
```

- Поскольку метод WriteLine является статическим, компилятор C# использует для его вызова инструкцию call.
- Для вызова виртуального метода GetHashCode применяется инструкция callvirt. 
- Метод GetType также вызывается с помощью инструкции callvirt. ~~Это выглядит странно, поскольку метод GetType невиртуальный~~ (Не знаю что смутило автора, если GetType это экземплярный метод и выше он писал, что экзмеплярные методы вызываются с помощью callvirt. 

	Тем не менее код работает, потому что во время JIT-компиляции CLR знает, что GetType - это невиртуальный метод, и вызывает его невиртуально. Разумеется, возникает вопрос: почему компилятор C# не использует инструкцию call? Разработчики C# решили, что JIT-компилятор должен генерировать код, который проверяет, не равен ли null вызывающий объект. Поэтому вызовы невиртуальных экземплярных методов выполняются чуть медленнее, чем могли бы – а также то, что следующий код в C# вызовет исключение NullReferenceException, хотя в некоторых языках все работает отлично.

> Если нам угодно, можно в IL коде изменить callvirt на call и тогда мы пропустим проверку значения переменной на null  и кот будет работать быстрее.


**Рихтер утверждает:**
Иногда компилятор вместо **callvirt** использует для вызова **виртуального метода** команду **call**. Такое поведение выглядит странно, но следующий пример показывает, почему это действительно бывает необходимо.

При вызове виртуального метода base.ToString компилятор C# вставляет команду call, чтобы метод ToString базового типа вызывался невиртуально. Это необходимо, ведь если ToString вызвать виртуально, вызов будет выполняться рекурсивно до переполнения стека потока — что, разумеется, нежелательно.

```csharp
internal class SomeClass
{
	// ToString - виртуальный метод базового класса Object
	public override String ToString()
	{
	// Компилятор использует команду call для невиртуального вызова
	// метода ToString класса Object
	// Если бы компилятор вместо call использовал callvirt, этот
	// метод продолжал бы рекурсивно вызывать сам себя до переполнения стека
		return base.ToString();
	}
}
```

Для виртуального вызова виртуального метода значимого типа CLR необходимо получить ссылку на объект значимого типа, чтобы воспользоваться его **таблицей методов**, а это требует упаковки значимого типа. Упаковка повышает нагрузку на кучу, увеличивая частоту сборки мусора и снижая производительность.

В .NET Fraemwork 4.5.2 используется callvirt для вызова метода и никакого переполнения не происходит.
![Pasted image 20230310200722.png](/img/user/Files/Image/Pasted%20image%2020230310200722.png)

Независимо от используемой для вызова экземплярного или виртуального метода инструкции — call или callvirt — эти методы всегда в первом параметре **получают скрытый аргумент this**, **ссылающийся на объект**, с которым производятся действия.



[cyberforum](https://www.cyberforum.ru/csharp-beginners/thread2184218.html)
[stackoverflow](https://stackoverflow.com/questions/193939/call-and-callvirt)
[stackoverflow](https://stackoverflow.com/questions/2134/do-sealed-classes-really-offer-performance-benefits)
[microsoft](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-6/#peanut-butter)
[cyberforum](https://www.cyberforum.ru/csharp-net/thread1631924.html)
[stackoverflow](https://ru.stackoverflow.com/questions/1075088/%D0%9A%D0%B0%D0%BA-%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%B0%D1%8E%D1%82-%D0%B2%D0%B8%D1%80%D1%82%D1%83%D0%B0%D0%BB%D1%8C%D0%BD%D1%8B%D0%B5-%D1%84%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D0%B8-%D0%B2-c)
[habr](https://habr.com/ru/company/clrium/blog/344556/)
[git](https://github.com/sidristij/dotnetbook)
[stackoverflow](https://ru.stackoverflow.com/questions/1075088/%D0%9A%D0%B0%D0%BA-%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%B0%D1%8E%D1%82-%D0%B2%D0%B8%D1%80%D1%82%D1%83%D0%B0%D0%BB%D1%8C%D0%BD%D1%8B%D0%B5-%D1%84%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D0%B8-%D0%B2-c)
[microsoft](https://learn.microsoft.com/en-us/archive/msdn-magazine/2005/may/net-framework-internals-how-the-clr-creates-runtime-objects)
[my cyberforum](https://www.cyberforum.ru/csharp-beginners/thread3088225.html#post16799663)
[my stackoverflow](https://ru.stackoverflow.com/questions/1504574/%d0%92%d1%8b%d0%bf%d0%be%d0%bb%d0%bd%d0%b5%d0%bd%d0%b8%d0%b5-%d0%b2%d0%b8%d1%80%d1%82%d1%83%d0%b0%d0%bb%d1%8c%d0%bd%d1%8b%d1%85-%d0%bc%d0%b5%d1%82%d0%be%d0%b4%d0%be%d0%b2-%d0%b2-%d0%b7%d0%b0%d0%bf%d0%b5%d1%87%d0%b0%d1%82%d0%b0%d0%bd%d0%bd%d1%8b%d1%85-%d0%ba%d0%bb%d0%b0%d1%81%d1%81%d0%b0%d1%85/1504949#1504949)
[MethodTable](https://stackoverflow.com/questions/33201811/net-clr-method-table-structure)

[stackoverflow](https://stackoverflow.com/questions/770547/how-does-compiler-optimize-virtual-methods-implemented-by-a-sealed-class)

```csharp
class A
{
	public virtual void Show()
	{
		Console.WriteLine("A");
	}
}
class B:A
{
	public override void Show()
	{
		Console.WriteLine("B");
	}
}

```

[bestprog](https://www.bestprog.net/ru/2020/04/10/c-late-and-early-binding-polymorphism-basic-concepts-examples-passing-a-reference-to-a-base-class-in-a-method_ru/#q01)
[stackoverflow](https://stackoverflow.com/questions/32536629/polymorphism-and-virtual-method)

### Sealed и вызов виртуальных методов невиртуально

Sealed вносит ограничения:
Нельзя наследовать класс помеченный оператором sealde => такой класс не может выступать больше базовым =>

Мы можем либо создать объект этого класса:
```csharp
B ob=new B();
```
(переменной ссылки типа данного класса может быть присвоена ссылка на объект типа данного класса). В этом случае произойдёт девиртуализация метода.

Либо этот класс может выступать производным:
```csharp
A ob2=new B();
```
(переменной ссылки на объект базового класса может быть присвоена ссылка на объект любого производного от него класса). Тут происходит upcasting. В этом случае девиртуализации не произойдёт

JITer различает эти две ситуации. JITer различает когда, происходит upcasting и downcasting.

```csharp
class Program
{
    public class BaseType
    {
        public virtual int M() => 1;
    }

    public class NonSealedType : BaseType
    {
        public override int M() => 2;
    }

    public sealed class SealedType : BaseType
    {
        public override int M() => 2;
    }
    
    public static int SealedInc(SealedType _sealed) => _sealed.M() + 42;
    
    public static int NonSealedInc(NonSealedType _nonSealed) => _nonSealed.M() + 42;

    static void Main()
    {
        SealedType _sealed = new SealedType();
        
        NonSealedType _nonSealed = new NonSealedType();
        
        SealedInc(_sealed);

        NonSealedInc(_nonSealed);
        
    }
}
```

В метод `SealedInc` может быть передана ссылка только типа класса SealedType, т.к. SealedType является запечатанным и не может быть такой ситуации, что он выступит базовым типом и примет ссылку на производный класс. То есть JITer или компилятор однозначно могут гарантировать, что этот метод больше не может быть переопределен, поэтому ему не нужно смотреть во время выполнения, какой метод выполнять(ему не нужно определять метод по типу объекта), он просто может автоматически выполнить SealedType.M() невиртуально(посредством call), и даже посредством прямой подстановки [inline](https://ru.stackoverflow.com/questions/606360/inline-%D1%82%D0%B5%D1%80%D0%BC%D0%B8%D0%BD-%D0%B2-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5%D0%BA%D1%81%D1%82%D0%B5-c-jit-%D0%BA%D0%BE%D0%BC%D0%BF%D0%B8%D0%BB%D1%8F%D1%82%D0%BE%D1%80%D0%B0), что раньше было невозможно.

```assembly
Program.SealedInc(SealedType)
    L0000: cmp [ecx], cl
    L0002: mov eax, 0x2c
    L0007: ret

Program.NonSealedInc(NonSealedType)
    L0000: push ebp
    L0001: mov ebp, esp
    L0003: mov eax, [ecx]
    L0005: mov eax, [eax+0x28]
    L0008: call dword ptr [eax+0x10]
    L000b: add eax, 0x2a
    L000e: pop ebp
    L000f: ret
```

Если мы сделаем класс незапечатанным, класс сможет выступать базовым клаccом для других классов, вследствие чего переменной ссылке базового класса может быть присвоена ссылка на объект производного класса, а как мы знаем у виртуальных методов определяется **нужный вариант** виртуального метода, который следует вызывать, исходя из типа объекта (объект-типа), к которому происходит обращение по ссылке, а не по типу ссылки на этот объект, причем это делается во время выполнения. После определения реального типа происходить обращение к таблице методов с **последующим вызовом** нужной реализации данного метода.

Если мы изменим метод `SealedInc` и сделаем чтобы он принимал ссылочную переменную типа `BaseType` 
```csharp
public static int SealedInc(BaseType _sealed) => _sealed.M() + 42;
```

```
Program.SealedInc(BaseType)
    L0000: push ebp
    L0001: mov ebp, esp
    L0003: mov eax, [ecx]
    L0005: mov eax, [eax+0x28]
    L0008: call dword ptr [eax+0x10]
    L000b: add eax, 0x2a
    L000e: pop ebp
    L000f: ret

Program.NonSealedInc(NonSealedType)
    L0000: push ebp
    L0001: mov ebp, esp
    L0003: mov eax, [ecx]
    L0005: mov eax, [eax+0x28]
    L0008: call dword ptr [eax+0x10]
    L000b: add eax, 0x2a
    L000e: pop ebp
    L000f: ret
```

То JITer уже не сможет вызвать виртуальный метод `M()` невиртуально, т.к. в метод может быть передана ссылка уже двух типов:
1. Типа `BaseType`. Это возможно т.к. переменной ссылки на объект базового класса может быть присвоена переменная на объект производного класса
2. Типа `SealedType`

И, следовательно, нужный метод уже нужно будет определять исходя из типа объекта(то есть заниматься виртуализацией)

То есть вызов виртуального метода невиртуально строиться на двух аспектах:
1. Ссылочной переменной типа запечатанного класса, может быть присвоена ссылка на объект только того же типа. 
2. Формальный параметр метода принимающий ссылку на запечатанный класс(экземпляр класса) должен быть того же типа запечатанного класса
    `public static int SealedInc(SealedType _sealed) => _sealed.M() + 42;`
    `SealedType _sealed = new SealedType();`
    `SealedInc(_sealed);`

Тесты вызова виртуальных методов невиртуально: [microsoft](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-6/#peanut-butter)

| Method    | Mean      | Ratio | Code Size |
| --------- | --------- | ----- | --------- |
| NonSealed | 1.7694 ns | 1.00  | 37 B      |
| Sealed    | 0.0749 ns | 0.04  | 36 B      |




### Как происходит вызов виртуальных методов в отличие от невитруальных.


```csharp
class Alpha
{
	public virtual void Show()
	{
		Console.WriteLine("Show Class Alpha");
	}
	public void Print()
	{
		Console.WriteLine("Print Class Alpha");
	}
}
class Beta : Alpha
{
	public override void Show()
	{
		Console.WriteLine("Show Class Beta");
	}
	new public void Print()
	{
		Console.WriteLine("Print Class Beta");
	}
}
public static void Main()
{
	Alpha alpha =new Alpha();
	alpha.Show();//Show Class Alpha
	alpha.Print();//Print Class Alpha

	Beta beta = new Beta();
	alpha = beta; //Upcasting
	alpha.Show();//Show Class Beta
	alpha.Print();//Print Class Alpha
}
```

При вызове JITer определяет именно тот вариант виртуального метода, который следует вызывать, исходя из объект-типа, к которому происходит обращение по ссылке, а не типа ссылки, причем это делается во время выполнения.

Ссылка типа просто ограничивает область видимости членов типа.

В последней строчке вызывается метод `Print()`, он невиртуальный и определён в классе` Alpha` и `Beta`, но т.к. ссылка тип `Alpha`(был произведён Upcasting), то и доступны только те методе, которые определены в типе Alpha.
