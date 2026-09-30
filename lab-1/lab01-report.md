# Лабораторная работа №1

## Среда

* **ОС:** Windows 11
* **Версия клиента:** 29.8.1
* **Версия сервера:** 29.8.1
* **Базовый образ:** `ngnix:alpine`

## Задания

### 1) Версии и теги

* Определение версии клиента и демона Docker

  ```bash
  docker version
  ```

  ![docker version](1.png)

* Выбранный образ в конкретном теге: `nginx:1.31-alpine`
  
  ```bash
  docker run --name select -d nginx:1.31-alpine
  ```
  
  ![select](2.png)

### 2) Первый запуск сервиса (detached)

* Запуск контейнера в фоновом режиме с именем `lab-web-hearwy` и доступом по `http://localhost:8080`.

  ```bash
  docker run -d --name lab-web-hearwy -p 8080:80 nginx:alpine
  ```
  
  ![8080](3.png)

* Перезапуск сервиса.

  ```bash
  #Остановка контейнера
  docker stop lab-web-hearwy

  #Удаление контейнера
  docker rm lab-web-hearwy
  ```

* Перезапуск контейнера на новом порту.

  ```bash
  docker run -d --name lab-web-hearwy -p 9090:80 nginx:alpine
  ```
  
  ![9090](4.png)

* Список работающих контейнеров

  ![truecont](5.png)

### 3) Том (bind‑mount): сайт/данные из папки хоста

* Создание папки на хосте с тестовым файлом.

  ```bash
  #Создание папки 
  mkdir docker\site

  #Создание тестового файла с подписью
  "Funny" | Set-Content .\site\index.html
  ```

* Перезапуск `lab-web-hearwy`, смонтировав папку как `read‑only`.
  ```bash
  #Остановка контейнера
  docker stop lab-web-hearwy

  #Удаление контейнера
  docker rm lab-web-hearwy

  #Поднимаем nginx и монтируем папку read-only (":ro")
  docker run -d --name lab-web-hearwy -p 8080:80 -v "${PWD}\site:/usr/share/nginx/html:ro" nginx:alpine
  ```

  * Страничка до:
    
    ![do](6.png)
    
* Изменение файла на хосте.

  ```bash
  "Bad" | Set-Content .\site\index.html
  ```
  
  * Страничка после:

    ![posle](7.png)

* Почему данные переживут удаление контейнера?

  * dasd
  
### 4) Интерактив / exec
* Вход внутрь работающего контейнера `lab-web-hearwy` интерактивно.

  ```bash
  docker exec -it lab-web-hearwy /bin/sh
  ```

* Просмотр содержимого каталога со статикой.

  ```bash
  #Переход в папку 
  cd /usr/share/nginx/html

  #Просмотр каталога со статикой
  ls -l 
  ```

  ![listing](8.png)

* Попытка создать файл внутри каталога статики при read-only.
  
  ```bash
  #Попытка создания тестового файла
  touch test.txt
  ```

  ![create ro](9.png)

* Перезапуск lab-web-hearwy с тем же bind-mount без :ro (read-write) и повторение создания файла.

  ```bash
  #Остановка контейнера
  docker stop lab-web-hearwy

  #Удаление контейнера
  docker rm lab-web-hearwy

  #Перезапуск lab-web-hearwy с тем же bind-mount без :ro
  docker run -d --name lab-web-hearwy -p 8080:80 -v "${PWD}\site:/usr/share/nginx/html" nginx:alpine

  #Вход в интерактивный режим
  docker exec -it lab-web-hearwy /bin/sh

  #Переход в папку 
  cd /usr/share/nginx/html

  #Создание тестового файла
  touch test.txt
  ```
  
  ![create noro](10.png)

* Проверка файла на хосте после режима RW.

  ![create noro](11.png)

### 5) Логи и attach

* Генерация HTTP-запросов: обновление страницы через curl.
  
  ```bash
  curl http://localhost:8080
  curl http://localhost:8080/nope
  curl http://localhost:8080/nope1
  curl http://localhost:8080/nope2
  ```
  
* Посмотр последних 10 строк журнала контейнера.
  
  ```bash
  docker logs --tail 10 lab-web-hearwy
  ```

  ![logs](12.png)

* Прикрепление к основному процессу контейнера, затем корректное отстыкование, не останавливая его.

  ```bash
  docker attach lab-web-hearwy
  ```
  
  * Для выхода из attach, не останавливая контейнер, необходимо нажать Ctrl + P, затем Ctrl + Q
  * Остановка основного процесса контейнера происходит засчет нажатия Ctrl + C

### 6) Краткоживущий процесс

* Запуск контейнера, который сразу завершится после выполнения вывода строки.

  ```bash
  docker run --name speedtest alpine echo "Hello!"
  ```

  ![speedtest](13.png)

* Почему он завершился сам?

### 7) Inspect

* Получение подробной информации по lab-web-hearwy.
  
  ```bash
  docker run --name speedtest alpine echo "Hello!"
  ```

* Вырезка Ports:

  ![Ports](14.png)
  
* Вырезка Mounts:

  ![Mounts](15.png)

### 8) Чистка

* Корректная остановка и удаление контейнеров с именами из этой лабораторной работы.

  ```bash
  #Просмотр всех контейнеров
  docker ps -a

  #Удаление всех контейнеров через остановку -f
  docker rm -f speedtest lab-web-hearwy select 
  ```

* Удаление одного из использованных образов.

   ```bash
  #Просмотр всех образов
  docker images

  #Удаление образа 
  docker rmi nginx:1.31-alpine 
  ```

* Итоговый список контейнеров/образов:

  ![Clear](16.png)

