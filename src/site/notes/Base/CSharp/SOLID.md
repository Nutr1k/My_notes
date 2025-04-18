---
{"dg-publish":true,"permalink":"/base/c-sharp/solid/"}
---

Принципы ООП для разработки качественного по

### S: Single Responsibility Principle (Принцип единственной ответственности).

**Каждый компонент(класс, структура, метод) должен иметь одну и только одну причину для изменения**, а это означает, что у него должна быть единая, чётко определённая ответственность.

«Проектируя классы, мы должны стремиться к тому, чтобы объединять родственные компоненты, то есть такие, изменения в которых происходят по одним и тем же причинам. Нам следует стараться разделять компоненты, изменения в которых вызывают различные причины».

**Когда применять принцип:**
Если компонент выполняет несколько различных действий, и они изменяются по отдельности, то это как раз тот случай, когда можно применить принцип единственной ответственности(обязанности). То есть иными словами, у компонента несколько причин для изменения.


**Распространенные случаи отхода от принципа SRP:**
Нередко принцип единственной обязанности нарушает при смешивании в одном классе функциональности разных уровней. Например, класс производит вычисления и выводит их пользователю, то есть соединяет в себя бизнес-логику и работу с пользовательским интерфейсом. Либо класс управляет сохранением/получением данных и выполнением над ними вычислений, что также нежелательно. 

Класс следует применять только для одной задачи - либо бизнес-логика, либо вычисления, либо работа с данными.

> [!success] Важно 
> Когда мы проектируем класс, мы должны понимать какие причины для его изменения могут возникнуть. Причина для изменения должна быть только одна.
> Если мы обнаруживаем к примеру, что при изменении хранилища, способа передачи данных нам приходиться изменять класс, который представляет собой сущность для хранение информации, то мы нарушаем принцип SRP и класс нужно отрефакторить



```ad-note
title:Распространенные сценарии выделения компонентов
collapse:
Есть ряд распространенных сценариев, которые обычно выносятся в отдельные компоненты:
- Логика хранения данных
- Валидация
- Механизм уведомлений пользователя
- Обработка ошибок
- Логгирование
- Выбор класса или создание его объекта
- Форматирование
- Парсинг
- Маппинг данных
```

```ad-note
title:Пример
collapse:
**Пример:**
Данный класс нарушает принцип единственной ответственности т.к. в соответствии с принципом единственной ответственности класс должен решать лишь какую-то одну задачу. 

Он же решает две:
1. Работой с хранилищем данных в методе `saveAnimal`
2. Манипуляция свойствами объекта в конструкторе и в методе `getAnimalName`.
```csharp
class Animal
{
	public Animal(string name) { }
	public string getAnimalName() { }
	public void saveAnimal(Animal a) { }
}

В соответствие с этим у нас могут возникнуть две причины для изменения кода:
1. Если измениться порядок работы хранилищем данных
2. Если изменяться порядок работы нашего класса

**Решение:**
Создадим ещё один класс, единственной задачей которого является работа с хранилищем


class Animal
{
	public Animal(string name) { }
	public string getAnimalName() { }
}
class AnimalDB
{
	public Animal getAnimal(int id) { }
	public void saveAnimal(Animal a) { }
}


Теперь причина изменение каждого класса только одна.
```



### O: Open-Closed Principle (Принцип открытости-закрытости).
**Программные сущности/объекты(класс, модули) должны быть открыты для расширения и закрыты для изменения.** 

Суть этого принципа состоит в том, что система должна быть построена таким образом, что все ее последующие изменения должны быть реализованы с помощью **добавления нового кода, а не изменения уже существующего**.

Иными словами, должна иметься возможность расширять поведение программных сущностей без **их** непосредственного изменения.

```ad-note
title:Пример
collapse:

