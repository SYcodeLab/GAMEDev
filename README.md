# decSec OS — SWX Language v7.0.0

> **SWX** (Shadow Web eXploit) — язык программирования с уникальным синтаксисом,  
> встроенной криптографией, HTML-рендерингом и системным доступом.  
> Автор: **Slywerx** | decSec Project | 2025

---

## Быстрый старт

```bash
# Требования: Node.js >= 18.0.0

# REPL (интерактивный терминал)
node decsec.js

# Запуск файла
node decsec.js examples/hello.swx

# Inline выполнение
node decsec.js -e "wx #x.sha256('hello')"

# Watch режим (перезапуск при изменении файла)
node decsec.js -w script.swx

# Глобальная установка
npm link
swx              # → REPL
swx script.swx   # → запуск файла
```

---

## Синтаксис SWX

### Переменные
```swx
swx x = 42
swx name = "ghost_{x}"    swx# интерполяция
swx arr  = [1, 2, 3]
swx obj  = { key: "val" }

swx# переприсвоение
x = 10
x ~= 5    swx# x += 5
x -= 2
x *~ 3    swx# x *= 3
x /~ 4    swx# x /= 4
```

### Вывод
```swx
wx "Hello {name}"
```

### Условия
```swx
sw x >~ 18 {
    wx "взрослый"
} dr sw x >~ 13 {
    wx "подросток"
} dr {
    wx "ребёнок"
}

swx# Логика: sw&(И)  sw|(ИЛИ)  sw!(НЕ)
sw a =? b sw& c !? d { wx "ok" }
```

### Циклы
```swx
swx# Повторить N раз (_i = индекс, _n = кол-во)
xs 5 { wx "итерация {_i}" }

swx# Перебор массива (_i = индекс)
swx arr = ["a", "b", "c"]
xw item in arr { wx "{_i}: {item}" }

swx# Перебор объекта
swx obj = { x: 1, y: 2 }
xw pair in obj { wx pair.key + " = " + str(pair.val) }
```

### Функции
```swx
sx add(a, b = 0) => {
    ws a + b
}
swx result = add(3, 4)

swx# Однострочная
sx double(n) => n * 2
```

### Ошибки
```swx
x? {
    swx data = #f.read("file.swx")
    wx data
} x!(err) {
    wx "Ошибка: {err}"
} x. {
    wx "finally выполнен"
}

swx! "пользовательская ошибка"
```

### Импорт/Экспорт
```swx
s> "stdlib"         swx# загрузить stdlib/stdlib.swx
s> "./mymodule"     swx# относительный путь
w> myFunc           swx# экспортировать функцию
```

### Комментарии
```swx
swx# однострочный комментарий
// тоже однострочный
/* блочный
   комментарий */
```

---

## Операторы

| Оператор | Значение |
|----------|----------|
| `=?`     | равно (==) |
| `!?`     | не равно (!=) |
| `<~`     | меньше или равно (<=) |
| `>~`     | больше или равно (>=) |
| `sw&`    | логическое И (&&) |
| `sw|`    | логическое ИЛИ (\|\|) |
| `sw!`    | логическое НЕ (!) |
| `~=`     | += |
| `*~`     | *= |
| `/~`     | /= |
| `**`     | возведение в степень |

---

## Веб-рендеринг (sx>)

```swx
sx> "output/index.html" {
    @style {
        body   >> bg(#030810) color(#fff) font(Courier New, monospace) pad(24px)
        .card  >> bg(#0c1420) pad(20px) radius(8px) border(1px solid #0d3050)
        h1     >> color(#00e5ff) size(24px) bold uppercase
        .btn   >> bg(#00e5ff) color(#000) pad(10px 20px) pointer radius(4px)
        .grid  >> grid cols(1fr 1fr) gap(16px)
        swx# Флаги: bold italic flex grid center pointer rounded uppercase
    }
    @body {
        div.card >> {
            h1 >> "Заголовок {title}"
            p  >> "Параграф"
            button.btn >> (onclick: "alert('SWX')") { "Нажми" }
        }
    }
    @script {
        wx "Скрипт выполнен"
    }
}
```

### CSS shorthand

| SWX         | CSS                   |
|-------------|----------------------|
| `bg(#000)`  | `background: #000`    |
| `color(#fff)` | `color: #fff`        |
| `size(16px)` | `font-size: 16px`    |
| `pad(20px)` | `padding: 20px`       |
| `mar(0 auto)` | `margin: 0 auto`    |
| `radius(8px)` | `border-radius: 8px` |
| `grid`      | `display: grid`       |
| `flex`      | `display: flex`       |
| `bold`      | `font-weight: bold`   |
| `center`    | `text-align: center`  |
| `pointer`   | `cursor: pointer`     |
| `uppercase` | `text-transform: uppercase` |

---

## Пространства имён

### `#x` — Криптография
```swx
#x.sha256(s)           #x.sha512(s)    #x.sha3(s)     #x.md5(s)
#x.hmac(key, msg)      #x.ripemd160(s) #x.crc32(s)
#x.aes_enc(text, key)  #x.aes_dec(cipher, key)
#x.rsa_keys([bits])    #x.rsa_sign(data, priv)   #x.rsa_verify(data, sig, pub)
#x.hex(s)     #x.unhex(h)    #x.base64(s)   #x.unbase64(b)
#x.xor(msg, key)   #x.caesar(s, n)   #x.rot13(s)   #x.vigenere(text, key)
#x.genkey([n])  #x.geniv()  #x.randbytes(n)  #x.uuid()
#x.entropy(s)   #x.pwd_strength(p)
#x.token_create(obj, key [, ttl])   #x.token_verify(token, key)
```

