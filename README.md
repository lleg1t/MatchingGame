# Игра «Найди пару» (Matching Game) на Windows Forms

Это моя практическая работа по C#. Я сделал игру, в которой нужно находить одинаковые значки. Игру делал по руководству от Microsoft в Visual Studio, ссылка на него внизу.

Правила простые: на поле 16 квадратов, внутри спрятаны 8 пар значков. Нажимаешь на два квадрата. Если значки одинаковые, они остаются открытыми. Если разные, то через секунду закрываются обратно. Когда открыл всё поле, выскакивает сообщение, что ты выиграл.

<!-- СКРИН 1: готовая игра в процессе, часть пар уже открыта. Сохранить как screenshots/00-game.png -->
![Игра в процессе](screenshots/00-game.png)

## Что нужно, чтобы запустить

- Visual Studio (я использовал бесплатную Community)
- Рабочая нагрузка **Разработка классических приложений .NET** (ставится через Visual Studio Installer)
- Windows, потому что Windows Forms работает только там

## Как запустить

1. Скачать репозиторий (кнопка **Code** -> **Download ZIP**) и распаковать.
2. Открыть файл `MatchingGame.sln` в Visual Studio.
3. Нажать **F5** (или зелёную кнопку «Запуск»).

## Как я делал

### 1. Создал проект

Открыл Visual Studio, нажал **Создать проект**, в поиске написал *Windows Forms* и выбрал шаблон **Приложение Windows Forms (.NET Framework)** для C#.

<!-- СКРИН 2: окно «Создание проекта» с выбранным шаблоном Windows Forms (.NET Framework). Файл screenshots/01-create-project.png -->
![Создание проекта](screenshots/01-create-project.png)

Назвал проект `MatchingGame` и нажал **Создать**.

<!-- СКРИН 3: окно «Настроить новый проект» с названием MatchingGame. Файл screenshots/02-configure-project.png -->
![Настройка проекта](screenshots/02-configure-project.png)

После этого открылась пустая форма в конструкторе.

### 2. Настроил форму

Нажал на форму и в окне **Свойства** поменял:

- `Text` с `Form1` на `Matching Game`, это заголовок окна
- `Size` на `550, 550`

<!-- СКРИН 4: форма в конструкторе и справа окно «Свойства» с изменёнными Text и Size. Файл screenshots/03-form-properties.png -->
![Свойства формы](screenshots/03-form-properties.png)

### 3. Сделал сетку 4 на 4

Из панели элементов (Ctrl+Alt+X) перетащил на форму `TableLayoutPanel` из раздела «Контейнеры». В свойствах поставил:

- `BackColor` = `CornflowerBlue` (он в самом низу списка, цвета не по алфавиту)
- `Dock` = `Fill`, чтобы таблица заняла всю форму
- `CellBorderStyle` = `Inset`, чтобы между клетками были границы

<!-- СКРИН 5: окно «Свойства» для tableLayoutPanel1 с этими тремя свойствами. Файл screenshots/04-tablelayoutpanel-properties.png -->
![Свойства TableLayoutPanel](screenshots/04-tablelayoutpanel-properties.png)

По умолчанию в таблице 2 строки и 2 столбца. Через маленький треугольник в правом верхнем углу таблицы открыл меню задач и два раза нажал **Добавить строку** и два раза **Добавить столбец**. Получилось 4 на 4.

Потом зашёл в **Изменить строки и столбцы** и поставил каждому столбцу ширину 25 %, а каждой строке высоту 25 %. Тогда все клетки одинаковые.

<!-- СКРИН 6: окно «Стили столбцов и строк» (со значениями 25 процентов). Файл screenshots/05-styles-rows-columns.png -->
![Стили строк и столбцов](screenshots/05-styles-rows-columns.png)

<!-- СКРИН 7: форма с пустой сеткой 4 на 4. Файл screenshots/06-grid-4x4.png -->
![Пустая сетка](screenshots/06-grid-4x4.png)

### 4. Добавил метки (Label)

В каждой клетке будет лежать значок, а значок это просто буква в специальном шрифте. Поэтому в клетки я кладу `Label`.

Взял `Label` из раздела «Стандартные элементы управления», положил в левую верхнюю клетку и поменял свойства:

- `BackColor` = `CornflowerBlue`
- `AutoSize` = `False`
- `Dock` = `Fill`
- `TextAlign` = `MiddleCenter`
- `Font` = `Webdings`, полужирный, размер 48
- `Text` = `c`

Webdings это шрифт, где вместо букв нарисованы значки. Буква `c` в нём выглядит как чёрный квадрат. Если значки не отображаются нормально, нужно поставить `UseCompatibleTextRendering` = `True`.

<!-- СКРИН 8: выбранная метка и справа её свойства (BackColor, Font, Text и т.д.). Файл screenshots/07-label-properties.png -->
![Свойства метки](screenshots/07-label-properties.png)

Потом скопировал эту метку (Ctrl+C) и вставил (Ctrl+V) во все остальные клетки. Всего получилось 16 меток.

