---
dg-publish: true
---
1. Суть проекта в разработки бэкэнда с вертикальным делением
2. Т.к. я не особо могу во фронтенд, то сначала будет минимальный интерфейс:
	- Страница авторизации
	- Страница регистрации по коду приглашения
	- Страница создания динамического кода
	- Страница редактирования динамического когда
		- Загрузка изображения(с поддержкой автоматической конвертации pdf to jpeg)
3. Автоадаптация под телефоны и пк
4. Сайт должен быть многопользовательский, у каждого юзера свои QR.

https://www.milanjovanovic.tech/blog/vertical-slice-architecture
https://code-maze.com/vertical-slice-architecture-aspnet-core/
https://bool.dev/blog/detail/obzor-vertical-slice-architecture




https://leonidchernenko.ru/vertical-slice/

https://github.com/jonowilliams26/StructuredMinimalApi

https://medium.com/@v.cheshmy/introduction-ae32b9f32ac5
https://www.youtube.com/watch?v=ZA2X1gaAhJk

https://www.youtube.com/watch?v=ZA2X1gaAhJk

https://github.com/json-editor/json-editor

111111111111111111111111111111111

Подключение к существующей базе данных

В Package Manager Console:
`Scaffold-DbContext "Server=localhost ;Database=DynamicQR;Trusted_Connection=True;TrustServerCertificate=True;Encrypt=False" Microsoft.EntityFrameworkCore.SqlServer -v`


Scaffold-DbContext "Server=localhost;Database=DynamicQR;Trusted_Connection=True;TrustServerCertificate=True;Encrypt=False" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Data/Types -ContextDir Data -Force -v


- `-OutputDir Models` — папка, в которую будут сгенерированы классы моделей (можно указать любую).
- `-Force` — перезапишет существующие классы.


