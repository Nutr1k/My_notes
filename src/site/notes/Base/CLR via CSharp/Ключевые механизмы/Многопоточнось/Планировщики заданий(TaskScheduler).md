---
dg-publish: true
---
TaskScheduler отвечает за выполнение запланированных заданий

В FCL существует два производных от `TaskScheduler` типа: 
- Планировщик заданий в пуле потоков
- Планировщик заданий контекста синхронизации. 


### Планировщик заданий в пуле потоков
По умолчанию все приложения используют первый из них, планирующий задания рабочих потоков в пуле. 
```csharp
var taskScheduler = TaskScheduler.Default;
```


### Планировщики заданий контекста синхронизации
Планировщики заданий контекста синхронизации обычно применяются в приложениях Windows Forms, Windows Presentation Foundation (WPF), Silverlight и Windows Store. Они планируют задания в потоке графического интерфейса приложения, обеспечивая оперативное обновление таких элементов интерфейса, как кнопки, пункты меню и т. п. **Этот планировщик вообще никак не использует пул потоков. **
```csharp
var taskScheduler = TaskScheduler.FromCurrentSynchronizationContext();
```


### Сравнение двух планировщиков

```csharp
private static void ComputeBoundOp(Object state)
{
	// Метод, выполняемый выделенным потоком
	Console.WriteLine($"Start:{Environment.CurrentManagedThreadId}");
	Console.WriteLine("In ComputeBoundOp: state={0}", state);
	Thread.Sleep(9000);// Имитация другой работы (1 секунда)
					   // После возвращения методом управления выделенный поток завершается
	Console.WriteLine($"End:{Environment.CurrentManagedThreadId}");
}

//Планировщик заданий в пуле потоков
Console.WriteLine($"Main start:{Environment.CurrentManagedThreadId}");
Task tsk1=new Task(ComputeBoundOp, 5);
var taskScheduler = TaskScheduler.Default;
tsk1.Start(taskScheduler);
Console.WriteLine("Продолжаем выполнение и ждём завершения");
tsk1.Wait();
Console.WriteLine($"Main End:{Environment.CurrentManagedThreadId}\n");
Console.ReadLine();

//Планировщики заданий контекста синхронизации
Console.WriteLine($"Main start:{Environment.CurrentManagedThreadId}");
Task tsk2 = new Task(ComputeBoundOp, 5);
SynchronizationContext.SetSynchronizationContext(new SynchronizationContext());
var taskSchedulerSync = TaskScheduler.FromCurrentSynchronizationContext();
tsk2.Start(taskSchedulerSync);
Console.WriteLine("Продолжаем выполнение и ждём завершения");
tsk2.Wait();
Console.WriteLine($"Main End:{Environment.CurrentManagedThreadId}\n");
Console.ReadLine();
```


![Pasted image 20241217194008.png](/img/user/Files/Image/Pasted%20image%2020241217194008.png)