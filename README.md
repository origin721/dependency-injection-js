**Простой и понятный DI-контейнер**, который поддерживает:  
- **Lazy Loading (отложенную инициализацию)**  
- **Области видимости (scopes)**  
- **Подмену зависимостей (например, для тестов)**  
- **Авто-инжекцию (не нужно передавать зависимости вручную)**  

---

### **📌 Простая реализация DI-контейнера**
```javascript
class DIContainer {
  constructor() {
    this.providers = new Map();
    this.instances = new Map();
  }

  // Регистрируем сервис (фабрика, а не объект!)
  register(key, factory, scope = "singleton") {
    this.providers.set(key, { factory, scope });
  }

  // Получаем инстанс (ленивая инициализация)
  resolve(key) {
    if (!this.providers.has(key)) {
      throw new Error(`Сервис ${key} не зарегистрирован`);
    }

    const { factory, scope } = this.providers.get(key);

    if (scope === "singleton") {
      if (!this.instances.has(key)) {
        this.instances.set(key, factory(this)); // Передаём DI-контейнер в фабрику
      }
      return this.instances.get(key);
    }

    return factory(this); // Новый инстанс для каждого вызова
  }

  // Подмена зависимостей (например, для тестов)
  override(key, mockInstance) {
    this.instances.set(key, mockInstance);
  }
}

// Создаём глобальный DI-контейнер
export const di = new DIContainer();
```

---

### **📌 Использование DI-контейнера**
#### **1️⃣ Регистрируем сервисы**
```javascript
// Логгер (простой сервис)
class Logger {
  log(message) {
    console.log(`[LOG]: ${message}`);
  }
}

// API-сервис (использует Logger, передаваемый через DI)
class ApiService {
  constructor(logger) {
    this.logger = logger;
  }

  fetchData() {
    this.logger.log("Запрос данных...");
    return { data: "Пример ответа" };
  }
}

// Регистрируем сервисы в DI-контейнере
di.register("logger", () => new Logger(), "singleton");
di.register("apiService", (di) => new ApiService(di.resolve("logger")), "singleton");
```

---

#### **2️⃣ Используем сервисы через DI**
```javascript
import { di } from "./di.js";

const api = di.resolve("apiService"); // Получаем ApiService с авто-инжекцией Logger
api.fetchData(); // Логгер автоматически используется внутри сервиса
```

---

#### **3️⃣ Подменяем зависимости (например, в тестах)**
```javascript
import { di } from "./di.js";

// Моковый логгер
class MockLogger {
  log(message) {
    console.log(`[MOCK LOG]: ${message}`);
  }
}

// Подменяем логгер перед тестами
di.override("logger", new MockLogger());

const api = di.resolve("apiService");
api.fetchData(); // Теперь использует моковый логгер
```

---

### **📌 Что мы получили?**
✅ **Lazy Loading** – сервис создаётся **только при первом вызове**.  
✅ **Авто-инжекция** – ApiService **сам получает зависимости** через `di.resolve()`.  
✅ **Глобальный и локальный скоуп** – можно делать singleton или новый инстанс.  
✅ **Легко заменять зависимости (например, для тестов)**.  

---

### **🔹 Итог**
Этот DI-контейнер даёт **ключевые преимущества Angular DI**, но остаётся **простым и удобным**. Он не требует классов `@Injectable` и аннотаций, но сохраняет ленивую загрузку, авто-инжекцию и подмену зависимостей.  

Теперь **React, Vue, Svelte или просто Node.js могут использовать DI так же, как Angular**. 🚀

(Есть либа InversifyJS)