<!-- СКРИН 9: форма с 16 чёрными квадратами. Файл screenshots/08-grid-filled.png -->
![Заполненная сетка](screenshots/08-grid-filled.png)

### 5. Случайные значки

Сначала в начале класса `Form1` объявил несколько переменных:

```csharp
Random random = new Random();

List<string> icons = new List<string>()
{
    "!", "!", "N", "N", ",", ",", "k", "k",
    "b", "b", "v", "v", "w", "w", "z", "z"
};

Label firstClicked = null;
Label secondClicked = null;
```

`icons` это список букв, каждая встречается два раза, потому что нужны пары. `firstClicked` и `secondClicked` хранят, на какие метки игрок уже нажал.

Потом написал метод `AssignIconsToSquares()`. Он проходит по всем меткам через `foreach`, берёт случайную букву из списка, записывает её в метку и удаляет из списка, чтобы она не попалась больше двух раз. Ещё я делаю цвет шрифта таким же, как цвет фона, и значок становится невидимым.

```csharp
private void AssignIconsToSquares()
{
    foreach (Control control in tableLayoutPanel1.Controls)
    {
        Label iconLabel = control as Label;
        if (iconLabel != null)
        {
            int randomNumber = random.Next(icons.Count);
            iconLabel.Text = icons[randomNumber];
            iconLabel.ForeColor = iconLabel.BackColor;
            icons.RemoveAt(randomNumber);
        }
    }
}
```

Этот метод вызывается в конструкторе формы после `InitializeComponent();`.

### 6. Обработчик нажатия

Нужно, чтобы при нажатии на любую метку происходило одно и то же. Поэтому я сделал один общий обработчик `label_Click` и привязал его ко всем 16 меткам: выделил метку, в окне «Свойства» нажал на значок молнии (события) и в событии `Click` выбрал `label_Click`.

<!-- СКРИН 10: окно «Свойства» на вкладке событий (молния), где у Click выбран label_Click. Файл screenshots/09-events.png -->
![События метки](screenshots/09-events.png)

В обработчике такая логика:

1. Если таймер сейчас работает, ничего не делаем (иначе можно открыть три значка).
2. Если значок уже открыт (чёрный), ничего не делаем.
3. Если это первый клик, запоминаем метку в `firstClicked` и делаем значок чёрным.
4. Если второй, запоминаем в `secondClicked`, делаем чёрным и проверяем, не выиграли ли мы.
5. Если значки совпали, обнуляем обе переменные и оставляем открытыми.
6. Если не совпали, запускаем таймер.

### 7. Таймер

Из панели элементов добавил `Timer`. Он появляется под формой, а не на ней. Интервал поставил `750` (миллисекунд).

<!-- СКРИН 11: таймер под формой и его свойства (Interval = 750). Файл screenshots/10-timer-properties.png -->
![Свойства таймера](screenshots/10-timer-properties.png)

Когда таймер срабатывает, он сам останавливается, прячет оба значка (цвет шрифта снова как у фона) и обнуляет `firstClicked` и `secondClicked`:

```csharp
private void timer1_Tick(object sender, EventArgs e)
{
    timer1.Stop();

    firstClicked.ForeColor = firstClicked.BackColor;
    secondClicked.ForeColor = secondClicked.BackColor;

    firstClicked = null;
    secondClicked = null;
}
```

### 8. Проверка победы

Метод `CheckForWinner()` проходит по всем меткам. Если хотя бы у одной цвет шрифта совпадает с фоном, значит она ещё скрыта, и метод просто выходит. Если таких нет, показывается сообщение и игра закрывается.

```csharp
MessageBox.Show("You matched all the icons!", "Congratulations");
Close();
```

## Как это выглядит в игре

<!-- СКРИН 12: открыт один значок (первый клик). Файл screenshots/11-first-click.png -->
![Первый клик](screenshots/11-first-click.png)

<!-- СКРИН 13: найдена пара, два одинаковых значка остались открытыми. Файл screenshots/12-pair-found.png -->
![Пара найдена](screenshots/12-pair-found.png)

<!-- СКРИН 14: окно «You matched all the icons!» после победы. Файл screenshots/13-win-message.png -->
![Победа](screenshots/13-win-message.png)

## Что в репозитории

- `Form1.cs` — вся логика игры
- `Form1.Designer.cs` — код формы, который Visual Studio создаёт сам при работе в конструкторе
- `Program.cs` — точка входа в программу
- `MatchingGame.sln`, `MatchingGame.csproj` — файлы решения и проекта
- `screenshots/` — скриншоты для этого руководства

## Чему я научился

- Создавать проект Windows Forms и работать в конструкторе форм
- Использовать `TableLayoutPanel` и `Label`
- Хранить данные в `List<T>` и перебирать их через `foreach`
- Делать один обработчик событий сразу для нескольких элементов
- Использовать `Timer`, который срабатывает один раз

## Источник

Делал по руководству Microsoft: [Руководство: Создание приложения Windows Forms для игры на сопоставление](https://learn.microsoft.com/ru-ru/visualstudio/get-started/csharp/tutorial-windows-forms-create-match-game?view=visualstudio)
