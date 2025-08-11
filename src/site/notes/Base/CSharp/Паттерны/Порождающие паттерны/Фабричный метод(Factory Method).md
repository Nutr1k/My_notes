---
dg-publish: true
---
Фабричный метод (Factory Method) - это паттерн, который определяет интерфейс для создания объектов некоторого класса, но непосредственное решение о том, объект какого класса создавать происходит в подклассах. То есть паттерн предполагает, что базовый класс делегирует создание объектов классам-наследникам.

![Фабричный_метод.drawio.png](/img/user/Files/Image/%D0%A4%D0%B0%D0%B1%D1%80%D0%B8%D1%87%D0%BD%D1%8B%D0%B9_%D0%BC%D0%B5%D1%82%D0%BE%D0%B4.drawio.png)
![[Фабричный_метод 1.drawio]]

![Фабричный метод №2.drawio 1.png](/img/user/Files/Image/%D0%A4%D0%B0%D0%B1%D1%80%D0%B8%D1%87%D0%BD%D1%8B%D0%B9%20%D0%BC%D0%B5%D1%82%D0%BE%D0%B4%20%E2%84%962.drawio%201.png)
![[Фабричный_метод №2.drawio]]

- Абстрактный класс Product определяет интерфейс класса, объекты которого надо создавать.
- Конкретные классы ConcreteProductA и ConcreteProductB представляют реализацию класса Product. Таких классов может быть множество
- Абстрактный класс Creator определяет абстрактный фабричный метод `FactoryMethod()`, который возвращает объект Product.
- Конкретные классы ConcreteCreatorA и ConcreteCreatorB - наследники класса Creator, определяющие свою реализацию метода `FactoryMethod()`. Причем метод `FactoryMethod()` каждого отдельного класса-создателя возвращает определенный конкретный тип продукта. Для каждого конкретного класса продукта определяется свой конкретный класс создателя.
    
Таким образом, **класс Creator делегирует создание объекта Product своим наследникам `ConcreteCreatorA и ConcreteCreatorB`**. А классы ConcreteCreatorA и ConcreteCreatorB могут самостоятельно выбирать какой конкретный тип продукта им создавать.

### Когда надо применять паттерн

- Когда заранее неизвестно, объекты каких типов необходимо создавать
- Когда система должна быть независимой от процесса создания новых объектов и расширяемой: в нее можно легко вводить новые классы, объекты которых система должна создавать.
- Когда создание новых объектов необходимо делегировать из базового класса классам наследникам

### Пример


```csharp
#region Абстрактные класс (Product)

public abstract class Mage//Product
{
	public abstract void CastSpell();
}
#endregion

#region Конкретный продукт (ConcreteProduct)

public class FireMage : Mage //ConcreteProductA
{
	private float _fireBallRadius;
	private float _fireBallDamage;
	public void Init(float fireBallRadius, float fireBallDamage)
	{
		_fireBallRadius = fireBallRadius;
		_fireBallDamage = fireBallDamage;
	}
	public override void CastSpell()
	{
		Console.WriteLine("Красный маг наложил заклинание");
	}
}

public class IceMage : Mage //ConcreteProductB
{
	private float _slowKoef;

	public void Init(float slowKoef)
	{
		_slowKoef = slowKoef;
	}

	public override void CastSpell()
	{
		Console.WriteLine("Синий маг наложил заклинание");
	}
}

public class GreenMage : Mage //ConcreteProductC
{
	private float _koef1;
	private float _koef2;
	public void Init(float koef1, float koef2)
	{
		_koef1 = koef1;
		_koef2 = koef2;
	}
	public override void CastSpell()
	{
		Console.WriteLine("Зелёный маг наложил заклинание");
	}
}
#endregion

#region Cоздатель, объявляет фабричный метод (Creator)
public abstract class Creator
{
	// Фабричный метод, который будет реализован в подклассах
	public abstract Mage FactoryMethod();//FactoryMethod

	// Общая логика для работы с магами
	public void PerformMageAction()//AnOperation()
	{
		// Создание продукта через фабричный метод
		// Делегирование создания объекта подклассу реализующему Creator
		Mage mage = FactoryMethod();

		// Выполнение операции над продуктом
		mage.CastSpell();
	}
}
#endregion

#region Конкретный создатель (ConcreteCreator)
public class FireCreator : Creator //ConcreteCreatorA
{
	public override Mage FactoryMethod()
	{
		var prefab = "Prefabs/mage_red";

		var mage = new FireMage();
		mage.Init(1.0f, 5.0f);
		return mage;
	}
}

public class IceCreator : Creator //ConcreteCreatorB
{
	public override Mage FactoryMethod()
	{
		var prefab = "Prefabs/mage_blue";

		var mage = new IceMage();
		mage.Init(1.4f);
		return mage;
	}
}

public class ForestCreator : Creator //ConcreteCreatorC
{
	public override Mage FactoryMethod()
	{
		var prefab = "Prefabs/mage_green";

		var mage = new GreenMage();
		mage.Init(1.8f, 4.5f);
		return mage;
	}
}
#endregion

internal class Program
{
	static void Main()
	{
		Console.WriteLine("r - маги огня");
		Console.WriteLine("i - маги воды");
		Console.WriteLine("f - маги земля\n");
		
		string fraction = Console.ReadLine();
		//creator получает конкретную 
		//реализацию(FireCreator,IceCreator,ForestCreator)
		Creator creator = GetFactory(fraction);

//Объект здесь вызывает свой собственный метод фабрики.Поэтому единственное,
//что может изменить возвращаемое значение, — это подкласс.
//Поэтому для создания объектов используется наследование
//Подклассы FireCreator,IceCreator,ForestCreator - определяют создание 
//объекта
		creator.PerformMageAction();
		//или

		Mage mage=creator.FactoryMethod();

		mage.CastSpell();
		
		Main();

		Console.ReadLine();
	}

	private static Creator GetFactory(string fractionName)
	{
		return fractionName.ToLower() switch
		{
			"r" => new FireCreator(),
			"i" => new IceCreator(),
			"f" => new ForestCreator(),
			_ => null
		};
	}
}
```

> [!success] Наследование
> **Клиент** (`Main`) использует базовый класс `Creator` и вызывает общий метод `PerformMageAction`. Сам `PerformMageAction` вызывает `FactoryMethod` для создания объекта, а реализация этого метода находится в подклассе (`ConcreteCreator`).
Таким образом, ответственность за создание объекта делегируется подклассу. 
Это делегирование основано на механизме **наследования**. 

`Creator creator = GetFactory(fraction);` получает конкретную реализацию(`FireCreator,IceCreator,ForestCreator`). Это и есть делегирование основанное на механизме наследования.




Пример из GoF:
![Pasted image 20250118185419.png](/img/user/Files/Image/Pasted%20image%2020250118185419.png)

Участники 
- Product (Document) – продукт: 
	- определяет интерфейс объектов, создаваемых фабричным методом; 
- ConcreteProduct (MyDocument) – конкретный продукт: 
	- реализует интерфейс Product;
- Creator (Application) – создатель: 
	- объявляет фабричный метод, возвращающий объект типа Product. 
	Creator может также определять реализацию по умолчанию фабричного метода, который возвращает объект ConcreteProduct; 
	- может вызывать фабричный метод для создания объекта Product. 
- ConcreteCreator (MyApplication) – конкретный создатель: 
	- замещает фабричный метод, возвращающий объект СoncreteProduct.