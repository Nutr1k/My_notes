---
{"dg-publish":true,"permalink":"/base/c-sharp/patterny/strukturnye-patterny/adapter-adapter/"}
---

**Адаптер** — это структурный паттерн, который позволяет подружить несовместимые объекты.

Цель паттерна - адаптация уже существующего кода. Т.е. ваш код умеет использовать некий интерфейс `IX` для своей цели (играть аудио). Вам надо реализовать его, но есть некий компонент, который делает то что надо, но не реализует `IX`. Вот тут и нужен адаптер. С его помощью вы приводите существующий код к заданному интерфейсу.

Адаптер выступает прослойкой между двумя объектами, превращая вызовы одного в вызовы понятные другому.

Адаптер получает конвертируемый объект в конструкторе или через параметры своих методов. Методы Адаптера обычно совместимы с интерфейсом одного объекта. Они делегируют вызовы вложенному объекту, превратив перед этим параметры вызова в формат, поддерживаемый вложенным объектом.

Главная задача Адаптера – реализация требуемого интерфейса и трансляция его вызовов адаптируемому объекту. Подобную ситуацию можно встретить при использовании сторонних библиотек. Они далеко не всегда предоставляют интерфейсы, которые необходимы в для связи с другими объектами. При этом изменить код или добавить поддержку интерфейса не предоставляется возможным.


### Когда надо применять паттерн
- существующий объект, называемый адаптируемым, предоставляет необходимые функции, но не поддерживает нужного интерфейса;
- (или) неизвестно заранее, с каким интерфейсами придется работать адаптируемому объекту;
- (или) формат входных или выходных данных метода не совпадает с требуемым.


Различают четыре роли, отводимые участвующим в работе шаблона объектам:
1. Адаптируемый объект (Adaptee);
2. Цель (Target), определяющая требуемый интерфейс;
3. Адаптер (Adapter);
4. Клиент (Client), который умеет работать с только объектами, реализующими интерфейс(`Target`) цели.


### Пример

В приложении необходимо воспроизводить звуковые сигналы для оповещения пользователя, которые в данный момент хранятся в формате .wav.

Для этого был создан класс `NotificationService`, он взаимодействует с интерфейсом 
`IAudioPlayer` и имеет метод `SendAlert`, который уведомляет пользователя звуковым сигналом.

Через несколько лет мы решили поменять аудио плеер на другой. Для этого был выбран `SoundPlayer` из `System.Media`. Однако данный класс не поддерживает заданный интерфейс. Поэтому воспользуемся шаблоном Адаптер:

```cs
public interface IAudioPlayer//Target
{
	void Load(string file);

	void Play();
}
public class NotificationService//Client
{
	private IAudioPlayer _audioPlayer;

	public NotificationService(IAudioPlayer audioPlayer)
	{
		_audioPlayer = audioPlayer;
	}

	public void SendAlert(string filePath)
	{
		Console.WriteLine("Отправка уведомления...");
		if (!string.IsNullOrEmpty(filePath))
		{
			_audioPlayer.Load(filePath);  // Загружаем звуковой файл
			_audioPlayer.Play();          // Воспроизводим звуковой сигнал
		} 
	}
}
//Adapter
public class SoundPlayerAdapter : IAudioPlayer 
{
	private readonly SoundPlayer _player = new SoundPlayer();//Adaptee (SoundPlayer = Adaptee)

	public void Load(string file)
	{
		_player.SoundLocation = file;
		_player.Load();
	}

	public void Play()
	{
		_player.Play();
	}
}
internal class Program
{
	static void Main(string[] args)
	{
		SoundPlayer soundPlayer = new SoundPlayer();
		NotificationService notificationService = new NotificationService(new SoundPlayerAdapter());
		notificationService.SendAlert(@"..\..\..\sample-3s.wav");
	}
}
```

Метод `Play()` класса `SoundPlayerAdapter` просто вызывает метод `Play()` объекта `SoundPlayer`.

Метод `Load(string file)` класса `SoundPlayerAdapter` уже выполняет необходимые преобразования, чтобы обеспечить правильную работу  `SoundPlayer`.
Сначала он устанавливает путь к файлу через свойство `SoundLocation` и потом уже вызывает метод `Load()`



