---
{"dg-publish":true,"permalink":"/base/c-sharp/i-enumerable-i-i-queryable/"}
---

![1_wNY_M1CggVdEaijsNaCptQ.webp](/img/user/Files/Image/1_wNY_M1CggVdEaijsNaCptQ.webp)
Интерфейс `IEnumerable` находится в пространстве имен `System.Collections`.
Объект `IEnumerable` представляет набор данных в памяти и может перемещаться по этим данным **только вперед**

Т.к. у него есть методы
**`MoveNext()`** — перемещает курсор на следующий элемент коллекции.
- **`Current`** — возвращает текущий элемент, на который указывает курсор.
- **`Reset()`** _(не всегда реализован)_ — сбрасывает курсор на начало коллекции (не поддерживается во многих реализациях).)

Интерфейс `IQueryable` располагается в пространстве имен `System.Linq`.
Объект `IQueryable` предоставляет удаленный доступ к базе данных и позволяет перемещаться по данным как в **прямом порядке** от начала до конца, так и в **обратном порядке**

Прямой и обратный порядок тут подразумевается в контексте базы данных:
- **Прямой порядок**: Когда вы запрашиваете данные, они возвращаются в их естественном порядке (например, по возрастанию идентификаторов).
- **Обратный порядок**: Вы можете запросить данные в обратном порядке, явно указав это с помощью метода `OrderByDescending`.


Основные отличия:
При использовании`IEnumerable` сначала будут загружены все записи из таблицы, а затем фильтрация(where), сортировка производиться средствами `.NET.`

При использовании `IQueryable` фильтрация(where), сортировка будет производиться на уровне базы данных, а загружены будут только отфильтрованные, отсортированные данные.
###### IEnumerable
Если говорить касательно особенности`IEnumerable`, то эта особенность проявляется когда мы сначала получаем данные, а потом выполняем фильтрацию.

```csharp
ShipsContext db = new ShipsContext();

IEnumerable<Ship> shipsIEnum = db.Ships;
var ships = shipsIEnum
	.Where(p => p.Class == "Kongo")
	.OrderBy(p=>p.Name).ToList();
```

Запрос в SQL Server Profiler приходит такой:
![Pasted image 20241211195624.png](/img/user/Files/Image/Pasted%20image%2020241211195624.png)

Если мы сразу применим метод `Where` и `OrderBy`, то фильтрация и сортировка будут выполнены на стороне сервера.

```csharp
ShipsContext db = new ShipsContext();

IEnumerable<Ship> ships = db.Ships
	.Where(p => p.Class == "Kongo")
	.OrderBy(p => p.Name).ToList();
```
Запрос в SQL Server Profiler приходит такой:
![Pasted image 20241211200402.png](/img/user/Files/Image/Pasted%20image%2020241211200402.png)
Это связанно с тем, что метод `Where` уже возвращает объект типа `IQueryable`.


###### IQueryable
```csharp
ShipsContext db = new ShipsContext();

IQueryable<Ship> shipsIQuer = db.Ships;
var ships= shipsIQuer
	.Where(p => p.Class == "Kongo")
	.OrderBy(p => p.Name).ToList();
```
Запрос в SQL Server Profiler приходит такой:
![Pasted image 20241211200940.png](/img/user/Files/Image/Pasted%20image%2020241211200940.png)

Основное различие заключается в том, где выполняется запрос. 
- В `IEnumerable` запрос обрабатывается в памяти, итерация происходит в коде.
- В `IQueryable` запрос выполняется на сервере, и только результат (например, строки SQL-запроса) возвращается в приложение, что снижает нагрузку на сеть.