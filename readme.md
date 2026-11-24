# Практична робота № 2

**Дисципліна:** Основи побудови інформаційних систем та мереж (ОК-13)  
**Тема:** Структура повідомлень прикладного протоколу HTTP. Формування запиту вручну  
**Студент:** Кіріяк Павло Олександрович  
**Група:** 1  
**Варіант:** 5  
**Домен:** `postgresql.org`  
**Середовище:** Killercoda, Ubuntu 24.04  
**Дата експерименту:** 24.11.2026

## Мета роботи

Дослідити структуру HTTP-повідомлень, сформувати запити без HTTP-клієнта, порівняти реакції сервера на зміну заголовків і визначити ознаки проміжних вузлів.

## Частина A. Збір експериментальних даних

### A.1 — Формування запиту вручну

Інтерактивне введення не вдалося завершити: сеанс закривався до введення всіх полів. Тому запит сформовано вручну як текст і передано через `printf` до `nc`.

Текст запиту (після останнього заголовка — порожній рядок):

```http
GET / HTTP/1.1
Host: postgresql.org
Connection: close

```

Команда:

```bash
printf 'GET / HTTP/1.1\r\nHost: postgresql.org\r\nConnection: close\r\n\r\n' | nc -v -w 30 postgresql.org 80 2>&1 | tee raw/a1-manual.txt
```

Вивід:

```text
Connection to postgresql.org (217.196.149.50) 80 port [tcp/http] succeeded!
HTTP/1.1 301 Moved Permanently
Date: Tue, 24 Nov 2026 08:09:58 GMT
Server: Varnish
X-Varnish: 426606665
Location: https://www.postgresql.org/
Content-Length: 0
Connection: close
```

### A.2 — Запит без Host

Команда:

```bash
printf 'GET / HTTP/1.1\r\nConnection: close\r\n\r\n' | nc -v -w 30 postgresql.org 80 2>&1 | tee raw/a2-no-host.txt
```

Вивід:

```text
Connection to postgresql.org (217.196.149.50) 80 port [tcp/http] succeeded!
HTTP/1.1 301 Moved Permanently
Date: Tue, 24 Nov 2026 08:10:58 GMT
Server: Varnish
X-Varnish: 426147983
Location: https://www.postgresql.org/
Content-Length: 0
Connection: close
```

### A.3.1 — Сторонній домен

Команда:

```bash
printf 'GET / HTTP/1.1\r\nHost: w3.org\r\nConnection: close\r\n\r\n' | nc -v -w 30 postgresql.org 80 2>&1 | tee raw/a3-1-foreign-host.txt
```

Вивід:

```text
Connection to postgresql.org (217.196.149.50) 80 port [tcp/http] succeeded!
HTTP/1.1 301 Moved Permanently
Date: Tue, 24 Nov 2026 08:11:42 GMT
Server: Varnish
X-Varnish: 415433447
Location: https://www.postgresql.org/
Content-Length: 0
Connection: close
```

### A.3.2 — Неіснуюче ім’я

Команда:

```bash
printf 'GET / HTTP/1.1\r\nHost: opism-pr02.invalid\r\nConnection: close\r\n\r\n' | nc -v -w 30 postgresql.org 80 2>&1 | tee raw/a3-2-invalid-host.txt
```

Вивід:

```text
Connection to postgresql.org (217.196.149.50) 80 port [tcp/http] succeeded!
HTTP/1.1 301 Moved Permanently
Date: Tue, 24 Nov 2026 08:12:21 GMT
Server: Varnish
X-Varnish: 416809368
Location: https://www.postgresql.org/
Content-Length: 0
Connection: close
```

### A.3.3 — HTTP/1.0

Команда:

```bash
printf 'GET / HTTP/1.0\r\n\r\n' | nc -v -w 30 postgresql.org 80 2>&1 | tee raw/a3-3-http10.txt
```

Вивід:

```text
Connection to postgresql.org (217.196.149.50) 80 port [tcp/http] succeeded!
HTTP/1.1 301 Moved Permanently
Date: Tue, 24 Nov 2026 08:13:03 GMT
Server: Varnish
X-Varnish: 404558002
Location: https://www.postgresql.org/
Content-Length: 0
Connection: close
```

### A.4 — Два запити в одному з’єднанні

Команда:

```bash
printf 'GET /opism-pr02-12345 HTTP/1.1\r\nHost: postgresql.org\r\n\r\nGET / HTTP/1.1\r\nHost: postgresql.org\r\nConnection: close\r\n\r\n' | nc -v -w 30 postgresql.org 80 2>&1 | tee raw/a4-pipeline.txt
```

Вивід:

