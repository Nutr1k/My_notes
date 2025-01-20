---
{"dg-publish":true,"permalink":"/base/c-sharp/patterny/porozhdayushhie-patterny/abstraktnaya-fabrika-abstract-factory/"}
---

Абстрактная фабрика(Abstract Factory) - предоставляет интерфейс для создания **семейств взаимосвязанных объектов или родственных объектов**, но не определяет, какие именно классы должны быть использованы для их создания.

То есть, абстрактная фабрика описывает, какие объекты можно создать, но не говорит, как именно они должны быть реализованы. Конкретные фабрики (классы, которые реализуют интерфейс абстрактной фабрики) отвечают за создание конкретных объектов.

Другими словами: абстрактная фабрика представляет собой стратегию создания семейства взаимосвязанных или родственных объектов.

В паттерне абстрактная фабрика акцент делается на создании **семейств объектов**, и это отличает его от других порождающих паттернов, создающих только один какой-то вид объектов


![Абстрактная фабрика.drawio (3).png](/img/user/Files/Image/%D0%90%D0%B1%D1%81%D1%82%D1%80%D0%B0%D0%BA%D1%82%D0%BD%D0%B0%D1%8F%20%D1%84%D0%B0%D0%B1%D1%80%D0%B8%D0%BA%D0%B0.drawio%20(3).png)
![[Абстрактная-фабрика.drawio]]

- Абстрактные классы `AbstractProductA` и `AbstractProductB` определяют интерфейс для классов, объекты которых будут создаваться в программе.
- Конкретные классы ProductA1 / ProductA2 и ProductB1 / ProductB2 представляют конкретную реализацию абстрактных классов
- Абстрактный класс фабрики `AbstractFactory` определяет методы для создания объектов. Причем методы возвращают абстрактные продукты, а не их конкретные реализации.
- Конкретные классы фабрик ConcreteFactory1 и ConcreteFactory2 реализуют абстрактные методы базового класса и непосредственно определяют какие конкретные продукты использовать
- Класс клиента Client использует класс фабрики для создания объектов. При этом он использует исключительно абстрактный класс фабрики `AbstractFactory` и абстрактные классы продуктов `AbstractProductA` и `AbstractProductB` и никак не зависит от их конкретных реализаций


### Когда надо применять паттерн

- Когда система не должна зависеть от способа создания и компоновки новых объектов
- Когда создаваемые объекты должны использоваться вместе и являются взаимосвязанными