Повар может готовить только картошку, но в дальнейшем функционал будет расширен и будут добавлены новые блюда
```csharp
class Cook
{
	public string Name { get; set; }
	public Cook(string name)
	{
		this.Name = name;
	}

	public void MakeDinner()
	{
		Console.WriteLine("Чистим картошку");
		Console.WriteLine("Ставим почищенную картошку на огонь");
		Console.WriteLine("Картофельное пюре готово");
	}
}

public static void Main()
{
	Cook bob = new Cook("Bob");
	bob.MakeDinner();
}

Однако одного умения готовить картофельное пюре недостаточно. В дальнейшем набор блюд может увеличиться. И в этом случае мы подходим **к необходимости изменения функционала класса**, а именно метода MakeDinner. Но в соответствии с рассматриваемым нами принципом классы должны быть **открыты для расширения**, но **закрыты для изменения**. То есть, нам надо сделать класс Cook отрытым для расширения, но при этом не изменять.

Для решения этой задачи мы можем воспользоваться паттерном **Стратегия**. В первую очередь нам надо вынести из класса и инкапсулировать всю ту часть, которая представляет изменяющееся поведение. В нашем случае это метод MakeDinner.

csharp
class Cook
{
	public string Name { get; set; }

	public Cook(string name)
	{
		this.Name = name;
	}

	public void MakeDinner(IMeal meal)
	{
		meal.Make();
	}
}

interface IMeal
{
	void Make();
}
class PotatoMeal : IMeal
{
	public void Make()
	{
		Console.WriteLine("Чистим картошку");
		Console.WriteLine("Ставим почищенную картошку на огонь");
		Console.WriteLine("Картофельное пюре готово");
	}
}
class SaladMeal : IMeal
{
	public void Make()
	{
		Console.WriteLine("Нарезаем помидоры и огурцы");
		Console.WriteLine("Посыпаем зеленью, солью и специями");
		Console.WriteLine("Салат готов");
	}
}


Теперь приготовление еды абстрагировано в интерфейсе IMeal, а конкретные способы приготовления определены в реализациях этого интерфейса. А класс Cook делегирует приготовление еды методу Make объекта IMeal.


Cook bob =new Cook("Bob");
bob.MakeDinner(new PotatoMeal());
Console.WriteLine();
bob.MakeDinner(new SaladMeal());
```


### L: Liskov Substitution Principle (Принцип подстановки Барбары Лисков).

**Наследующий класс должен дополнять, а не замещать поведение базового класса.**
Принцип LSP используется когда код использует объект через интерфейс или базовый класс, он предполагает, что поведение этого объекта будет соответствовать контракту, заданному базовым классом.

LSP- Этот принцип не про реализацию кода в методах, а про не нарушение их поведения для внешнего окружения. То есть, есть код который вызывает у объекта Аккаунт какой-либо метод, то и все наследники должны иметь такой же и с таким же поведением (но не реализацией). Пре и постусловия это, грубо говоря, про количество аргументов и их типы, и аналогично про возвращаемое значение


Вы должны иметь возможность использовать любой производный класс там где используется базовый класс и иметь возможность вести себя с ним таким же образом как и с базовым классом без внесения изменений.

Код, который использует базовый класс, не должен «ломаться» или вести себя неожиданным образом при подстановке производного класса.

Функции, которые используют ссылки на базовые классы, должны иметь возможность использовать объекты производных классов, не зная об этом

Описание принципа Лисков полностью основано на принципах проектирования по контракту.
#### Производные классы не должны усиливать предусловия (не должны требовать большего от своих клиентов).
Предусловия представляют набор условий, необходимых для безошибочного выполнения метод
```csharp
public virtual void SetCapital(int money)
{
	if (money < 0)
		throw new Exception("Нельзя положить на счет меньше 0");
	this.Capital = money;
}
```
Здесь условное выражение(if) служит предусловием - без его выполнения не будут выполняться остальные действия, а метод завершится с ошибкой.

