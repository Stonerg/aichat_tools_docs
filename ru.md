```markdown
# Основные концепции

## Функции JavaScript
Ваш пользовательский инструмент — это, по сути, функция JavaScript.

## Входные данные из UI
Вы определяете входные данные для вашего инструмента через Пользовательский Интерфейс (UI). Эти входные данные становятся доступными как переменные в области видимости вашей функции.

## Предустановленные библиотеки
Доступен ряд распространенных библиотек Node.js.

## Возвращаемое строковое значение
Ваша функция в конечном итоге должна вернуть одно строковое значение. Эта строка будет передана обратно в LLM.

# Написание вашей функции

Ваш код будет телом асинхронной функции JavaScript.

```javascript
// Весь ваш код будет неявно обернут в асинхронную функцию
// Например:
// async function() {
//   // Ваш код здесь
//   return "какой-то строковый результат";
// }
// Вам нужно написать только тело функции.
```

## 1. Доступ к входным данным из UI

При настройке вашего пользовательского инструмента вы определяете поля ввода. Если вы создаете поле ввода с именем свойства, например `searchQuery`, вы можете получить доступ к его значению в вашем коде JavaScript, используя переменную с префиксом `$:`.

- **Имя свойства в UI:** `searchQuery`
- **Переменная в коде:** `$searchQuery`

- **Имя свойства в UI:** `userId`
- **Переменная в коде:** `$userId`

**Пример:**
Если у вас есть входные данные UI `city` (город) и `country` (страна):

```javascript
const location = `Пользователь хочет погоду для ${$city}, ${$country}`;
// ... используйте переменную location
```

## 2. Использование доступных библиотек

Вы можете использовать `require()` для импорта распространенных библиотек Node.js, которые предустановлены в среде выполнения. Некоторые часто используемые библиотеки включают:

-   `node-fetch`: для выполнения HTTP-запросов к внешним API.
-   `cheerio`: для парсинга и манипулирования HTML/XML (веб-скрейпинг).
-   `tls`: для низкоуровневых безопасных соединений TLS/SSL.
-   Другие стандартные встроенные модули Node.js (например, `crypto`, `path`, `fs` - хотя доступ к файловой системе может быть ограничен из соображений безопасности).

**Пример:**

```javascript
const fetch = require('node-fetch'); // Для выполнения HTTP-запросов
const cheerio = require('cheerio');   // Для парсинга HTML
```

## 3. Асинхронные операции

Многие инструменты будут включать асинхронные операции, такие как вызовы API. Используйте `async` и `await` для их обработки. Весь код вашего пользовательского инструмента неявно обернут в функцию `async`, поэтому вы можете использовать `await` на верхнем уровне вашего скрипта.

**Пример:**

```javascript
const fetch = require('node-fetch');
const url = `https://api.example.com/data?param=${$someInput}`; // $someInput из UI

try {
    const response = await fetch(url);
    const data = await response.json(); // Или response.text()
    // Обработка данных
    return JSON.stringify(data); // Помните, что нужно вернуть строку
} catch (error) {
    return `Ошибка при получении данных: ${error.message}`; // Вернуть строку ошибки в LLM
}
```

## 4. Возврат значения

Функция вашего пользовательского инструмента **должна** возвращать одно строковое значение.

-   Если ваша функция естественным образом производит строку (например, сводку, прямой ответ), верните ее напрямую.
-   Если ваша функция производит объект или массив (например, данные JSON из API, результаты скрейпинга), вы должны преобразовать их в строку перед возвратом. `JSON.stringify()` обычно используется для этого.

**Пример возврата простой строки:**

```javascript
const name = $userName || "Гость"; // $userName из UI
return `Привет, ${name}!`;
```

**Пример возврата строки JSON:**

```javascript
const fetch = require('node-fetch');
const userId = $userId; // $userId из UI
const API_KEY = "API KEY";

try {
    const response = await fetch(`https://api.example.com/users/${userId}`, {
        headers: { 'Authorization': `Bearer ${API_KEY}` } // Пример использования
    });
    const userData = await response.json();
    return JSON.stringify(userData); // Преобразовать объект в строку JSON
} catch (error) {
    return `Ошибка при получении данных пользователя: ${error.message}`;
}
```

## 5. Обработка ошибок

Крайне важно реализовать надежную обработку ошибок с использованием блоков `try...catch`. Если возникает ошибка, ваша функция должна перехватить ее и вернуть осмысленное сообщение об ошибке в виде строки. Это предотвращает сбой всего потока и предоставляет обратную связь LLM.

**Пример:**

```javascript
const fetch = require('node-fetch');