```text
Connection to postgresql.org (217.196.149.50) 80 port [tcp/http] succeeded!
HTTP/1.1 301 Moved Permanently
Date: Tue, 24 Nov 2026 08:13:49 GMT
Server: Varnish
X-Varnish: 430243882
Location: https://www.postgresql.org/opism-pr02-12345
Content-Length: 0
Connection: keep-alive

HTTP/1.1 301 Moved Permanently
Date: Tue, 24 Nov 2026 08:13:49 GMT
Server: Varnish
X-Varnish: 430243883
Location: https://www.postgresql.org/
Content-Length: 0
Connection: close
```

### A.5 — Запит за допомогою curl

Команда:

```bash
curl -v --http1.1 http://postgresql.org/ -o /dev/null 2>&1 | tee raw/a5-curl.txt
```

Вивід:

```text
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host postgresql.org:80 was resolved.
* IPv6: 2a02:16a8:dc51::50
* IPv4: 217.196.149.50
*   Trying 217.196.149.50:80...
* Connected to postgresql.org (217.196.149.50) port 80
> GET / HTTP/1.1
> Host: postgresql.org
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 301 Moved Permanently
< Date: Tue, 24 Nov 2026 08:14:27 GMT
< Server: Varnish
< X-Varnish: 409896599
< Location: https://www.postgresql.org/
< Content-Length: 0
< Connection: keep-alive
< 
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
* Connection #0 to host postgresql.org left intact
```

### A.6 — Запит через TLS

Той самий текст запиту передано через `printf` із завершеннями CRLF до OpenSSL.

Команда:

```bash
printf 'GET / HTTP/1.1\r\nHost: postgresql.org\r\nConnection: close\r\n\r\n' | openssl s_client -connect postgresql.org:443 -servername postgresql.org -quiet 2>&1 | tee raw/a6-tls.txt
```

Вивід:

```text
depth=3 C = US, O = Internet Security Research Group, CN = ISRG Root X1
verify return:1
depth=2 C = US, O = ISRG, CN = Root YR
verify return:1
depth=1 C = US, O = Let's Encrypt, CN = YR1
verify return:1
depth=0 CN = www.postgresql.org
verify return:1
HTTP/1.1 301 Moved Permanently
Date: Tue, 24 Nov 2026 08:14:57 GMT
Server: Varnish
X-Varnish: 427917425
Location: https://www.postgresql.org/
Content-Length: 0
Connection: close

40A73DD781720000:error:0A000126:SSL routines:ssl3_read_n:unexpected eof while reading:../ssl/record/rec_layer_s3.c:316:
```

У A.4 отримано дві відповіді 301; перша містить `Connection: keep-alive`, друга — `Connection: close`. У A.6 HTTP-відповідь отримано, після неї OpenSSL повідомив про неочікуване завершення TLS. Перевірку відповідності імені сертифіката окремим параметром не виконано.

### Зведення результатів (додаток Д)

Вузол: `postgresql.org` (`217.196.149.50`), порт 80.

| Проба | Значення Host | Версія | Код стану | Обсяг тіла | Збігається з A.1 |
|---|---|---|---|---|---|
| A.1 | postgresql.org | 1.1 | 301 | 0 Б | — |
| A.2 | відсутнє | 1.1 | 301 | 0 Б | так |
| A.3.1 | w3.org | 1.1 | 301 | 0 Б | так |
| A.3.2 | opism-pr02.invalid | 1.1 | 301 | 0 Б | так |
| A.3.3 | відсутнє | 1.0 | 301 | 0 Б | так |

Змінювалися значення Host, його наявність і версія запиту. В усіх пробах код стану, адреса перенаправлення та обсяг тіла збіглися. Значення Date і X-Varnish відрізнялися, тому збіг не є побайтовим.

## Частина B. Розбір полів заголовка (додаток Б)

Загальна кількість полів відповіді A.1: **6**. Рядок стану не є полем заголовка.

| № | Поле | Значення | Призначення | Походження | Обґрунтування |
|---|---|---|---|---|---|
| 1 | Date | Tue, 24 Nov 2026 08:09:58 GMT | Час формування повідомлення | Не визначено | Час може встановити сервер або проміжний вузол |
| 2 | Server | Varnish | Назва програмного забезпечення, що обробляє відповідь | Проміжний вузол (імовірно) | Varnish є програмним забезпеченням зворотного проксі |
| 3 | X-Varnish | 426606665 | Ідентифікатор обробки запиту у Varnish | Проміжний вузол (імовірно) | Назва поля пов’язана з Varnish; значення змінюється між запитами |
| 4 | Location | https://www.postgresql.org/ | Адреса перенаправлення клієнта | Не визначено | Правило перенаправлення може бути налаштоване на різних вузлах |
| 5 | Content-Length | 0 | Розмір тіла відповіді в байтах | Не визначено | Відповідь без тіла може сформувати як проксі, так і кінцевий сервер |
| 6 | Connection | close | Вказівка закрити поточне з’єднання | Не визначено | Поле стосується поточного з’єднання; точний вузол, що його сформував, не встановлено |

