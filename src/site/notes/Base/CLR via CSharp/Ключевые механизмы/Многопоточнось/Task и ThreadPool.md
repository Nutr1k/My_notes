---
{"dg-publish":true,"permalink":"/base/clr-via-c-sharp/klyuchevye-mehanizmy/mnogopotochnos/task-i-thread-pool/"}
---

Данный методы используют поток из ThreadPool

```csharp
ThreadPool.QueueUserWorkItem(ComputeBoundOp, 5); // Вызов QueueUserWorkItem
new Task(ComputeBoundOp, 5).Start();// Аналог предыдущей строки
Task.Run(() => ComputeBoundOp(5));// Еще один аналог
```

Пример:
```csharp
private static void ComputeBoundOp(Object state)
{
	// Метод, выполняемый выделенным потоком
	Console.WriteLine($"Start:{Environment.CurrentManagedThreadId}");
	Console.WriteLine("In ComputeBoundOp: state={0}", state);
	Thread.Sleep(1000);// Имитация другой работы (1 секунда)
					   // После возвращения методом управления выделенный 
					   // поток завершается
	Console.WriteLine($"End:{Environment.CurrentManagedThreadId}");
}
static async Task Main(string[] args)
{
	//Способ 1
	Console.WriteLine($"Main start:{Environment.CurrentManagedThreadId}");
	var resetEvent = new ManualResetEvent(false);
	ThreadPool.QueueUserWorkItem(
		arg =>
		{
			ComputeBoundOp(5);
			resetEvent.Set();
		});
	Console.WriteLine("Продолжаем выполнение и ждём завершения");
	resetEvent.WaitOne();
	Console.WriteLine($"Main End:{Environment.CurrentManagedThreadId}\n");
	Console.ReadLine();

	//Способ 2
	Console.WriteLine($"Main start:{Environment.CurrentManagedThreadId}");
	Task tsk1=new Task(ComputeBoundOp, 5,TaskCreationOptions.DenyChildAttach);
	tsk1.Start();
	Console.WriteLine("Продолжаем выполнение и ждём завершения");
	tsk1.Wait();
	Console.WriteLine($"Main End:{Environment.CurrentManagedThreadId}\n");
	Console.ReadLine();


	//Способ 3
	Console.WriteLine($"Main start:{Environment.CurrentManagedThreadId}\n");
	var tsk2=Task.Run(() => ComputeBoundOp(5));
	Console.WriteLine("Продолжаем выполнение и ждём завершения");
	tsk2.Wait();
	Console.WriteLine($"Main End:{Environment.CurrentManagedThreadId}");
	Console.ReadLine();
	}
```

### Отмена выполнения

```csharp

CancellationTokenSource cts = new CancellationTokenSource(); 
Task t = new Task(() => Sum(cts.Token, 10000), cts.Token); 
t.Start(); // Позднее отменим CancellationTokenSource, чтобы отменить Task 
cts.Cancel(); // Это асинхронный запрос, задача уже может быть завершена

try 
{
	// В случае отмены задания метод Result генерирует
	// исключение AggregateException
	Console.WriteLine("The sum is: " + t.Result); // Значение Int32
}
catch (AggregateException x) 
{
	// Считаем обработанными все объекты OperationCanceledException
	// Все остальные исключения попадают в новый объект AggregateException,
	// состоящий только из необработанных исключений
	 x.Handle(e => e is OperationCanceledException);
	// Строка выполняется, если все исключения уже обработаны
	 Console.WriteLine("Sum was canceled");
}
```

### Выполнения задачи в вызывающем потоке
Поток это программная конструкция. Т.к. мы без проблем можешь реализовать свой [[Base/CLR via CSharp/Ключевые механизмы/Многопоточнось/Планировщики заданий(TaskScheduler)\|Планировщики заданий(TaskScheduler)]], который будет делать всё что нам захочеться.

Вот реализация планировщика заданий, который запускает Task в вызывающем потоке
```csharp
class InlineTaskScheduler : TaskScheduler
{
	protected override void QueueTask(Task task)
	{
		TryExecuteTask(task); // Немедленно выполняем задачу
	}

	protected override bool TryExecuteTaskInline(Task task, bool taskWasPreviouslyQueued)
	{
		return TryExecuteTask(task);
	}

	protected override IEnumerable<Task> GetScheduledTasks() => null;
}


Task tsk1=new Task(ComputeBoundOp, 5);


var scheduler = new InlineTaskScheduler();
tsk1.Start(scheduler);
Console.WriteLine("Продолжаем выполнение и ждём завершения");
tsk1.Wait();
```

Несмотря на то, что метод ожидает завершения выполнения задачи `tsk1.Wait()` находиться после метода вывода сообщения `Console.WriteLine("Продолжаем выполнение и ждём завершения");`, `метод вывода сообщения` не будет вызван т.к. задача `tsk1` выполняется в текущем потоке что и вызывающий метод и ядро процессора будет занято обработкой именно `текущей задачи` и никак не сможет выполнить `метода вывода сообщения`.