if (!$searchTerm) { // $searchTerm из UI
    return "Ошибка: Поисковый запрос является обязательным полем.";
}

const url = `https://api.example.com/search?q=${encodeURIComponent($searchTerm)}`;

try {
    const response = await fetch(url);
    if (!response.ok) {
        // Обработка HTTP-ошибок (например, 404, 500)
        return `Ошибка API: ${response.status} - ${response.statusText}`;
    }
    const results = await response.json();
    return JSON.stringify(results);
} catch (error) {
    return `Произошла непредвиденная ошибка: ${error.message}`; // Для LLM
}
```

# Примеры сценариев:

## 1. Получение данных о погоде (простой вызов API)

**Входные данные UI:** `city` (строка)

```javascript
const fetch = require('node-fetch');
const city = $city;

const weatherApiKey = "API_KEY";

if (!city) {
    return "Ошибка: Город является обязательным полем.";
}

const url = `https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(city)}&appid=${weatherApiKey}&units=metric`;

try {
    const response = await fetch(url);
    const data = await response.json();

    if (response.ok) {
        const weatherDescription = data.weather[0].description;
        const temperature = data.main.temp;
        return `Текущая погода в ${data.name}: ${weatherDescription}, Температура: ${temperature}°C.`;
    } else {
        return `Ошибка при получении погоды: ${data.message || response.statusText}`;
    }
} catch (error) {
    return `Не удалось получить данные о погоде: ${error.message}`;
}
```

## 2. Базовый веб-скрейпер

**Входные данные UI:** `pageUrl` (строка), `selector` (строка, например, `h1`)

```javascript
const fetch = require('node-fetch');
const cheerio = require('cheerio');

const urlToScrape = $pageUrl;
const cssSelector = $selector;

if (!urlToScrape || !cssSelector) {
    return "Ошибка: pageUrl и selector являются обязательными полями.";
}

try {
    const response = await fetch(urlToScrape, { headers: {'User-Agent': 'CustomToolBot/1.0'} });
    if (!response.ok) {
        return `HTTP-ошибка! Статус: ${response.status} при получении ${urlToScrape}`;
    }
    const html = await response.text();
    const $ = cheerio.load(html);

    const elements = [];
    $(cssSelector).each((i, elem) => {
        elements.push($(elem).text().trim());
    });

    if (elements.length === 0) {
        return `Элементы с селектором "${cssSelector}" не найдены на странице ${urlToScrape}.`;
    }

    // Вернуть как строку, возможно, в формате JSON, если требуется больше структуры
    return JSON.stringify({
        count: elements.length,
        foundElements: elements.slice(0, 5) // Вернуть первые 5, чтобы строка оставалась управляемой
    });
} catch (error) {
    return `Ошибка при скрейпинге страницы: ${error.message}`;
}
```

## 3. Валидация данных (например, номер телефона - упрощенно)

**Входные данные UI:** `phoneNumber` (строка)

```javascript
const phoneNumber = $phoneNumber;

if (!phoneNumber) {
    return JSON.stringify({ isValid: false, error: "Номер телефона обязателен." });
}

// Базовая валидация: проверка, содержит ли только цифры, +, (), - и имеет разумную длину
const cleaned = phoneNumber.replace(/[\s()-]/g, ''); // Удалить пробелы, скобки, дефисы
if (!/^\+?\d{7,15}$/.test(cleaned)) {
    return JSON.stringify({
        isValid: false,
        phoneNumber: phoneNumber,
        error: "Неверный формат или длина номера телефона."
    });
}
return JSON.stringify({
    isValid: true,
    phoneNumber: cleaned,
    note: "Номер телефона выглядит действительным (базовая проверка)."
});
```

# Рекомендации

-   **Проверяйте входные данные:** Всегда проверяйте, предоставлены ли обязательные входные данные из UI (например, `$yourInput`) и соответствуют ли они ожидаемому формату.
-   **Корректно обрабатывайте ошибки:** Используйте блоки `try...catch` и возвращайте информативные сообщения об ошибках в виде строк. LLM получит эти сообщения об ошибках.
-   **Делайте инструменты сфокусированными:** Разрабатывайте инструменты так, чтобы они хорошо выполняли одну конкретную задачу.
-   **Возвращайте строки:** Всегда убеждайтесь, что конечным результатом вашей функции является строка. Используйте `JSON.stringify()` для сложных объектов/массивов.
-   **Учитывайте потребление LLM:** Возвращаемая строка будет обработана LLM. Форматируйте ее четко. Для больших объемов данных рассмотрите возможность суммирования.
-   **Помните о времени выполнения:** Длительные операции могут привести к тайм-аутам. Оптимизируйте свой код и внешние вызовы.
```
