---
created: <% tp.date.now("YYYY-MM-DD HH:mm:ss (Z), dddd") %>
---
# Трекинг времени
```dataviewjs
// Извлекаем год и месяц из имени файла
const fileName = dv.current().file.name;
const [year, month] = fileName.split('-').map(Number);

// Функция для преобразования времени из строки в секунды
function timeToSeconds(time) {
    const [hours, minutes, seconds] = time.split(':').map(Number);
    return hours * 3600 + minutes * 60 + seconds;
}

// Функция для преобразования секунд в часы и минуты
function secondsToTime(seconds) {
    const hours = Math.floor(seconds / 3600);
    const minutes = Math.floor((seconds % 3600) / 60);
    return `${hours}ч ${minutes}м`;
}

// Функция для парсинга интервалов рабочего времени из содержимого файла
function parseWorkIntervals(content) {
    const lines = content.trim().split('\n');
    const workIntervals = [];
    const issues = [];

    let startTime = null;

    lines.forEach(line => {
        const match = line.match(/(\d{2}:\d{2}:\d{2})(!!)?/);
        if (match) {
            const time = match[1];
            const isWork = !!match[2];

            if (isWork) {
                if (startTime === null) {
                    startTime = time;
                }
            } else {
                if (startTime !== null) {
                    workIntervals.push({ start: startTime, end: time });
                    startTime = null;
                }
            }
        }
    });

    // Если последний интервал не закрыт, добавляем в список проблемных дней
    if (startTime !== null) {
        issues.push("Последний интервал не закрыт");
    }

    return { workIntervals, issues };
}

// Функция для вычисления общего рабочего времени в секундах
function calculateWorkTime(intervals) {
    let totalWorkTime = 0;

    intervals.forEach(interval => {
        const start = new Date(`1970-01-01T${interval.start}Z`);
        const end = new Date(`1970-01-01T${interval.end}Z`);
        const diff = (end - start) / 1000; // Разница в секундах

        totalWorkTime += diff;
    });

    return totalWorkTime;
}

// Функции для вычисления статистик
const sum = arr => arr.reduce((a, b) => a + b, 0);
const average = arr => sum(arr) / arr.length;
const median = arr => {
    const sorted = [...arr].sort((a, b) => a - b);
    const mid = Math.floor(sorted.length / 2);
    return sorted.length % 2 !== 0 ? sorted[mid] : (sorted[mid - 1] + sorted[mid]) / 2;
};

// Фильтруем все файлы в базе данных и ищем те, которые соответствуют шаблону YYYY-MM-DD за указанный месяц
const files = dv.pages()
    .where(f => f.file.name.match(/^\d{4}-\d{2}-\d{2}$/) && f.file.name.startsWith(`${year}-${String(month).padStart(2, '0')}`));

if (files.length === 0) {
    dv.span(`Не найдено файлов за указанный месяц.`);
} else {
    // Переменные для сбора сводной информации
    let totalWorkTime = 0;
    let dailyWorkTimes = [];
    let dailyWorkFiles = [];
    let issueFiles = [];

    let filePromises = files.map(async file => {
        const content = await dv.io.load(file.file.path);

        // Проверяем, есть ли содержимое файла
        if (!content) {
            return;
        }

        // Парсинг интервалов рабочего времени и проверка на проблемы
        const { workIntervals, issues } = parseWorkIntervals(content);
        if (issues.length > 0) {
            issueFiles.push({ file: file.file.name, issues });
        }

        const dailyTotal = calculateWorkTime(workIntervals);
        if (dailyTotal > 0) {
            totalWorkTime += dailyTotal;
            dailyWorkTimes.push(dailyTotal);
            dailyWorkFiles.push(file.file.name);
        }
    });

    Promise.all(filePromises).then(() => {
        if (dailyWorkTimes.length > 0) {
            // Вычисление статистик для сводной информации
            const minDailyWorkTime = Math.min(...dailyWorkTimes);
            const maxDailyWorkTime = Math.max(...dailyWorkTimes);
            const avgDailyWorkTime = average(dailyWorkTimes);
            const medDailyWorkTime = median(dailyWorkTimes);

            const minDay = dailyWorkFiles[dailyWorkTimes.indexOf(minDailyWorkTime)];
            const maxDay = dailyWorkFiles[dailyWorkTimes.indexOf(maxDailyWorkTime)];

            // Отображение сводной информации в виде таблицы
            dv.table(["Параметр", "Значение"], [
                ["Дней с отметками", dailyWorkTimes.length],
                ["Минимальное рабочее время за день", `${secondsToTime(minDailyWorkTime)} ([${minDay}](${minDay}))`],
                ["Максимальное рабочее время за день", `${secondsToTime(maxDailyWorkTime)} ([${maxDay}](${maxDay}))`],
                ["Среднее рабочее время за день", secondsToTime(avgDailyWorkTime)],
                ["Медианное рабочее время за день", secondsToTime(medDailyWorkTime)],
                ["Суммарное рабочее время за месяц", secondsToTime(totalWorkTime)]
            ]);
        } else {
            dv.span(`Не удалось найти данные о рабочем времени за указанный месяц.`);
        }

        if (issueFiles.length > 0) {
            dv.header(3, "Проблемные дни");
            issueFiles.forEach(issueFile => {
                dv.list([`[[${issueFile.file}]]: ${issueFile.issues.join(', ')}`]);
            });
        }
    });
}

```

# Рефлексия за месяц:
- <% tp.file.cursor(1) %>
(Сюда стоит добавить вопросы или темы, к которым хотите возвращаться регулярно. Например, [что-то такое](https://www.google.com/search?q=%D0%B2%D0%BE%D0%BF%D1%80%D0%BE%D1%81%D1%8B%2C+%D0%BA%D0%BE%D1%82%D0%BE%D1%80%D1%8B%D0%B5+%D1%81%D1%82%D0%BE%D0%B8%D1%82+%D0%B7%D0%B0%D0%B4%D0%B0%D1%82%D1%8C+%D1%81%D0%B5%D0%B1%D0%B5+%D0%B2+%D0%BA%D0%BE%D0%BD%D1%86%D0%B5+%D0%BC%D0%B5%D1%81%D1%8F%D1%86%D0%B0)