Пример нарушения принципа Лисков при добавляет дополнительное предусловие:
```csharp
class MicroAccount : Account
{
	public override void SetCapital(int money)
	{
		if (money < 0)
			throw new Exception("Нельзя положить на счет меньше 0");

		if (money > 100)
			throw new Exception("Нельзя положить на счет больше 100");

		this.Capital = money;
	}
}
```

#### Производные классы не должны ослаблять постусловия (должны гарантировать(требовать), как минимум тоже, что и базовый класс).
Постусловия проверяют состояние возвращаемого объекта на выходе из функции.

```csharp
public static float GetMedium(float[] numbers)
{
	if (numbers.Length == 0)
		throw new Exception("длина массива равна нулю");

	float result = numbers.Sum() / numbers.Length;
	//Постусловие
	if (result < 0)
		throw new Exception("Результат меньше нуля");
	return result;
}
```
Пример нарушения принципа Лисков при ослаблении постусловия:
```csharp
class Account
{
	public virtual decimal GetInterest(decimal sum, int month, int rate)
	{
		// предусловие
		if (sum < 0 || month > 12 || month < 1 || rate < 0)
			throw new Exception("Некорректные данные");

		decimal result = sum;
		for (int i = 0; i < month; i++)
			result += result * rate / 100;

		// постусловие
		if (sum >= 1000)
			result += 100; // добавляем бонус

		return result;
	}
}

class MicroAccount : Account
{
	public override decimal GetInterest(decimal sum, int month, int rate)
	{
		if (sum < 0 || month > 12 || month < 1 || rate < 0)
			throw new Exception("Некорректные данные");

		decimal result = sum;
		for (int i = 0; i < month; i++)
			result += result * rate / 100;

		//Тут нед посусловия
		return result;
	}
}
```
#### Производные классы не должны нарушать инварианты базового класса (инварианты базового класса и наследников суммируются)
Инвариант — это условие который всегда истинный.
В коде инварианты чаще всего никак не выражены, но иногда ставятся защитные проверки которые их проверяют.
Примеры инвариантов.
`List<>`: 0 ≤ _size ≤ _items.Length
`List<>.Enumerator`: list.version = const = version; есть защитная проверка
`List<>.Enumerator`: 0 ≤ index ≤ list._size+

Инварианты - это либо (чаще) условие которое остается истинным после вызова любых методов объекта в любой последовательности, либо (реже) выражение которое сохраняет свое значение после вызова любых методов. То есть это основополагающие класса, при нарушении которых класс теряет свою суть и им невозможно пользоваться


#### Производные классы не должны генерировать исключения, не описанные базовым классом.

Т.к. вызывающий код просто не будет знак как обработать эти исключения.



### I: Interface Segregation Principle (Принцип разделения интерфейса).
Данный принцип относится к тем случаям, когда классы имеют **"жирный интерфейс"**, то есть слишком раздутый интерфейс, не все методы и свойства которого используются и могут быть востребованы его реализатором. Таким образом, интерфейс получатся слишком избыточен или "жирным".

Клиенты не должны вынужденно зависеть от методов, которыми не пользуются.


При нарушении этого принципа клиент, использующий некоторый интерфейс со всеми его методами и вынужден реализовывать эти методы, которыми не пользуется.

Если мы столкнулись с этим, то:
1. Нарушается принципом единой ответственности т.к. компоненты в интерфейсах слабо согласованы и мы вынуждены реализовывать, то что нам не надо. В частности оставлять реализацию пустой(ставить заглушку)
2. Если мы оставляем реализацию пустой, то мы нарушаем принцип подстановки Лисков). То есть используя наш объект по ссылке интерфейса мы можем наткнуться на ту версию, где данный метод представляет пустую реализацию.

Пример:
Мы создали интерфейс `IPhone`, описывающий функции телефона и реализовали данный интерфейс в классе `Phone`.
```csharp
interface IPhone
{
	void Call();
	void TakePhoto();
	void MakeVideo();
	void BrowseInternet();
}
class Phone : IPhone
{
	public void Call() => Console.WriteLine("Звоним");
	public void TakePhoto() => Console.WriteLine("Фотографируем");
	public void MakeVideo() => Console.WriteLine("Снимаем видео");
	public void BrowseInternet() => Console.WriteLine("Серфим в интернете");
}
```

