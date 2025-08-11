---
dg-publish: true
---

**Управляемый модуль** (managed module) — стандартный переносимый исполняемый (portable executable, PE) файл 32-разрядной (PE32) или 64-разрядной Windows (PE32+), который требует для своего выполнения CLR. 
> [!info] Определение ECMA 335
>  Модуль: единичный файл, содержащий исполняемый код для системы виртуального выполнения(VES)
>  **CLR — это реализация VES (Virtual Execution System) от Microsoft**

Управляемые сборки всегда используют преимущества функции безопасности «предотвращения выполнения данных» (DEP, Data Execution Prevention) и технологию ASLR (Address Space Layout Optimization), применение этих технологий повышает информационную безопасность всей системы.

[Больше о netmodul](https://ru.stackoverflow.com/questions/1161449/%D0%A7%D1%82%D0%BE-%D1%82%D0%B0%D0%BA%D0%BE%D0%B5-netmodule-%D0%B8-%D0%BA%D0%B0%D0%BA-%D0%BE%D0%BD-%D1%81%D0%B2%D1%8F%D0%B7%D0%B0%D0%BD-%D1%81-%D1%83%D0%BF%D1%80%D0%B0%D0%B2%D0%BB%D1%8F%D0%B5%D0%BC%D1%8B%D0%BC-%D0%BC%D0%BE%D0%B4%D1%83%D0%BB%D0%B5%D0%BC)

Части управляемого модуля

| Часть                          | Описание                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Заголовок PE32 или PE32+       | Стандартный заголовок PE-файла Windows, аналогичный заголовку Common Object File Format (COFF). Файл с заголовком в формате PE32 может выполняться в 32- и 64-разрядной версиях Windows, а с заголовком PE32+ — только в 64-разрядной. Заголовок обозначает тип файла: GUI, CUI или DLL, он также имеет временную метку, показывающую, когда файл был собран. Для модулей, содержащих только IL-код, основной объем информации в заголовке PE32(+) игнорируется. В модулях, содержащих машинный код, этот заголовок содержит сведения о машинном коде |
| Заголовок CLR                  | Содержит информацию (интерпретируемую CLR и утилитами), которая превращает этот модуль в управляемый. Заголовок включает нужную версию CLR, некоторые флаги, метку метаданных MethodDef точки входа в управляемый модуль (метод Main), а также месторасположение/размер метаданных модуля, ресурсов, строгого имени, некоторых флагов и пр.                                                                                                                                                                                                           |
| Метаданные                     | Каждый управляемый модуль содержит таблицы метаданных. Есть два основных вида таблиц — это таблицы, описывающие типы данных и их члены, определенные в исходном коде, и таблицы, описывающие типы данных и их члены, на которые имеются ссылки в исходном коде                                                                                                                                                                                                                                                                                        |
| Код Intermediate Language (IL) | Код, создаваемый компилятором при компиляции исходного кода. Впоследствии CLR компилирует IL в машинные команды                                                                                                                                                                                                                                                                                                                                                                                                                                       |


### Создание управляемого модуля

Путь к csc: %SystemRoot%\Microsoft.NET\Framework(64)\
```
csc "Имя файла".c  /r:"Ссылка на сборку если она нужна\имя сборки.dll"
csc "Имя файла".cs /t:module /lib:"Ссылка на сборку если она нужна" /r:"имя сборки по ссылке"

```
```
csc "Имя файла".cs /t:module
```


Манифест модуля c сылкой на библиотеку классов(сборку):
![Pasted image 20230215163019.png](/img/user/Files/Image/Pasted%20image%2020230215163019.png)

Манифест модуля:
![Pasted image 20230215171230.png](/img/user/Files/Image/Pasted%20image%2020230215171230.png)

Как видно в нём отсутствует информация о сборке т.к. это модуль, а не сборка.

```
csc /t:module RUT.cs
```

![Pasted image 20230215170833.png](/img/user/Files/Image/Pasted%20image%2020230215170833.png)

### Указание ссылки на модуль библиотеки классов

```
csc Program.cs /t:exe /lib:"G:\source\repos\CalcFraemwork\Lib" /addmodule:LibClass.netmodule
```
LibClass.netmodule - модуль библиотеки классов
/lib: - путь до папки где содержится .netmodule

Теперь сборка Program ссылается на модуль LibClass.netmodule
![Pasted image 20230215170236.png](/img/user/Files/Image/Pasted%20image%2020230215170236.png)


```
csc /out:MultiFileLibrary.dll /t:library /addmodule:RUT.netmodule FUT.cs
```
![Pasted image 20230215171434.png](/img/user/Files/Image/Pasted%20image%2020230215171434.png)

### Многофайловая сборка из управляемых модулей
Так же можно создавать библиотеки(не сборки) которые в себе не содержат ничего кроме таблицы метаданных манифеста, указывающие, что файлы RUT.netmodule и FUT. netmodule входят в состав сборки.
```
csc /t:module RUT.cs 
csc /t:module FUT.cs 
al /out: MultiFileLibrary.dll /t:library FUT.netmodule RUT.netmodule
```
dll библиотека:
![Pasted image 20230215180609.png](/img/user/Files/Image/Pasted%20image%2020230215180609.png)

Так же можно создать исполняемый файл:
```
al /out:Promgram.exe /t:exe /main:Prog1.Program.Main Prog1.netmodule Prog2.netmodule
```
Он будет содержать в себе точку входа:
![Pasted image 20230215182315.png](/img/user/Files/Image/Pasted%20image%2020230215182315.png)

Таблица метаданных манифеста:
![Pasted image 20230215175432.png](/img/user/Files/Image/Pasted%20image%2020230215175432.png)


Для экспериментов:

```
csc Prog2.cs /t:module

csc Prog1.cs /t:module /addmodule:Prog2.netmodule

Для exe
al /out:Promgram.exe /t:exe /main:Prog1.Program.Main Prog1.netmodule Prog2.netmodule

Для dll
al /out:Promgram.dll /t:library Prog1.netmodule Prog2.netmodule
```
/main - точка входа в программу

Результирующая сборка состоит из трех файлов: Promgram.dll, Prog1.netmodule и Prog2.netmodule

Многофайловая сборка из трех управляемых модулей и манифеста
![Pasted image 20230215181722.png](/img/user/Files/Image/Pasted%20image%2020230215181722.png)


Prog1.cs
```csharp
using System;
using Prog2;

namespace Prog1
{
	class Program
	{
		public static void Main()
		{
			string name="Prog1";
			Prog2.Program.Show(name);
			Console.ReadLine();
		}
	}
}
```

Prog2.cs
```csharp
using System;

namespace Prog2
{
	class Program
	{
		public static void Show(string text)
		{
			Console.WriteLine(text);
		}
		
		public static void Main()
		{
			string name="Prog2";
			Console.WriteLine(name);
			Console.ReadLine();
		}
	}
}
```

