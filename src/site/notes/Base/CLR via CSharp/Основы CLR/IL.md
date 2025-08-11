---
dg-publish: true
---

IL (Intermediate Language, Промежуточный язык) - высокоуровневый ассемблер. Код, создаваемый CLR совместимым компилятором при компиляции исходного кода. Впоследствии CLR компилирует IL в машинные команды посредством [[Base/CLR via CSharp/Основы CLR/JIT-компилятор\|JIT-компилятора]] среды CLR. IL код это не зависящий от процессора машинный язык. Он считается более высокоуровневым по сравнению с большинством других **машинных языков**. IL является стековым языком.

IL, CIL, MSIL - синонимы, но с некоторым отличием.

1. CIL — (Спецификация). Общий промежуточный язык. (Международный термин)
Термин используемый компанией ECMA, которая занимается стандартизации информационных и коммуникационных технологий.
![Pasted image 20241015200039.png](/img/user/Files/Image/Pasted%20image%2020241015200039.png)
3. IL - промежуточный язык, наиболее распространенное и простое название, часто используемое в повседневной практике.
4. MSIL — это термин, обозначающий [реализацию этого стандарта](https://msdn.microsoft.com/netframework/) корпорацией Microsoft .
[Ссылка 1](https://flerka.github.io/personal-blog/2022-06-15-cil-msil-il/#:~:text=First%2C%20let%20us%20decrypt%20acronyms,MSIL%20is%20Microsoft%20Intermediate%20Language)
[Ссылка 2](https://learn.microsoft.com/en-us/archive/blogs/brada/whats-the-differnece-between-cil-and-msil)

Просмотреть IL код можно разными способами:
1. Через ildasm.exe->Файл->Дамп->ОК
2. Через сайт с удобной навигацией и объяснениями [[Base/CLR via CSharp/Проектирование типов/Просмотр IL кода(stack,heap)\|Просмотр IL кода(stack,heap)]]
3. [MicroScope](https://marketplace.visualstudio.com/items?itemName=bert.microscope)
4. ReSharper
Выберите ReSharper | Окна | IL-просмотрщик

Скомпилировать можно через ilasm.exe. Он расположен по пути: "C:\Windows\Microsoft.NET\Framework64\v4.0.30319"

```
ilasm.exe имя_файла.il /exe
```


Приложение на .NET Fraemwork удалось запустить из чисто скомпилированного il файла.
Приложение на .NET Core удалось запустить только через 

```
dotnet имя_файла
```
Жалуется на отсутствие System.Runtime и прочего
Наверно, нужно установить .NET Core не только как SDK, а как .NET Desktop Runtime или .NET Runtime

Debug IL Код:
[[Pasted image 20230131171657.png]]


Release IL Код:
[[Pasted image 20230131172513.png]]


### Словесное описание запуска и исполнения программы в среде CLR

```csharp
public sealed class Program 
{ 
	public static void Main() 
	{ 
	System.Console.WriteLine("Hi"); 
	} 
}
```

1. Допустим, в результате компиляции и построения этого кода получилась сборка Program.exe.
2. При запуске приложения происходит загрузка и инициализация CLR.
3. Затем CLR сканирует CLR-заголовок сборки в поисках атрибута <span class="P">MethodDefToken</span>, идентифицирующего метод Main, представляющий точку входа в приложение
![Pasted image 20241015204937.png](/img/user/Files/Image/Pasted%20image%2020241015204937.png)
5. CLR находит в таблице метаданных MethodDef смещение, по которому в файле находится IL-код этого метода, и компилирует его в машинный код процессора при помощи JIT-компилятора
6. Этот процесс включает в себя проверку безопасности типов в компилируемом коде, после чего начинается исполнение полученного машинного кода

IL код метода Main (Вид -> Показать байты):
![Pasted image 20230218203214.png](/img/user/Files/Image/Pasted%20image%2020230218203214.png)

Во время JIT-компиляции этого кода CLR обнаруживает все ссылки на типы и члены и загружает сборки, в которых они определены (если они еще не загружены)

Показанный код содержит ссылку на метод System.Console.WriteLine: команда Call ссылается на маркер метаданных **0A000007**. Этот маркер идентифицирует запись **7** таблицы метаданных <span class="B">MemberRef</span> (таблица **0A**). Просматривая эту запись, CLR видит, что одно из ее полей ссылается на элемент таблицы TypeRef (описывающий тип System.Console). Запись таблицы TypeRef направляет CLR к следующей записи в таблице AssemblyRef: MSCorLib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089.
На этом этапе CLR уже знает, какая сборка нужна, и ей остается лишь найти и загрузить эту сборку
MemberRef:
![Pasted image 20230218204143.png](/img/user/Files/Image/Pasted%20image%2020230218204143.png)

IL код методов типа System.Console содержится в библиотеке MSCorLib
![Pasted image 20230218205456.png](/img/user/Files/Image/Pasted%20image%2020230218205456.png)

### Изменение IL кода
[[Base/CLR via CSharp/Основы CLR/Изменение IL кода\|Изменение IL кода]]
