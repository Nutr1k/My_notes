---
dg-publish: true
---

_Значимые типы_ допускающий значение _NULL_ называются null-совместимыми значимыми типами

Итак, чтобы использовать в коде null-совместимый тип Int32, вы пишете конструкцию следующего вида: 
```csharp
Nullable<Int32> x = 5; 
Nullable<Int32> y = null;
Console.WriteLine("x: HasValue={0}, Value={1}", x.HasValue, x.Value); Console.WriteLine("y: HasValue={0}, Value={1}", y.HasValue, y.GetValueOrDefault());

//x: HasValue=True, Value=5 
//y: HasValue=False, Value=0
```

Упрощённый синтаксис
```csharp
Int32? x = 5; 
Int32? y = null;
```
В C# запись `Int32?` аналогична записи `Nullable`

Следует учесть, что для операций с экземплярами null-совместимых типов генерируется большой объем кода. 
К примеру, рассмотрим метод: 
```csharp
private static Int32? NullableCodeSize(Int32? a, Int32? b) 
{ return a + b; }
```

Вот эквивалент этого кода на C#:
```csharp
private static Nullable<Int32> NullableCodeSize(Nullable<Int32> a,
Nullable<Int32> b)
{
	Nullable<Int32> nullable1 = a;
	Nullable<Int32> nullable2 = b;
	if (!(nullable1.HasValue & nullable2.HasValue))
	{
		return new Nullable<Int32>();
	}
	return new Nullable<Int32>(
	nullable1.GetValueOrDefault() + nullable2.GetValueOrDefault());
}
```

#### Оператор объединения null-совместимых значений

В C# существует оператор объединения null-совместимых значений (null-coalescing operator). Он обозначается знаками `??` и работает с двумя операндами

```csharp
левый_операнд ?? правый_операнд
```

Оператор `??` возвращает левый операнд, если этот операнд не равен `null`. Иначе возвращается правый операнд


```csharp
string? text = null;
string name = text ?? "Tom";// равно Tom, так как text равен null
```

Также можно использовать производный оператора `??=`

```csharp
string? text = null;
text ??= "Sam";
// аналогично
// text = text ?? "Sam";
Console.WriteLine(text);    // Sam
```

#### Упаковка null-совместимых значимых типов

Представим переменную типа Nullable, которой логически присваивается значение null. Для передачи этой переменной методу, ожидающему ссылки на тип Object, ее следует упаковать и передать методу ссылку на упакованный тип. Однако при этом в метод будет передано отличное от null значение, несмотря на то что тип Nullable содержит null.

Если экземпляра Nullable равен null,то вместо упаковки передаётся значение null.

Если экземпляра Nullable не равен null, а равен какому-то значимому типу, то упаковывается это значение. Другими словами, тип Nullable со значением 5 упаковывается в тип Int32 с аналогичным значением.

```csharp
// После упаковки Nullable<T> возвращается null или упакованный тип T
	Int32? n = null;
	Object o = n; // o равно null
	Console.WriteLine("o is null={0}", o == null); // "True"
	
	n = 5;
	o = n; // o ссылается на упакованный тип Int32
	Console.WriteLine("o's type={0}", o.GetType()); // "System.Int32"
```

#### Распаковка null-совместимых значимых типов

В CLR упакованный значимый тип T распаковывается в T или в Nullable. Если ссылка на упакованный значимый тип равна null и выполняется распаковка в тип Nullable, иначе в тип Т.

```csharp
// Создание упакованного типа Int32
Object o = 5;
// Распаковка этого типа в Nullable<Int32> и в Int32
Int32? a = (Int32?)o; // a = 5
Int32 b = (Int32)o; // b = 5
					// Создание ссылки, инициализированной значением null
o = null;
// "Распаковка" ее в Nullable<Int32> и в Int32
a = (Int32?) o; // a = null
b = (Int32) o; // NullReferenceException
```

> [!success]  Если кратко
> CLR знает когда необходимо использовать Nullable тип, а когда сам тип. То есть если в типе содержится реальное значение(значимый тип), то по максимуму он будет рассматриваться просто как value type иначе как Nullable.