В будущем мы решили создать класс `Camera` и решили не создавать новый интерфейс, а использовать существующий.

```csharp
class Camera : IPhone
{
	public void Call() { }
	public void TakePhoto()
	{
		Console.WriteLine("Фотографируем");
	}
	public void MakeVideo() { }
	public void BrowseInternet() { }
}
```
Клиент - класс Camera зависит от методов, которые он не использует - то есть методов `Call, MakeVideo, BrowseInternet`.
В дальнейшем если мы будем использовать данный класс через ссылку интерфейса, то может наткнуться на пустые методы, которые ничего не делают. В более худшей ситуации когда метод возвращает какое-то значение - это может привести к непредсказуемым последствиям.

Для решения возникшей задачи мы можем воспользоваться принципом разделения интерфейсов:

```csharp
interface ICall
{
	void Call();
}
interface IPhoto
{
	void TakePhoto();
}
interface IVideo
{
	void MakeVideo();
}
interface IWeb
{
	void BrowseInternet();
}

class Camera : IPhoto
{
	public void TakePhoto()
	{
		Console.WriteLine("Фотографируем с помощью фотокамеры");
	}
}

class Phone : ICall, IPhoto, IVideo, IWeb
{
	public void Call()
	{
		Console.WriteLine("Звоним");
	}
	public void TakePhoto()
	{
		Console.WriteLine("Фотографируем с помощью смартфона");
	}
	public void MakeVideo()
	{
		Console.WriteLine("Снимаем видео");
	}
	public void BrowseInternet()
	{
		Console.WriteLine("Серфим в интернете");
	}
}
```
### D: Dependency Inversion Principle (Принцип инверсии зависимостей).
Cлужит для создания слабосвязанных сущностей, которые легко тестировать, модифицировать и обновлять.

1. Модули верхнего уровня не должны зависеть от модулей нижнего уровня. И те и другие должны зависеть от абстракций.
2. Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций.

Это значит, что если детали меняются, они не должны влиять на абстракцию. Абстракция — это то, как клиенты видят объект. Что именно происходит внутри объекта, не важно. Возьмем, к примеру, автомобиль: педали, руль и рычаг переключения передач — это абстракции того, что происходит внутри двигателя. Они не зависят от деталей, потому что если кто-то заменит мой старый двигатель на новый, я все равно смогу водить машину, не зная, что двигатель изменился.

С другой стороны, детали ДОЛЖНЫ соответствовать тому, что говорит абстракция. Я бы не хотел реализовывать двигатель, который внезапно заставляет тормоза удваивать скорость автомобиля. Я могу перереализовывать тормоза любым способом, который захочу, пока внешне они ведут себя так же.


В процессе разработки программного обеспечения существует момент, когда функционал приложения перестаёт помещаться в рамках одного модуля. 
Когда это происходит, нам приходится решать проблему зависимостей модулей. В результате, например, может оказаться так, что** высокоуровневые компоненты зависят от низкоуровневых компонентов.**