### `#f` — Файловая система
```swx
#f.read(path)       #f.write(path, data)    #f.append(path, data)
#f.exists(path)     #f.list([dir])          #f.stat(path)
#f.mkdir(path)      #f.delete(path)         #f.rename(a, b)
#f.json_read(path)  #f.json_write(path, obj)
```

### `#s` — Система
```swx
#s.time()    #s.date()    #s.datetime()   #s.timestamp()
#s.platform() #s.arch()   #s.pid()        #s.cwd()
#s.env(key)  #s.env_set(k, v)  #s.args()  #s.exit([code])
#s.hostname() #s.user()   #s.homedir()
```

### `#n` — Сеть
```swx
#n.get(url)         #n.post(url, data)    #n.ping(host)
```

### `#ai` — ИИ (в разработке)
```swx
#ai.prompt(text)    #ai.analyze(data)     #ai.grade(code)
```

---

## Встроенные функции

### Типы
`str` `num` `int` `bool` `type` `is_num` `is_str` `is_arr` `is_null` `parse` `json`

### Математика
`floor` `ceil` `round` `abs` `sqrt` `cbrt` `pow` `log` `log2` `log10`
`sin` `cos` `tan` `max` `min` `clamp` `rand` `randf` `even` `odd`

### Строки
`len` `upper` `lower` `trim` `triml` `trimr` `split` `join` `replace`
`includes` `starts` `ends` `index_of` `slice` `repeat` `pad` `padr`
`chars` `lines` `words` `count` `format` `regex_test` `regex_match`
`to_hex` `to_bin` `from_hex` `from_bin`

### Массивы
`push` `pop` `shift` `unshift` `reverse` `sort` `flat` `unique` `contains`
`first` `last` `nth` `range` `chunk` `zip` `flatten` `shuffle`

### Объекты
`keys` `values` `entries` `has_key` `merge` `del_key` `from_entries`

---

## Структура проекта

```
decsec/
├── decsec.js          ← точка входа / REPL терминал
├── package.json
├── README.md
├── src/
│   ├── lexer.js       ← лексический анализатор
│   ├── parser.js      ← синтаксический анализатор (AST)
│   ├── interpreter.js ← интерпретатор (tree-walking)
│   └── crypto.js      ← криптографический движок (Node.js crypto)
├── stdlib/
│   └── stdlib.swx     ← стандартная библиотека
├── examples/
│   ├── hello.swx      ← базовый синтаксис
│   ├── crypto.swx     ← криптография
│   ├── web.swx        ← веб-рендеринг
│   └── stdlib_demo.swx ← демо stdlib
└── output/            ← сюда сохраняются HTML файлы
```

---

## REPL команды

```
:help            полный справочник
:run  <файл>     запустить .swx файл (новый интерпретатор)
:load <файл>     загрузить в текущий контекст
:watch <файл>    следить за файлом, перезапускать при сохранении
:new  <файл>     создать новый .swx файл
:edit <файл>     показать файл с подсветкой синтаксиса
:save <имя>      сохранить набранный код сессии в файл
:clear           очистить экран
:reset           сбросить интерпретатор
:vars            показать переменные
:fns             показать функции
:ls [dir]        список файлов
:pwd             текущая директория
:cd <dir>        сменить директорию
:type <expr>     тип выражения
:ast <код>       AST дерево
:tokens <код>    токены лексера
:time <код>      время выполнения
:bench N <код>   N итераций: avg / min / max
:history [n]     история команд
:session save    сохранить сессию (~/.swx_session.json)
:session load    восстановить сессию
```

---

## ~/.swxrc

Файл автозагрузки при старте REPL:

```swx
swx# ~/.swxrc
swx AUTHOR = "ghost_7x"
swx PROJECT = "decSec"
s> "stdlib"
```

---

## Требования

- **Node.js >= 18.0.0**
- Нет внешних зависимостей (только встроенные модули Node.js)

---

MIT License — decSec Project 2025 — Slywerx

---

## Что нового в v7.0.0

### Новые конструкции языка
- **`xl` (while цикл)** — `xl condition { ... }` — выполняет пока условие истинно
- **`match / case / default`** — сопоставление с образцом
- **`break`** — выйти из цикла
- **`next`** — перейти к следующей итерации
- **`|>` (pipe оператор)** — `val |> func` эквивалентно `func(val)`

### Исправления
- Арифметика `n - 1` больше не конфликтует с CSS-идентификаторами
- CSS классы с дефисом (`card-title`, `btn-primary`) парсятся корректно
- `xl` (while) безопасен: лимит 1M итераций
- `stdlib.swx` полностью переработан под v7 синтаксис

### Примеры v7
```swx
swx# while
xl x > 0 { x -= 1 }

swx# match/case
match status {
    case 200 => { wx "OK" }
    case 404 => { wx "Not Found" }
    default  => { wx "Other" }
}

swx# pipe
swx result = "hello" |> upper    swx# = HELLO
swx n = arr |> len               swx# = длина массива

swx# break / next
xs 100 {
    sw _i > 10 { break }
    sw _i % 2 !? 0 { next }
    wx _i
}
```
