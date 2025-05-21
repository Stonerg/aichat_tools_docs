```markdown
# Concepte de Bază

## Funcții JavaScript
Instrumentul dvs. personalizat este, în esență, o funcție JavaScript.

## Intrări UI
Definiți intrările pentru instrumentul dvs. printr-o Interfață Utilizator (UI). Aceste intrări devin disponibile ca variabile în cadrul domeniului de vizibilitate al funcției dvs.

## Biblioteci Pre-disponibile
O gamă de biblioteci Node.js comune sunt disponibile pentru utilizare.

## Valoare String Returnată
Funcția dvs. trebuie să returneze în final o singură valoare de tip string. Acest string este cel care va fi transmis înapoi către LLM.

# Scrierea Funcției Dvs.

Codul dvs. va fi corpul unei funcții JavaScript asincrone.

```javascript
// Întregul dvs. cod va fi încapsulat implicit într-o funcție asincronă
// De exemplu:
// async function() {
//   // Codul dvs. aici
//   return "un rezultat de tip string";
// }
// Trebuie doar să scrieți corpul funcției.
```

## 1. Accesarea Intrărilor din UI

Când configurați instrumentul dvs. personalizat, veți defini câmpuri de intrare. Dacă creați un câmp de intrare cu un nume de proprietate precum `searchQuery`, puteți accesa valoarea sa în codul JavaScript folosind o variabilă prefixată cu `$:`.

- **Nume Proprietate în UI:** `searchQuery`
- **Variabilă în Cod:** `$searchQuery`

- **Nume Proprietate în UI:** `userId`
- **Variabilă în Cod:** `$userId`

**Exemplu:**
Dacă aveți intrări UI `city` (oraș) și `country` (țară):

```javascript
const location = `Utilizatorul dorește vremea pentru ${$city}, ${$country}`;
// ... utilizați variabila location
```

## 2. Utilizarea Bibliotecilor Disponibile

Puteți utiliza `require()` pentru a importa biblioteci Node.js comune care sunt pre-disponibile în mediul de execuție. Unele biblioteci utilizate frecvent includ:

-   `node-fetch`: Pentru efectuarea de cereri HTTP către API-uri externe.
-   `cheerio`: Pentru parsarea și manipularea HTML/XML (web scraping).
-   `tls`: Pentru conexiuni securizate TLS/SSL de nivel scăzut.
-   Alte module standard încorporate în Node.js (de ex., `crypto`, `path`, `fs` - deși accesul la sistemul de fișiere ar putea fi restricționat din motive de securitate).

**Exemplu:**

```javascript
const fetch = require('node-fetch'); // Pentru efectuarea de cereri HTTP
const cheerio = require('cheerio');   // Pentru parsarea HTML
```

## 3. Operații Asincrone

Multe instrumente vor implica operații asincrone, cum ar fi apelurile API. Utilizați `async` și `await` pentru a le gestiona. Întregul cod al instrumentului dvs. personalizat este încapsulat implicit într-o funcție `async`, astfel încât puteți utiliza `await` la nivelul superior al scriptului dvs.

**Exemplu:**

```javascript
const fetch = require('node-fetch');
const url = `https://api.example.com/data?param=${$someInput}`; // $someInput din UI

try {
    const response = await fetch(url);
    const data = await response.json(); // Sau response.text()
    // Procesează datele
    return JSON.stringify(data); // Nu uitați să returnați un string
} catch (error) {
    return `Eroare la preluarea datelor: ${error.message}`; // Returnează string-ul erorii către LLM
}
```

## 4. Returnarea unei Valori

Funcția instrumentului dvs. personalizat **trebuie** să returneze o singură valoare de tip string.

-   Dacă funcția dvs. produce în mod natural un string (de ex., un rezumat, un răspuns direct), returnați-l direct.
-   Dacă funcția dvs. produce un obiect sau un array (de ex., date JSON de la un API, rezultate extrase), trebuie să îl convertiți într-un string înainte de a-l returna. `JSON.stringify()` este frecvent utilizat în acest scop.

**Exemplu de returnare a unui string simplu:**

```javascript
const name = $userName || "Oaspete"; // $userName din UI
return `Salut, ${name}!`;
```

**Exemplu de returnare a unui JSON stringificat:**

```javascript
const fetch = require('node-fetch');
const userId = $userId; // $userId din UI
const API_KEY = "api_key";

try {
    const response = await fetch(`https://api.example.com/users/${userId}`, {
        headers: { 'Authorization': `Bearer ${API_KEY}` } // Exemplu de utilizare
    });
    const userData = await response.json();
    return JSON.stringify(userData); // Convertește obiectul într-un string JSON
} catch (error) {
    return `Eroare la preluarea datelor utilizatorului: ${error.message}`;
}
```

## 5. Gestionarea Erorilor

Este crucial să implementați o gestionare robustă a erorilor folosind blocuri `try...catch`. Dacă apare o eroare, funcția dvs. ar trebui să o prindă și să returneze un mesaj de eroare semnificativ sub formă de string. Acest lucru previne blocarea întregului flux și oferă feedback către LLM.

**Exemplu:**

```javascript
const fetch = require('node-fetch');