```csharp
// Базовый класс для работы с HTTP-запросами
public class HttpRequestService
{
	protected readonly HttpClient HttpClient;

	public HttpRequestService()
	{
		HttpClient = new HttpClient
		{
			Timeout = TimeSpan.FromSeconds(30)
		};
	}

}

// Производный класс, добавляющий работу с XML
public class XmlHttpService : HttpRequestService
{
	public async void RequestXml(string url, string xmlData)
	{
		// Логика для отправки XML-данных
		var content = new StringContent(xmlData, Encoding.UTF8, "application/xml");
		HttpResponseMessage response = await HttpClient.PostAsync(url, content);
		Console.WriteLine($"Request: {url} with XML data: {xmlData}");
	}
}

public class Http
{
	private readonly XmlHttpService _xmlhttpService;

	public Http(XmlHttpService xmlhttpService)
	{
		_xmlhttpService = xmlhttpService;
	}

	public void Get(string url, string content)
	{
		_xmlhttpService.RequestXml(url, content);
	}

	public void Post(string url, string content)
	{
		_xmlhttpService.RequestXml(url, content);
	}
}
```
Класс `Http` представляет собой высокоуровневый компонент
Класс `XMLHttpService` — низкоуровневый. 
Такая архитектура нарушает пункт №1 принципа инверсии зависимостей: 
«Модули верхних уровней не должны зависеть от модулей нижних уровней. Оба типа модулей должны зависеть от **абстракций**».

Класс `Http` вынужденно зависит от класса `XMLHttpService`. Если мы решим изменить механизм, используемый классом `Http` для взаимодействия с сетью — скажем, это будет `JSON` формат или, например, сервис-заглушка, применяемый для целей тестирования, нам придётся отредактировать все экземпляры класса `Http`, изменив соответствующий код. Это нарушает принцип открытости-закрытости.

Класс `Http` не должен знать о том, что и какой формат данных именно используется для организации сетевого соединения.

```csharp
public interface IConnection
{
	void Request(string url, string content);
}
```

Интерфейс `Connection` содержит описание метода `Request` и мы передаём классу `Http` аргумент типа `Connection`:

```csharp
public class Http
{
	private readonly IConnection _httpConnection;

	public Http(IConnection httpConnection)
	{
		_httpConnection = httpConnection;
	}

	public void Get(string url, string content)
	{
		_httpConnection.Request(url, content);
	}

	public void Post(string url, string content)
	{
		_httpConnection.Request(url, content);
	}
}
```

Теперь, вне зависимости от того, что именно используется для организации взаимодействия с сетью, класс `Http` может пользоваться тем, что ему передали, не заботясь о том, что скрывается за интерфейсом `Connection`.


Перепишем класс `XMLHttpService` таким образом, чтобы он реализовывал этот интерфейс:

```csharp
public class XmlHttpService : HttpRequestService,IConnection
{
	public async void Request(string url, string xmlData)
	{
		// Логика для отправки XML-данных
		var content = new StringContent(xmlData, Encoding.UTF8, "application/xml");
		HttpResponseMessage response = await HttpClient.PostAsync(url, content);
		Console.WriteLine($"Request: {url} with XML data: {xmlData}");
	}
}

```

В результате мы можем создать множество классов, реализующих интерфейс `IConnection` и подходящих для использования в классе `Http` для организации обмена данными по сети:

```csharp
public class JsonHttpService : HttpService,IConnection
{
	public void Request(string url, string method, HttpContent content)
	{
		// Логика для отправки JSON-данных
	}
}

public class SoaHttpService : HttpService,IConnection
{
	public void Request(string url, string method, HttpContent content)
	{
		// Логика для отправки JSON-данных
	}
}
```

Как можно заметить, здесь высокоуровневые и низкоуровневые модули зависят от абстракций. Класс `Http` (высокоуровневый модуль) зависит от интерфейса `IConnection` (абстракция). Классы `XMLHttpService`, `JsonHttpService` и `SoaHttpService` (низкоуровневые модули) также зависят от интерфейса `IConnection`.  

```csharp
public static class Program
{
	public static void Main()
	{
		IConnection xmlHttpService = new XmlHttpService();
		Http http = new Http(xmlHttpService);
	}
}
```


Кроме того, стоит отметить, что следуя принципу инверсии зависимостей, мы соблюдаем и принцип подстановки Барбары Лисков. А именно, оказывается, что типы `XMLHttpService`, `JsonHttpService` и `SoaHttpService` могут служить заменой базовому типу `Connection`.

[SOLID ASP.NET](https://www.telerik.com/blogs/aspnet-core-basics-understanding-practicing-solid)