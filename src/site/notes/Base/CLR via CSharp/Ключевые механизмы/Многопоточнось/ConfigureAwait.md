---
dg-publish: true
---
ConfigureAwait - нужен для того, чтобы указать, является ли наш асинхронный метод зависимым от того в каком контексте он будет выполняться. 

`ConfigureAwait(continueOnCapturedContext: false)` используется для предотвращения принудительного вызова колбэка в исходном контексте или планировщике.
Если код после `await` не требует выполнения в исходном контексте, то используем `ConfigureAwait(false)`

Что делает `.ConfigureAwait(false)`?
- Убирает необходимость возвращаться в исходный контекст синхронизации (например, UI-поток).
- Код после `await` может быть выполнен в **любом свободном потоке**, выбранном планировщиком задач (обычно из пула потоков).

`ConfigureAwait(true)`: запускает оставшуюся часть кода в том же контексте синхронизации, в котором был запущен код до await. Не обязательно тот же поток, но тот же контекст синхронизации. Контекст синхронизации может решить, как запустить код. В приложении UI это будет тот же поток. В ASP.NET это может быть не тот же поток, но он будет доступен `HttpContext`, как и раньше.

`ConfigureAwait(false)`сообщает, что не нужен контекст, поэтому код может быть запущен _где угодно_ . Это может быть любой поток, который его запускает.

Без `ConfigureAwait(false)`
```csharp
async Task GetTextAsync()
{
	MessageBox.Show($"Start:{Environment.CurrentManagedThreadId}");
	await Task.Delay(1000);
	MessageBox.Show($"Stop:{Environment.CurrentManagedThreadId}");
}

private void Button_Click_3(object sender, RoutedEventArgs e)
{
	var task=GetTextAsync(); 
	task.Wait();// синхронное ожидание
}
```

- UI-поток заблокирован вызовом `task.Wait()`.
- `GetTextAsync` ждёт завершения `Task.Delay(1000)`, но его продолжение(всё что после строки `await Task.Delay(1000)`) должно выполниться в UI-потоке.
- Продолжение задачи `GetTextAsync` никогда не выполнится, потому что UI-поток недоступен. **Deadlock**.

С `ConfigureAwait(false)`:
```csharp
async Task GetTextAsync()
{
	MessageBox.Show($"Start:{Environment.CurrentManagedThreadId}");
	await Task.Delay(1000).ConfigureAwait(false);
	MessageBox.Show($"Stop:{Environment.CurrentManagedThreadId}");
}

private void Button_Click_3(object sender, RoutedEventArgs e)
{
	var task=GetTextAsync(); 
	task.Wait();// синхронное ожидание
}
```

- UI-поток заблокирован вызовом `task.Wait()`.
- `GetTextAsync` ждёт завершения `Task.Delay(1000)`, но его продолжение(всё что после строки `await Task.Delay(1000)`) не обязательно должно выполниться в UI-потоке, а может выполниться в любом доступном потоке из пула потоков.
- Метод завершает выполнение, результат возвращается в вызов
- Заблокированный вызовом `task.Wait()` UI-поток разблокируется, получив результат задачи.

### Когда код может выполняться в UI-потоке даже с `.ConfigureAwait(false)`?

1. **Если `await` завершился в том же потоке (UI), где началось выполнение:**
    
    - Если задача завершилась синхронно (уже готова до `await`), то продолжение будет выполнено в том же потоке, даже с `.ConfigureAwait(false)`.

```csharp
async Task Example()
{
    await Task.CompletedTask.ConfigureAwait(false); // Задача уже завершена
    // Код здесь выполнится в текущем (UI) потоке
}
```

https://hamidmosalla.com/2018/06/24/what-is-synchronizationcontext/
https://habr.com/ru/articles/482354/