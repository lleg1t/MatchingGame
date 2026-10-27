# Игра «Найди пару» (Matching Game) на Windows Forms

Это моя практическая работа по C#. Я сделал игру, в которой нужно находить одинаковые значки. Делал по руководству от Microsoft в Visual Studio, ссылка на него внизу.

Правила простые: на поле 16 квадратов, внутри спрятаны 8 пар значков. Нажимаешь на два квадрата. Если значки одинаковые, они остаются открытыми. Если разные, то через секунду закрываются обратно. Когда открыл всё поле, выскакивает сообщение, что ты выиграл.

<!-- СКРИН 1: игра в процессе, часть пар открыта. Файл screenshots/00-game.png -->
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

### 1. Создал проект и сетку

Создал проект **Приложение Windows Forms (.NET Framework)** на C#, назвал его `MatchingGame`. В свойствах формы поставил `Text` = `Matching Game` и `Size` = `550, 550`.

Потом из панели элементов перетащил на форму `TableLayoutPanel` и поменял свойства:

- `BackColor` = `CornflowerBlue`
- `Dock` = `Fill`, чтобы таблица заняла всю форму
- `CellBorderStyle` = `Inset`, чтобы между клетками были границы

По умолчанию в таблице 2 строки и 2 столбца. Через меню задач (маленький треугольник в углу таблицы) добавил ещё по две строки и два столбца, а в окне **Изменить строки и столбцы** поставил всем по 25 %. Получилась сетка 4 на 4, где все клетки одинаковые.

### 2. Добавил метки (Label)

В каждой клетке лежит значок, а значок это просто буква в специальном шрифте. Поэтому в клетки я кладу `Label`. Для первой метки поменял свойства:

- `BackColor` = `CornflowerBlue`
- `AutoSize` = `False`
- `Dock` = `Fill`
- `TextAlign` = `MiddleCenter`
- `Font` = `Webdings`, полужирный, размер 48
- `Text` = `c`

Webdings это шрифт, где вместо букв нарисованы значки. Буква `c` в нём выглядит как чёрный квадрат. Если значки не отображаются нормально, нужно поставить `UseCompatibleTextRendering` = `True`.

<!-- СКРИН 2: выбранная метка и справа её свойства. Файл screenshots/01-label-properties.png -->
![Свойства метки](screenshots/01-label-properties.png)

Потом скопировал эту метку (Ctrl+C) и вставил (Ctrl+V) во все остальные клетки. Всего получилось 16 меток.

<!-- СКРИН 3: форма с 16 чёрными квадратами. Файл screenshots/02-grid-filled.png -->
![Сетка из 16 меток](screenshots/02-grid-filled.png)

### 3. Случайные значки

В начале класса `Form1` объявил несколько переменных:

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

### 4. Обработчик нажатия

Нужно, чтобы при нажатии на любую метку происходило одно и то же. Поэтому я сделал один общий обработчик `label_Click` и привязал его ко всем 16 меткам через событие `Click` в окне «Свойства».

В обработчике такая логика:

1. Если таймер сейчас работает, ничего не делаем (иначе можно открыть три значка).
2. Если значок уже открыт (чёрный), ничего не делаем.
3. Если это первый клик, запоминаем метку в `firstClicked` и делаем значок чёрным.
4. Если второй, запоминаем в `secondClicked`, делаем чёрным и проверяем, не выиграли ли мы.
5. Если значки совпали, обнуляем обе переменные и оставляем открытыми.
6. Если не совпали, запускаем таймер.

### 5. Таймер

Из панели элементов добавил `Timer`. Он появляется под формой, а не на ней. Интервал поставил `750` (миллисекунд).

<!-- СКРИН 4: таймер под формой и его свойства (Interval = 750). Файл screenshots/03-timer-properties.png -->
![Свойства таймера](screenshots/03-timer-properties.png)

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

### 6. Проверка победы

Метод `CheckForWinner()` проходит по всем меткам. Если хотя бы у одной цвет шрифта совпадает с фоном, значит она ещё скрыта, и метод просто выходит. Если таких нет, показывается сообщение и игра закрывается.

```csharp
MessageBox.Show("You matched all the icons!", "Congratulations");
Close();
```

<!-- СКРИН 5: окно «You matched all the icons!» после победы. Файл screenshots/04-win-message.png -->
![Победа](screenshots/04-win-message.png)

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