## Частина C. Обговорення

Таблиці частин A і B підготовлено для аудиторного обговорення. Результати обговорення не зафіксовано.

## Контрольні питання

Нижче наведено чернетки відповідей для самостійної перевірки та формулювання студентом.

1. Порожній рядок (CRLF CRLF) позначає кінець заголовка. До нього отримувач не знає, чи будуть додаткові поля. У моєму успішному запиті завершення передано через printf; окремого спостереження очікування саме порожнього рядка не зафіксовано.
2. У всіх п’яти пробах отримано `HTTP/1.1 301 Moved Permanently` та `Location: https://www.postgresql.org/`. Поле Host дає змогу визначити потрібний віртуальний вузол на спільній IP-адресі. У цьому експерименті перенаправлення не залежало від його значення. Це не означає, що Host необов’язковий у HTTP/1.1.
3. Отримано дві відповіді 301. У першій Location містить `/opism-pr02-12345`, у другій — `/`. Код не змінився, але шлях збережено. Це узгоджується з роллю вузла перенаправлення на HTTPS. Повторне використання одного TCP-з’єднання зменшує витрати на встановлення нових з’єднань для вкладених ресурсів.
4. Curl додав `User-Agent: curl/8.5.0` (ідентифікація клієнта) та `Accept: */*` (готовність приймати будь-який тип вмісту). Також автоматично сформував обов’язкове для HTTP/1.1 поле `Host: postgresql.org`, яке в A.1 було задано вручну.
5. Поля `Server: Varnish` та `X-Varnish: 426606665` є ознаками ймовірної участі проміжного вузла Varnish. Вони не доводять повну топологію мережі.

### Питання 6 (додаток В)

| № | Рядок виводу | Джерело |
|---|---|---|
| 1 | `X-Varnish: 426606665` | A.1 |
| 2 | `> User-Agent: curl/8.5.0` | A.5 |
| 3 | `depth=1 C = US, O = Let's Encrypt, CN = YR1` | A.6 |

## Частина D. Висновки (чернетка для перевірки студентом)

**D.1.** Найбільш несподіваним результатом є однакова реакція вузла на різні варіанти HTTP-запиту. У завданнях A.1, A.2 та A.3.1–A.3.3 отримано рядок `HTTP/1.1 301 Moved Permanently`, хоча змінювалися значення поля Host, його наявність і версія протоколу. Адреса перенаправлення також залишалася однаковою. Це дає підстави припустити, що на порту 80 діє загальне правило перенаправлення на HTTPS. У A.4 вузол зберіг запитаний шлях у полі Location, а також повернув дві відповіді через одне TCP-з’єднання. У A.6 перенаправлення спостерігалося навіть після встановлення TLS, але вже на домен із префіксом www.

**D.2.** Найскладніше визначити походження поля Location. Його значення вказує на наступну адресу звернення, проте не розкриває, де саме сформовано правило перенаправлення. Наявність полів Server і X-Varnish свідчить про ймовірну участь проміжного вузла, але не дає достатньої підстави приписати йому всі поля відповіді. Тому походження Location позначено як невизначене.

**D.3.** Залишилося питання, чи виконує Varnish перенаправлення самостійно, чи отримує відповідну вказівку від кінцевого сервера. Для відповіді потрібні додаткові дані про налаштування вузлів або журнали обробки запитів. Наявні результати дозволяють описати зовнішню поведінку системи, але не встановити її внутрішню будову однозначно.

## Декларування використання ШІ

**Рівень:** Р3 — ШІ як співвиконавець.  
**Засіб:** ChatGPT.

ШІ допомагав розібрати методичні вимоги, підготувати команди для Ubuntu, діагностувати помилки введення, проаналізувати фактичні відповіді сервера та сформувати чернетку звіту.

| № | Розділ | Дослівний промпт |
|---|---|---|
| 1 | Аналіз методички | що ти скажеш стосовно цієї практичної |
| 2 | Частина A | Надіслані зображення термінала без текстового промпту |
| 3 | Оформлення | Надісланий файл results.txt без текстового промпту |

Перелік потрібно доповнити іншими фактично надісланими промптами, якщо вони були використані.

**Дії студента з результатом:** студент виконав команди в Killercoda, повторював невдалі спроби, зберіг виводи та передав їх для аналізу. Перед поданням необхідно самостійно перевірити таблиці, сформулювати контрольні відповіді й висновки та підтвердити повноту декларації.

**Автентичність:** усі наведені експериментальні виводи перенесено з наданого студентом файла results.txt; синтетичні відповіді сервера не додавалися.
