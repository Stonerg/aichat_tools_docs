```
# Core Concepts

## JavaScript Functions
Your custom tool is essentially a JavaScript function.

## UI Inputs
You define inputs for your tool through a User Interface (UI). These inputs become available as variables within your function's scope.

## Pre-available Libraries
A range of common Node.js libraries are available for use.

## String Return Value
Your function must ultimately return a single string value. This string is what will be passed back to the LLM.

# Writing Your Function

Your code will be the body of an asynchronous JavaScript function.

```javascript
// Your entire code will be wrapped in an async function implicitly
// For example:
// async function() {
//   // Your code here
//   return "some string result";
// }
// You just need to write the body.
```

## 1. Accessing Inputs from the UI

When you configure your custom tool, you'll define input fields. If you create an input field with a property name like `searchQuery`, you can access its value in your JavaScript code using a variable prefixed with `$:`.

- **Property Name in UI:** `searchQuery`
- **Variable in Code:** `$searchQuery`

- **Property Name in UI:** `userId`
- **Variable in Code:** `$userId`

**Example:**
If you have UI inputs `city` and `country`:

```javascript
const location = `User wants weather for ${$city}, ${$country}`;
// ... use location variable
```

## 2. Using Available Libraries

You can use `require()` to import common Node.js libraries that are pre-available in the execution environment. Some commonly used libraries include:

-   `node-fetch`: For making HTTP requests to external APIs.
-   `cheerio`: For parsing and manipulating HTML/XML (web scraping).
-   `tls`: For low-level TLS/SSL secure connections.
-   Other standard Node.js built-in modules (e.g., `crypto`, `path`, `fs` - though filesystem access might be restricted for security).

**Example:**

```javascript
const fetch = require('node-fetch'); // For making HTTP requests
const cheerio = require('cheerio');   // For parsing HTML
```

## 3. Asynchronous Operations

Many tools will involve asynchronous operations like API calls. Use `async` and `await` to handle these. Your entire custom tool code is implicitly wrapped in an `async` function, so you can use `await` at the top level of your script.

**Example:**

```javascript
const fetch = require('node-fetch');
const url = `https://api.example.com/data?param=${$someInput}`; // $someInput from UI

try {
    const response = await fetch(url);
    const data = await response.json(); // Or response.text()
    // Process data
    return JSON.stringify(data); // Remember to return a string
} catch (error) {
    return `Error fetching data: ${error.message}`; // Return error string to LLM
}
```

## 4. Returning a Value

Your custom tool function **must** return a single string value.

-   If your function naturally produces a string (e.g., a summary, a direct answer), return it directly.
-   If your function produces an object or an array (e.g., JSON data from an API, scraped results), you must convert it to a string before returning. `JSON.stringify()` is commonly used for this.

**Example returning a simple string:**

```javascript
const name = $userName || "Guest"; // $userName from UI
return `Hello, ${name}!`;
```

**Example returning stringified JSON:**

```javascript
const fetch = require('node-fetch');
const userId = $userId; // $userId from UI
// Assume API_KEY is hardcoded for this example, or managed by the platform if possible
const API_KEY = "YOUR_HARDCODED_API_KEY"; // Not ideal, see Best Practices

try {
    const response = await fetch(`https://api.example.com/users/${userId}`, {
        headers: { 'Authorization': `Bearer ${API_KEY}` } // Example usage
    });
    const userData = await response.json();
    return JSON.stringify(userData); // Convert the object to a JSON string
} catch (error) {
    return `Error fetching user data: ${error.message}`;
}
```

## 5. Error Handling

It's crucial to implement robust error handling using `try...catch` blocks. If an error occurs, your function should catch it and return a meaningful error message as a string. This prevents the entire flow from crashing and provides feedback to the LLM.

**Example:**

```javascript
const fetch = require('node-fetch');

if (!$searchTerm) { // $searchTerm from UI
    return "Error: Search term is a required input.";
}

const url = `https://api.example.com/search?q=${encodeURIComponent($searchTerm)}`;