### Пример
```csharp
#region Общие абстрактные класс (AbstractProduct)

public abstract class Knight//AbstractProductA
{
	public abstract void Parry();
}
public abstract class Archer//AbstractProductB
{
	public abstract void Shoot();
}
public abstract class Mage//AbstractProductC
{
	public abstract void CastSpell();
}
#endregion


#region Конкретные продукты (Product)

#region ProductA
public class RedKnight : Knight //ProductA1
{
	private float _redKnightKoef;
	public void Init(float redKnightKoef)
	{
		_redKnightKoef = redKnightKoef;
	}
	public override void Parry()
	{
		Console.WriteLine("Красный рыцарь отразил атаку");
	}
}
public class RedMage : Mage //ProductA2
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
public class RedArcher : Archer //ProductA3
{
	private float _fireArrowDamage;
	public void Init(float fireArrowDamage, float rangeDistance)
	{
		_fireArrowDamage = fireArrowDamage;

	}
	public override void Shoot()
	{
		Console.WriteLine("Красный лучник произвёл выстрел");
	}
}
#endregion

#region ProductB
public class BlueKnight : Knight //ProductB1
{
	private float _koef1;
	private float _koef2;
	public void Init(float koef1, float koef2)
	{
		_koef1 = koef1;
		_koef2 = koef2;
	}
	public override void Parry()
	{
		Console.WriteLine("Синий рыцарь отразил атаку");
	}
}
public class BlueMage : Mage //ProductB2
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
public class BlueArcher : Archer //ProductB3
{
	private float _koef1;
	private float _koef2;
	public void Init(float koef1, float koef2)
	{
		_koef1 = koef1;
		_koef2 = koef2;
	}
	public override void Shoot()
	{
		Console.WriteLine("Синий лучник произвёл выстрел");
	}
}
#endregion

#region ProductC
public class GreenKnight : Knight //ProductC1
{
	private float _koef1;
	private float _koef2;
	public void Init(float koef1, float koef2)
	{
		_koef1 = koef1;
		_koef2 = koef2;
	}
	public override void Parry()
	{
		Console.WriteLine("Зелёный рыцарь отразил атаку");
	}
}
public class GreenMage : Mage //ProductC2
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
public class GreenArcher : Archer //ProductC3
{
	private float _koef1;
	private float _koef2;
	public void Init(float koef1, float koef2)
	{
		_koef1 = koef1;
		_koef2 = koef2;
	}
	public override void Shoot()
	{
		Console.WriteLine("Зелёный лучник произвёл выстрел");
	}
}
#endregion

#endregion

#region Абстрактная фабрика (AbstractFactory)
public abstract class UnitsFactory
{
	public abstract Knight CreateKnight();
	public abstract Mage CreateMage();
	public abstract Archer CreateArcher();
}
#endregion

#region Конкретная реализация фабрики для огненных ютиов (ConcreteFactory1)
public class RedUnitsFactory : UnitsFactory
{
	public override Knight CreateKnight()
	{
		var prefab = "Prefabs/knight_red.png";
		var knight = new RedKnight();
		knight.Init(3.3f);
		return knight;
	}
	public override Mage CreateMage()
	{
		var prefab = "Prefabs/mage_red";

		var mage = new RedMage();
		mage.Init(1.0f, 5.0f);
		return mage;
	}
	public override Archer CreateArcher()
	{
		var prefab = "Prefabs/archer_red";

		var archer = new RedArcher();
		archer.Init(1.0f, 1.0f);
		return archer;
	}
}
#endregion

#region Конкретная реализация фабрики для ледяных ютиов (ConcreteFactory2)
public class BlueUnitsFactory : UnitsFactory
{
	public override Knight CreateKnight()
	{
		var prefab = "Prefabs/knight_blue.png";
		var knight = new BlueKnight();
		knight.Init(2.3f,3.3f);
		return knight;
	}
	public override Mage CreateMage()
	{
		var prefab = "Prefabs/mage_blue";

		var mage = new BlueMage();
		mage.Init(1.4f);
		return mage;
	}
	public override Archer CreateArcher()
	{
		var prefab = "Prefabs/archer_blue";

		var archer = new BlueArcher();
		archer.Init(1.3f, 1.5f);
		return archer;
	}
}
#endregion

#region Конкретная реализация фабрики для земляных ютиов (ConcreteFactory3)
public class GreenUnitsFactory : UnitsFactory
{
	public override Knight CreateKnight()
	{
		var prefab = "Prefabs/knight_green.png";
		var knight = new GreenKnight();
		knight.Init(4.3f,2.5f);
		return knight;
	}
	public override Mage CreateMage()
	{
		var prefab = "Prefabs/mage_green";

		var mage = new GreenMage();
		mage.Init(1.8f, 4.5f);
		return mage;
	}
	public override Archer CreateArcher()
	{
		var prefab = "Prefabs/archer_green";

		var archer = new GreenArcher();
		archer.Init(1.7f, 2.5f);
		return archer;
	}
}
#endregion

public class Client
{
	private readonly UnitsFactory _factory; // Композиция фабрики

//В данном классе Client фабрика UnitsFactor передаётся через конструктор и 
//сохраняется как поле.
//Это и есть композиция: клиент зависит от фабрики и делегирует ей создание 
//объектов.
	public Client(UnitsFactory factory)
	{
		_factory = factory; // Фабрика передаётся клиенту(Композиция фабрики)
	}

	public void Run()
	{
		Knight _knight = _factory.CreateKnight();
		Archer _archer = _factory.CreateArcher();
		Mage _mage = _factory.CreateMage();

		_knight.Parry();
		_mage.CastSpell();
		_archer.Shoot();
	}
}

internal class Program
{
	static void Main()
	{
		string factions = Console.ReadLine();

		UnitsFactory currentFactory = GetFactory(factions);

		Client client = new Client(currentFactory);
		
		client.Run();

		Main();

		Console.ReadLine();
	}
	private static UnitsFactory GetFactory(string fractionName)
	{
		return fractionName.ToLower() switch
		{
			"r" => new RedUnitsFactory(),
			"b" => new BlueUnitsFactory(),
			"g" => new GreenUnitsFactory(),
			_ => null
		};
	}
}
```

```ad-success Композиция
title:Фаза №1. Маркировка (marking)
 - **Композиция фабрики в клиенте**: 
    - В классе `Client` фабрика `UnitsFactory` передаётся через конструктор и сохраняется как поле. Это и есть композиция: клиент зависит от фабрики и делегирует ей создание объектов.
- **Делегирование создания объектов**:
    - Вместо того чтобы создавать объекты типа `Knight` или `Archer`, `Mage` напрямую, клиент вызывает методы фабрики `CreateKnight` и `CreateArcher` и `CreateMage`. Таким образом, ответственность за создание продуктов передаётся фабрике. 
```


