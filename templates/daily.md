---
created: <% tp.date.now("YYYY-MM-DD HH:mm:ss (Z), dddd") %>
---

- <% tp.file.cursor(1) %>

# Рабочее время:
```dataviewjs
// Загрузка содержимого текущего файла
const currentFilePath = dv.current().file.path;
dv.io.load(currentFilePath).then(content => {
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

    // Функция для парсинга рабочих интервалов
    function parseWorkIntervals(content) {
        const lines = content.trim().split('\n');
        const workIntervals = [];

        let startTime = null;
        let endTime = null;

        lines.forEach(line => {
            const match = line.match(/(\d{2}:\d{2}:\d{2})(!!)?/);
            if (match) {
                const time = match[1];
                const isWork = !!match[2];

                if (isWork) {
                    if (startTime === null) {
                        startTime = time;
                    }
                    endTime = time;
                } else {
                    if (startTime !== null && endTime !== null) {
                        workIntervals.push({ start: startTime, end: endTime });
                        startTime = null;
                        endTime = null;
                    }
                }
            }
        });

        // If the last interval is not closed, close it with the last endTime
        if (startTime !== null && endTime !== null) {
            workIntervals.push({ start: startTime, end: endTime });
        }

        return workIntervals;
    }

    // Функция для вычисления времени работы
    function calculateWorkTime(intervals) {
        let totalWorkTime = 0;
        let workDurations = [];

        intervals.forEach(interval => {
            const start = new Date(`1970-01-01T${interval.start}Z`);
            const end = new Date(`1970-01-01T${interval.end}Z`);
            const diff = (end - start) / 1000; // Difference in seconds

            totalWorkTime += diff;
            workDurations.push(diff);
        });

        return { totalWorkTime, workDurations };
    }

    // Парсинг меток времени и вычисление интервалов рабочего времени
    const workIntervals = parseWorkIntervals(content);
    const { totalWorkTime, workDurations } = calculateWorkTime(workIntervals);

    // Функции для вычисления статистик
    const sum = arr => arr.reduce((a, b) => a + b, 0);
    const average = arr => sum(arr) / arr.length;
    const median = arr => {
        const sorted = [...arr].sort((a, b) => a - b);
        const mid = Math.floor(sorted.length / 2);
        return sorted.length % 2 !== 0 ? sorted[mid] : (sorted[mid - 1] + sorted[mid]) / 2;
    };

    if (workDurations.length > 0) {
        // Вычисление минимального, максимального, среднего и медианного значения
        const minInterval = Math.min(...workDurations);
        const maxInterval = Math.max(...workDurations);
        const avgInterval = average(workDurations);
        const medInterval = median(workDurations);

        // Подготовка диагностической информации
        //const diagnosticInfo = workIntervals.map((interval, index) => {
        //    return `Интервал ${index + 1}: ${interval.start} - ${interval.end}, Длительность: ${secondsToTime(workDurations[index])}`;
        //}).join('\n');

        // Отображение результатов в виде таблицы
        dv.table(["Параметр", "Значение"], [
            ["Количество рабочих периодов", workDurations.length],
            ["Минимальный рабочий интервал", secondsToTime(minInterval)],
            ["Максимальный рабочий интервал", secondsToTime(maxInterval)],
            ["Средний рабочий интервал", secondsToTime(avgInterval)],
            ["Медианный рабочий интервал", secondsToTime(medInterval)],
            ["Суммарное рабочее время за день", secondsToTime(totalWorkTime)],
            //["Диагностическая информация", diagnosticInfo]
        ]);
    } else {
        dv.span(`Не удалось вычислить интервалы рабочего времени.`);
    }
});

```

# Рабочие задачи:
## [[Рабочие задачи на сегодня|Список на день]]
## [[Запланированные рабочие задачи|Список на будущее]]
## Выполненные:
```tasks
done on <% tp.file.title %>
path includes Рабочие
group by filename
hide backlink
```

# Личные дела:
## [[Личные дела на сегодня|Список на день]]
## [[Запланированные личные дела|Список на будущее]]
## Выполненные:
```tasks
done on <% tp.file.title %>
path does not include templates
path does not include Рабочие
group by filename
hide backlink
```