try {
    const response = await fetch(url);
    if (!response.ok) {
        // Handle HTTP errors (e.g., 404, 500)
        return `API Error: ${response.status} - ${response.statusText}`;
    }
    const results = await response.json();
    return JSON.stringify(results);
} catch (error) {
    return `An unexpected error occurred: ${error.message}`; // For the LLM
}
```

# Example Scenarios:

## 1. Fetching Weather Data (Simple API Call)

**UI Inputs:** `city` (string)

```javascript
const fetch = require('node-fetch');
const city = $city;

const weatherApiKey = "YOUR_ACTUAL_HARDCODED_API_KEY_HERE";

if (!city) {
    return "Error: City is a required input.";
}

const url = `https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(city)}&appid=${weatherApiKey}&units=metric`;

try {
    const response = await fetch(url);
    const data = await response.json();

    if (response.ok) {
        const weatherDescription = data.weather[0].description;
        const temperature = data.main.temp;
        return `Current weather in ${data.name}: ${weatherDescription}, Temperature: ${temperature}°C.`;
    } else {
        return `Error fetching weather: ${data.message || response.statusText}`;
    }
} catch (error) {
    return `Failed to get weather data: ${error.message}`;
}
```

## 2. Basic Web Scraper

**UI Inputs:** `pageUrl` (string), `selector` (string, e.g., `h1`)

```javascript
const fetch = require('node-fetch');
const cheerio = require('cheerio');

const urlToScrape = $pageUrl;
const cssSelector = $selector;

if (!urlToScrape || !cssSelector) {
    return "Error: pageUrl and selector are required inputs.";
}

try {
    const response = await fetch(urlToScrape, { headers: {'User-Agent': 'CustomToolBot/1.0'} });
    if (!response.ok) {
        return `HTTP error! Status: ${response.status} while fetching ${urlToScrape}`;
    }
    const html = await response.text();
    const $ = cheerio.load(html);

    const elements = [];
    $(cssSelector).each((i, elem) => {
        elements.push($(elem).text().trim());
    });

    if (elements.length === 0) {
        return `No elements found with selector "${cssSelector}" on page ${urlToScrape}.`;
    }

    // Return as a string, potentially stringified JSON if more structure is needed
    return JSON.stringify({
        count: elements.length,
        foundElements: elements.slice(0, 5) // Return first 5 to keep string manageable
    });
} catch (error) {
    return `Error scraping page: ${error.message}`;
}
```

## 3. Data Validation (e.g., Phone Number - simplified)

**UI Inputs:** `phoneNumber` (string)

```javascript
const phoneNumber = $phoneNumber;

if (!phoneNumber) {
    return JSON.stringify({ isValid: false, error: "Phone number is required." });
}

// Basic validation: check if it contains only digits, +, (), - and has a reasonable length
const cleaned = phoneNumber.replace(/[\s()-]/g, ''); // Remove spaces, parens, hyphens

if (!/^\+?\d{7,15}$/.test(cleaned)) {
    return JSON.stringify({
        isValid: false,
        phoneNumber: phoneNumber,
        error: "Invalid phone number format or length."
    });
}

return JSON.stringify({
    isValid: true,
    phoneNumber: cleaned,
    note: "Phone number appears valid (basic check)."
});
```

# Best Practices

-   **Validate Inputs:** Always check if required UI inputs (like `$yourInput`) are provided and are in the expected format.
-   **Handle Errors Gracefully:** Use `try...catch` blocks and return informative error messages as strings. The LLM will receive these error messages.
-   **Keep Tools Focused:** Design tools to perform a specific task well.
-   **Return Strings:** Always ensure your function's final output is a string. Use `JSON.stringify()` for complex objects/arrays.
-   **Consider LLM Consumption:** The string returned will be processed by an LLM. Format it clearly. For large amounts of data, consider summarizing.
-   **Be Mindful of Execution Time:** Long-running operations can lead to timeouts. Optimize your code and external calls.
```
