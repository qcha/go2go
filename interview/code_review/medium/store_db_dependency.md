# Инициализация хранилища с sql.DB

## Условие

Есть конструктор хранилища истории статусов заказа:

```go
type OrderStatusHistoryStore struct {
    db *sql.DB
}

func NewStore(db *sql.DB) (*OrderStatusHistoryStore, error) {
    db, err := sql.Open("postgres", connStr)
    if err != nil {
        return nil, fmt.Errorf("failed open db: %w", err)
    }

    if err := db.Ping(); err != nil {
        return nil, fmt.Errorf("ping failed: %w", err)
    }

    return &OrderStatusHistoryStore{db: db}, nil
}
```

Импорты, драйвер базы данных и `connStr` для краткости опущены.

Вопросы:

* Какие вы видите проблемы в этом коде?
* Что можно исправить?

## Решение

### Область ответственности

Взглянем на сигнатуру метода:

```go
func NewStore(db *sql.DB) (*OrderStatusHistoryStore, error) {
```

По сигнатуре кажется, что подключение уже создано снаружи и передается в store как зависимость. Но внутри конструктор открывает новое подключение:

```go
db, err := sql.Open("postgres", connStr)
```

В итоге параметр `db` оказывается бесполезным, а ответственность конструктора становится неясной.

Чаще всего `*sql.DB` - это пул соединений, который создается один раз на уровне приложения. Там его открывают, настраивают, проверяют, передают в разные хранилища и закрывают при завершении приложения.

Конкретный store не должен знать `connStr`, не должен сам вызывать `sql.Open` и не должен владеть жизненным циклом общего пула соединений.

### Потенциально не закрытое соединение

Еще одна проблема в текущем коде: если `Ping` завершится ошибкой, открытый `db` не закрывается.

```go
if err := db.Ping(); err != nil {
    return nil, fmt.Errorf("ping failed: %w", err)
}
```

Если код сам создал `db` через `sql.Open` (как в нашем примере), то при ошибке после открытия нужно вызвать `db.Close()`. Но лучше вообще не открывать подключение внутри конструктора store.

Корректное решение должно сделать три вещи:

* использовать переданный `*sql.DB`;
* убрать `sql.Open` из конструктора store;
* оставить открытие, проверку и закрытие подключения уровню приложения.

### Исправленный вариант

```go
type OrderStatusHistoryStore struct {
    db *sql.DB
}

func NewStore(db *sql.DB) *OrderStatusHistoryStore {
    return &OrderStatusHistoryStore{db: db}
}
```

Такой конструктор просто сохраняет зависимость.

Если в проекте принято явно валидировать зависимости в конструкторах, можно вернуть ошибку при `nil`:

```go
func NewStore(db *sql.DB) (*OrderStatusHistoryStore, error) {
    if db == nil {
        return nil, errors.New("db is nil")
    }

    return &OrderStatusHistoryStore{db: db}, nil
}
```

Открытие, проверка и закрытие подключения остаются выше, например при старте приложения:

```go
db, err := sql.Open("postgres", connStr)
if err != nil {
    return fmt.Errorf("open db: %w", err)
}

if err := db.Ping(); err != nil {
    db.Close()
    return fmt.Errorf("ping db: %w", err)
}

store := NewStore(db)
```

## Итого

Технически можно сделать конструктор, который принимает `connStr` и сам вызывает `sql.Open`, но это плохой подход. Это смешивает конструктор объекта с логикой приложения (логикой старта), усложняет переиспользование общего пула соединений, тестирование и корректное закрытие ресурсов.
