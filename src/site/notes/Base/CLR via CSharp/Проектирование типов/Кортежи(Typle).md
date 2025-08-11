---
dg-publish: true
---

Кортежный тип (tuple type) — это тип, который содержит коллекцию свойств, каким-то образом связанных друг с другом. Кортежи в отличии от анонимных типов не создают неявный класс по которому создают впоследствии объект. Для кортежей заранее предусмотрен класс с множеством перегрузок для множества значений.

Кортежи доступны только для чтения.
Предоставляют свои значения через свойства

Упрощённая форма кортежного класса
```csharp
[Serializable]
public class Tuple<T1,T2, T3>
{
	private T1 m_Item1;
	private T2 m_Item12;
	private T3 m_Item3;
	...
	public Tuple(T1 item1, T2 item2, T3 item3 ...) 
	{ 
		m_Item1 = item1; 
		m_Item2 = item2; 
		m_Item3 = item3;
		...
	}
	public T1 Item1 { get { return m_Item1; } }
	public T2 Item1 { get { return m_Item2; } }
	public T2 Item1 { get { return m_Item3; } }
	public TRest Rest { get { return m_Rest; } }
}

```
На самом деле у кортежей есть множество перегрузок для разного числа принимаемых формальных параметров(от 1 до 8)



Способы создание кортежей:
```csharp
//1. new
Tuple<int, int> tuple1 = new Tuple<int, int>(1, 2);
//Упрощённо
var tuple1_1 = new Tuple<int, int>(1, 2);

//2. Create
Tuple<int, int> tuple2 = Tuple.Create<int, int>(1, 2);
//Упрощённо
var tuple2_2 = Tuple.Create(1, 2);

```
Отличие кортежей от анонимных типов

|                                    | Кортежи | Анонимные типы |
| ---------------------------------- | :-------: |: --------------:|
| Доступны только для чтения         | ✓        | ✓              |
| Изменение имён свойств             | ✓       | ✓              |
| Может быть формальным параметром   | ✓       | ✕              |
| Может быть возвращаемым параметром | ✓       | ✕              |

![Pasted image 20230512203135.png](/img/user/Files/Image/Pasted%20image%2020230512203135.png)

#### ValueTyple
Это структура. Доступны для изменения. Предоставляют свои значения через свойства

```csharp
//3. Неявное определение типов
var tuple3 = (5, 10);

//4. Явное определение типов 
(int, int) tuple4 = (5, 10);
//Аналогично
ValueTuple<int, int> valueTuple = new ValueTuple<int, int>(5, 10);

//5. Кортежи с названиями полей
var tuple5 = (count:5, sum:10);
```

Обращение к кортежам:
```csharp
//1
var tuple = (5, 10);
Console.WriteLine(tuple.Item1); // 5
Console.WriteLine(tuple.Item2); // 10

//2
var tuple = (count:5, sum:10);
Console.WriteLine(tuple.count); // 5
Console.WriteLine(tuple.sum); // 10

//Деконструкция(декомпозиция) кортежа на отдельные переменные
var (name, age) = ("Tom", 23);
Console.WriteLine(name);    // Tom
Console.WriteLine(age);     // 23
//Деконструкция с отбрасыванием ненужного элемента
var (name, _) = ("Tom", 23);
```

Возврат ValueTyple
```csharp
public static (int sum, int count) DoStuff(IEnumerable<int> values) 
{
    var res = (sum: 0, count: 0);
    foreach (var value in values) { res.sum += value; res.count++; }
    return res;
}
public static void Main()
{
	var result = DoStuff(Enumerable.Range(0, 10));
	Console.WriteLine(result.sum);
	
	//или

	var (sum, count) = DoStuff(Enumerable.Range(0, 10));
	Console.WriteLine(sum);
}
```

Отбрасывание при деконструкции элементов
```csharp

static (int, string, string) GetPerson() 
{
    return (Id:1, FirstName: "Bill", LastName: "Gates");
}

(var id, var FName, _) = GetPerson(); 
```

|Name|Access modifier|Type|Custom member name|Deconstruction support|Expression tree support*|
|---|---|---|---|---|---|
|Anonymous types|internal|class|✔️|❌|✔️|
|Tuple|public|class|❌|❌|✔️|
|ValueTuple|public|struct|✔️|✔️|❌|

*Поддержка дерева выражений

ValueType поддерживают дерево выражений,но только с явным вызовом метода
```csharp
Expression<Func<int, int, object>> f = (x, y) => ValueTuple.Create(x, y);
```

В таблице подразумевается использование неявного создания кортежа:
Ошибка: Дерево выражений не может содержать литералы кортежей
```csharp
Expression<Func<int, int, object>> f = (x, y) => (x, y) //error
```


https://www.tutorialsteacher.com/csharp/valuetuple
https://habr.com/ru/articles/345376/
https://stackoverflow.com/questions/2613829/anonymous-type-and-tuple
https://stackoverflow.com/questions/41084411/whats-the-difference-between-system-valuetuple-and-system-tuple