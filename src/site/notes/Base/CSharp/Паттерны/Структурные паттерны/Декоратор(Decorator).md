---
{"dg-publish":true,"permalink":"/base/c-sharp/patterny/strukturnye-patterny/dekorator-decorator/"}
---

**Декоратор** — это структурный паттерн проектирования, который позволяет динамически добавлять объекту новую функциональность, посредством оборачивая их в полезные «обёртки». То есть наше действия должны быть направленны на расширение функционала исходного объекта, а не изменение его сути.

![Pasted image 20250121025200.png](/img/user/Files/Image/Pasted%20image%2020250121025200.png)

Пример выстроения логики для реализации паттерна "Декоратор"
1. Есть человек умеющий одевать `майку`
Мы хотим добавить возможность надевания `свитера`
2. Человек теперь умеет после майки надевать `свитер`
Мы хотим добавить возможность надевания куртки
3. Человек теперь умеет после свитера надевать `куртку`

Первая базовая возможность никуда не делась. Мы не можем следуя паттерну декоратор сделать чтобы человек одевал только `свитер` и `куртку`. Мы расширяем функционал исходного объекта, а исходный объект у нас `человек умеющий надевать майку`.

> [!success]  Паттерн
> Данный паттерн следует использовать тогда, все дальнейшие действия будут связанны с первоначальным объектом и все дальнейшие действия будут бессмысленны если убрать первоначальный объект.

Есть у нас первоначальный объект `FileDataSource` с двумя методами на чтение и запись. Мы решили создать два декоратора, один шифрование данных с последующей записью/чтением, второй декоратор на сжимание данных с последующей записью/чтением.

Существование этих двух декораторов невозможно без первоначального объекта `FileDataSource`. 

Что делает декоратор шифрования:
1. Шифрует полученные данные и потом вызывает базовый метод записи класса `FileDataSource`
2. Расшифровывает полученные данные и вызывает базовый метода чтения класса `FileDataSource`

То есть мы специфическим образом изменяем поведение базового класса, но каждый из методов всё равно связан с функционалом базового класса.




### Пример

```cs
// ID: custom-block
public class LogEntry//Клас для тестов, не несёт смысловой нагрузки
{
}
public interface ILogSaver //Component
{
	Task SaveLogEntry(string applicationId, LogEntry logEntry);
}

public sealed class ElasticsearchLogSaver : ILogSaver //ConcreteComponent
{
	public Task SaveLogEntry(string applicationId, LogEntry logEntry)
	{
		// Сохраняем переданную запись в Elasticsearch
		return Task.FromResult<object>(null);
	}
}
public abstract class LogSaverDecorator : ILogSaver //Decorator 
{
	protected readonly ILogSaver _decoratee;
	protected LogSaverDecorator(ILogSaver decoratee)
	{
		_decoratee = decoratee;
	}
	public abstract Task SaveLogEntry(string applicationId, LogEntry logEntry);
}


public class ThrottlingLogSaverDecorator : LogSaverDecorator//ConcreteDecoratorA
{
	public ThrottlingLogSaverDecorator(ILogSaver decoratee) : base(decoratee)
	{ }
	public override async Task SaveLogEntry(string applicationId, LogEntry logEntry)
	{
		if (!QuotaReached(applicationId))
		{
			IncrementUsedQuota();
			// Сохраняем записи. Обращаемся к декорируемому объекту!
			await _decoratee.SaveLogEntry(applicationId, logEntry);
			return;
		}

		// Сохранение невозможно! Лимит приложения исчерпан!
		throw new Exception();
	}
	private bool QuotaReached(string applicationId)
	{
		return false;
	}
	private void IncrementUsedQuota()
	{
		//...
	}
}

class Program
{
	static void Main(string[] args) //Client
	{
		ILogSaver logSaver = new ElasticsearchLogSaver();

		logSaver.SaveLogEntry(Process.GetCurrentProcess().ProcessName, new LogEntry());

		//Добавляем декоратор для ограничения числа запросов
		logSaver = new ThrottlingLogSaverDecorator(new ElasticsearchLogSaver());

		logSaver.SaveLogEntry(Process.GetCurrentProcess().ProcessName, new LogEntry());

		//Добавляем ещё один декоратор для замера времени

		logSaver = new ThrottlingLogSaverDecorator(new TraceLogSaverDecorator(new ElasticsearchLogSaver()));

		logSaver.SaveLogEntry(Process.GetCurrentProcess().ProcessName, new LogEntry());
	}
}

// Декоратор для замера времени исполнения метода
public class TraceLogSaverDecorator : LogSaverDecorator ////ConcreteDecoratorB
{
	public TraceLogSaverDecorator(ILogSaver decoratee) : base(decoratee)
	{ }

	public override async Task SaveLogEntry(string applicationId, LogEntry logEntry)
	{
		var sw = Stopwatch.StartNew();
		try
		{
			await _decoratee.SaveLogEntry(applicationId, logEntry);
		}
		finally
		{
			Console.WriteLine("Операция сохранения завершена за {0} мс",  sw.ElapsedMilliseconds);
			Trace.TraceInformation("Операция сохранения завершена за {0} мс", sw.ElapsedMilliseconds);
		}
	}
}

```


- Component (`ILogSaver`) — базовый класс компонента, чье поведение будет расширяться декораторами; 
- Client (`Main`) — работает с компонентом, не зная о существовании декораторов; 
- ConcreteComponent (`ElasticsearchLogSaver`) — конкретная реализация компонента; 
- Decorator (`LogSaverDecorator`) — базовый класс декоратора, предназначенный для расширения поведения компонента; 
- ConcreteDecoratorA (`ThrottlingLogSaverDecorator`) — конкретный декоратор, который добавляет декорируемому объекту специфическое поведение.



Давайте рассмотрим задачу импорта логов несколько с иной стороны. 

Помимо консольной утилиты или локального Windows-сервиса, мы можем разработать облачное приложение, которое будет принимать трассировочные сообщения от приложений пользователя и предоставлять интерфейс для последующего поиска. 

Большинство облачных приложений не позволяют конкретному пользователю передавать произвольное количество данных. При достижении определенного лимита, например десять сообщений в секунду от конкретного пользователя, включается режим ограничений входящих запросов (**throttling**). 

В простом случае логику троттлинга можно смешать с логикой сохранения, но такое решение сложно поддерживать в длительной перспективе. Вместо этого можно воспользоваться декоратором (листинг 14.1).

> [!success] Метод ThrottlingLogSaverDecorator
> Идея паттерна «Декоратор» в том, что у интерфейса (`ILogSaver`) появляется два вида реализаций: основная реализация бизнес-функциональности (`ElasticsearchLogSaver`) и набор классов-декораторов, которые реализуют тот же интерфейс, но довольно специфическим образом. Декоратор принимает в конструкторе тот же самый интерфейс, а в реализации делегирует работу декорируемому объекту с присоединением некоторого поведения до или после вызова метода 

Декоратор с ограничением числа вызовов (`ThrottlingLogSaverDecorator`) вначале проверяет, не исчерпано ли число запросов со стороны текущего приложения, и лишь затем делегирует работу основному объекту (полю `decorate`). 

Если квота не достигла лимита, то вызывается основной метод и данные успешно сохраняются. В противном случае генерируется исключение `QuotaReachedException`, которое прикажет клиенту попробовать выполнить тот же самый запрос через некоторое время