```ad-note
title:Сложный пример
collapse:

Изначально мы хранили логи в SqlServer, для этого создали класс `SqlServerLogSaver` и класс `Client`, который каждый последний день месяца сохраняет логи.

Потом другой разработчик реализовал ещё один класс для хранения логов `ElasticsearchLogSaver`, но он стал обладать интерфейсомо отличным от интерфейса SqlServerLogSaver=> клиент уже не может без проблем использовать 
`ElasticsearchLogSaver`

Чтобы не переписывать `ElasticsearchLogSaver` или `Client`(его не всегда есьб возможность переписать) можно создать промежуточный адаптер, который будет преобразовать входные параметры таким образом, чтобы они стали пригодными для интерфейса с которым работает клиент.

Задача:
1.Преобразовать входные параметры
2.Вызвать метод адаптируемого класса

Как мы вижим интерфейс `ILogSaver` требует, чтобы метод `SaveLog` принимал класс `LogEntry`, а наш адаптируемый класс `ElasticsearchLogSaver` принимает `SimpleLogEntry`. Тут есть два решения.

1.Если класс `LogEntry` является незапечатанным и мы готовы чуть поменять код который оперирует класс `ElasticsearchLogSaver`, то лучше унаследоваться от `LogEntry` и определить конструктор с параметрами которые действительно нужны для `ElasticsearchLogSaver`. 
В нашем случае `ElasticsearchLogSaver` представляет более упрощённую версию чем `SqlServerLogSaver` и ему нужны только параметры `applicationId,string message`.Определяем свой конструктор с этими параметрами, а в базовый класс передаём null или дефольные параметры

2.Если класс `LogEntry` является запечатанным, то придётся при вызове `SaveLog` для ненужных параметров каждый раз передовать null или дефольные параметры и в адаптере выбирать только уже нужные.

В данном коде реализован первый вариант.


```cs
public class SqlServerLogSaver : ILogSaver
{
	public async void SaveLog(LogEntry logEntry)
	{
		await Task.FromResult<object>(null);
		Console.WriteLine($"Add {nameof(logEntry)} to log");
	}
}

interface ILogElasticsearchSaver
{
	Task SaveSimpleLog(SimpleLogEntry simpleLogEntry);
}

public sealed class ElasticsearchLogSaver : ILogElasticsearchSaver //Adaptee
{
	public Task SaveSimpleLog(SimpleLogEntry simpleLogEntry)
	{
		// Сохраняем переданную запись в Elasticsearch
		Console.WriteLine($"Add {nameof(simpleLogEntry)} to log");
		return Task.FromResult<object>(null);
	}
}

public class LogEntry
{
	internal string applicationId;
	internal string severity;
	internal DateTime dateTime;

	public LogEntry(string applicationId, string severity, DateTime dateTime, string message)
	{
		this.applicationId = applicationId;
		this.severity = severity;
		this.dateTime = dateTime;
	}
}

public class SimpleLogEntry:LogEntry
{
	internal string message;

	public SimpleLogEntry(string applicationId, string message)
	:base(applicationId,null,default,null)
	{
		this.message = message;
	}
}

interface ILogSaver//Target
{
	void SaveLog(LogEntry logEntry);
}

public class ElasticsearchLogSaverAdapter : ILogSaver //Adapter
{
	private readonly ElasticsearchLogSaver _elasticServerLogSaver;

	public ElasticsearchLogSaverAdapter(ElasticsearchLogSaver elasticServerLogSaver)
	{
		_elasticServerLogSaver = elasticServerLogSaver;
	}

	public async void SaveLog(LogEntry logEntry)
	{
		
		var simpleEntry = logEntry as SimpleLogEntry;
		if (simpleEntry != null)
		{
			await _elasticServerLogSaver.SaveSimpleLog(simpleEntry);
		}
	}
}


internal class Program
{
	static void Main(string[] args)
	{
		Client client = new Client();

		SqlServerLogSaver sqlServerLogSaver = new SqlServerLogSaver();
		
		ILogSaver logSqlSaver = sqlServerLogSaver;

		client.SaveLog(logSqlSaver, new LogEntry(Process.GetCurrentProcess().ProcessName, "Critical", DateTime.Now, "Connection timeout"));


		//Понадобилось поменять хранилище данных на Elasticsearch
		ElasticsearchLogSaver elasticsearchLogSaver = new ElasticsearchLogSaver();
		ILogSaver logElasticServer = new ElasticsearchLogSaverAdapter(elasticsearchLogSaver);


		client.SaveLog(logElasticServer, new SimpleLogEntry(Process.GetCurrentProcess().ProcessName, "Connection timeout"));
	}
}

class Client//Client
{
	public void SaveLog(ILogSaver target, LogEntry logEntry)
	{
		DateTime today = DateTime.Today;

		// Проверяем, является ли текущий день последним днем месяца
		if (today.Day == DateTime.DaysInMonth(today.Year, today.Month))
		{
			// Выполняем переданное действие
			target.SaveLog(logEntry);
		}
		
	}
}
```

https://andrey.moveax.ru/post/patterns-oop-structural-adapter