if (!$searchTerm) { // $searchTerm din UI
    return "Eroare: Termenul de căutare este o intrare obligatorie.";
}

const url = `https://api.example.com/search?q=${encodeURIComponent($searchTerm)}`;

try {
    const response = await fetch(url);
    if (!response.ok) {
        // Gestionează erorile HTTP (de ex., 404, 500)
        return `Eroare API: ${response.status} - ${response.statusText}`;
    }
    const results = await response.json();
    return JSON.stringify(results);
} catch (error) {
    return `A apărut o eroare neașteptată: ${error.message}`; // Pentru LLM
}
```

# Scenarii Exemplu:

## 1. Preluarea Datelor Meteo (Apel API Simplu)

**Intrări UI:** `city` (string)

```javascript
const fetch = require('node-fetch');
const city = $city;

const weatherApiKey = "api_key";

if (!city) {
    return "Eroare: Orașul este o intrare obligatorie.";
}

const url = `https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(city)}&appid=${weatherApiKey}&units=metric`;

try {
    const response = await fetch(url);
    const data = await response.json();

    if (response.ok) {
        const weatherDescription = data.weather[0].description;
        const temperature = data.main.temp;
        return `Vremea curentă în ${data.name}: ${weatherDescription}, Temperatură: ${temperature}°C.`;
    } else {
        return `Eroare la preluarea datelor meteo: ${data.message || response.statusText}`;
    }
} catch (error) {
    return `Preluarea datelor meteo a eșuat: ${error.message}`;
}
```

## 2. Extractor Web de Bază

**Intrări UI:** `pageUrl` (string), `selector` (string, de ex., `h1`)

```javascript
const fetch = require('node-fetch');
const cheerio = require('cheerio');

const urlToScrape = $pageUrl;
const cssSelector = $selector;

if (!urlToScrape || !cssSelector) {
    return "Eroare: pageUrl și selector sunt intrări obligatorii.";
}

try {
    const response = await fetch(urlToScrape, { headers: {'User-Agent': 'CustomToolBot/1.0'} });
    if (!response.ok) {
        return `Eroare HTTP! Stare: ${response.status} la preluarea ${urlToScrape}`;
    }
    const html = await response.text();
    const $ = cheerio.load(html);

    const elements = [];
    $(cssSelector).each((i, elem) => {
        elements.push($(elem).text().trim());
    });

    if (elements.length === 0) {
        return `Niciun element găsit cu selectorul "${cssSelector}" pe pagina ${urlToScrape}.`;
    }

    // Returnează ca string, potențial JSON stringificat dacă este necesară mai multă structură
    return JSON.stringify({
        count: elements.length,
        foundElements: elements.slice(0, 5) // Returnează primele 5 pentru a menține string-ul gestionabil
    });
} catch (error) {
    return `Eroare la extragerea paginii: ${error.message}`;
}
```

## 3. Validarea Datelor (de ex., Număr de Telefon - simplificat)

**Intrări UI:** `phoneNumber` (string)

```javascript
const phoneNumber = $phoneNumber;

if (!phoneNumber) {
    return JSON.stringify({ isValid: false, error: "Numărul de telefon este obligatoriu." });
}

// Validare de bază: verifică dacă conține doar cifre, +, (), - și are o lungime rezonabilă
const cleaned = phoneNumber.replace(/[\s()-]/g, ''); // Elimină spațiile, parantezele, cratimele
if (!/^\+?\d{7,15}$/.test(cleaned)) {
    return JSON.stringify({
        isValid: false,
        phoneNumber: phoneNumber,
        error: "Format sau lungime invalidă a numărului de telefon."
    });
}
return JSON.stringify({
    isValid: true,
    phoneNumber: cleaned,
    note: "Numărul de telefon pare valid (verificare de bază)."
});
```

# Cele Mai Bune Practici

-   **Validați Intrările:** Verificați întotdeauna dacă intrările UI necesare (cum ar fi `$yourInput`) sunt furnizate și sunt în formatul așteptat.
-   **Gestionați Erorile Elegant:** Utilizați blocuri `try...catch` și returnați mesaje de eroare informative sub formă de string-uri. LLM-ul va primi aceste mesaje de eroare.
-   **Mențineți Instrumentele Concentrate:** Proiectați instrumente pentru a îndeplini bine o sarcină specifică.
-   **Returnați String-uri:** Asigurați-vă întotdeauna că rezultatul final al funcției dvs. este un string. Utilizați `JSON.stringify()` pentru obiecte/array-uri complexe.
-   **Luați în Considerare Consumul LLM:** String-ul returnat va fi procesat de un LLM. Formatați-l clar. Pentru cantități mari de date, luați în considerare rezumarea.
-   **Fiți Atent la Timpul de Execuție:** Operațiunile de lungă durată pot duce la timeout-uri. Optimizați codul și apelurile externe.
```
