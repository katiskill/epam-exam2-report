# epam-exam2-report
В рамках данной задачи вы должны создать окружение и инфраструктуру для CI/CD, а также настроить pipeline для сборки и деплоя простого web-приложения.

Задача:  
1. Форкнуть GitHub репозиторий (web-приложение) по адресу https://github.com/merovigen/student-exam2 в личный GitHub репозиторий.  
2. Ознакомиться с документацией к проекту и с кодом приложения. Разобраться как запускать тесты приложения и само приложение. Работающее приложение должно выводить страницу в браузере с калькулятором на JS и хостнеймом сервера, на котором оно запущено.  
> Репозиторий: <https://github.com/a-chernov/student-exam2>
3. Написать Dockerfile для web-приложения. Образ, собранный из этого Dockerfile, должен содержать все файлы/утилиты/библиотеки, необходимые для работы web-приложения, а также само приложение. При запуске контейнера на основе данного образа, должно запускаться web-приложение внутри контейнера с экспортированным на host-машину портом, то есть команда `curl 127.0.0.1:[PORT]`, выполненная с host-машины (centos) должна возвращать такой же результат, как и в п.2.  
> [Dockerfile](https://github.com/a-chernov/student-exam2/blob/master/Dockerfile)  
> Сборка образа и запуск контейнера из директории с Dockerfile:
> ```
> docker build -t achernov101/epam-exam:flask .
> docker run -d -p 5000:5000 --hostname flask-in-docker achernov101/epam-exam:flask
> ```
4. Запустить и выполнить первоначальную настройку Jenkins, работающего внутри docker-контейнера. 
> Для запуска контейнера был создан файл [docker-compose.yml](https://github.com/a-chernov/epam-exam2-report/blob/master/04/docker-compose.yml)  
> Контейнер запускается из директории с этим файлом командой: `docker-compose up`
5. Создать пользователей admin и developer, включить matrix access и настроить права доступа:  
    1) admin – полные права на все 
    2) developer: 
        * Overall: read 
        * Job: build, cancel, discover, read, workspace 
        * Agent: build  
> [Скриншот](https://github.com/a-chernov/epam-exam2-report/blob/master/05/shot.png)
6. Установить плагин Ansible и настроить в Global Tool Configuration путь до директории с ansible который установлен на агенте (подробнее в п.7).
> [Установленный плагин](https://github.com/a-chernov/epam-exam2-report/blob/master/06/shot01.png) и [настройка плагина](https://github.com/a-chernov/epam-exam2-report/blob/master/06/shot02.png)
7. Для сборки и тестирования web-приложения нужно использовать не Jenkins-мастер, установленный и настроенный ранее, а специальный Jenkins-агент, работающий в отдельном docker-контейнере. Для начала нужно подготовить образ этого агента. Для этого требуется написать Dockerfile для Jenkins-агента. В образе должны присутствовать: java8, ansible, python3, пользователь Jenkins и ключ для авторизации Jenkins-мастера. Способ подключения агента – ssh.
> [Dockerfile](https://github.com/a-chernov/epam-exam2-report/blob/master/07/Dockerfile)  
> Сборка образа из директории с Dockerfile:
> ```
> docker build -t achernov101/epam-exam:jenkins-slave .
> ```
> При сборке генирируются ssh-ключи для пользователя jenkins. В stdout выводится приватный ssh-ключ, который потребуется в п. 9.
8. Зарегистрировать аккаунт на https://hub.docker.com/ и создать приватный репозиторий, в котором должны храниться созданные в п.3 и п.7 образы.
> Загрузка образов в репозиторий:
> ```
> docker login
> docker push achernov101/epam-exam:flask
> docker push achernov101/epam-exam:jenkins-slave
> ```
> [Образы на Docker Hub](https://github.com/a-chernov/epam-exam2-report/blob/master/08/shot.png)
9. Добавить Jenkins credentials (SSH Username with private key) для авторизации на агенте. 
> [Скриншот](https://github.com/a-chernov/epam-exam2-report/blob/master/09/shot.png)
10. Запустить контейнер для Jenkins агента и подключить агента к мастеру Jenkins. 
> Запуск контейнера:
> ```
> docker run --privileged -it achernov101/epam-exam:jenkins-slave
> ```
> Ключ `--privileged` потребовался для работы демона docker. К сожалению, проброс сокета docker'а не сработал, поэтому пришлось добавлять docker непосредственно в образ jenkins-агента (docker-in-docker). Docker требуется в jenkins-агенте для сборки образов по CI-пайплайну (п. 12).  
> При запуске контейнера требуется запуск демонов ssh и docker:
> ```
> service ssh restart; service docker start
> ```
> [Настройка агента](https://github.com/a-chernov/epam-exam2-report/blob/master/10/shot01.png) и [запущенные мастер/агент](https://github.com/a-chernov/epam-exam2-report/blob/master/10/shot02.png)
11. Запретить сборки на мастере Jenkins (настроить число executors на мастере равным 0).
> [Скриншот](https://github.com/a-chernov/epam-exam2-report/blob/master/11/shot.png)
12. Создать Jenkins pipeline для CI: 
    1) Jenkinsfile должен браться из репозитория с web-приложением 
    2) Jenkins должен запускать job для каждого нового коммита в master ветку – для этого нужно включить триггер на SCM Poll и поставить ежеминутную проверку 
    3) В Jenkinsfile должно быть: 
        * Запускаться python тесты 
        * Билдится docker image
        * Аутентификация в docker hub (docker login, логин и пароль должны браться из credentials Jenkins’а) 
        * Выгрузка image на docker hub (docker push)  
> [Jenkinsfile](https://github.com/a-chernov/student-exam2/blob/master/Jenkinsfile)  
> Для хранения пароля от аккаунта на Docker Hub используется [credentials](https://github.com/a-chernov/epam-exam2-report/blob/master/12/shot00.png).  
> [Настроенный pipeline с ежеминутной проверкой](https://github.com/a-chernov/epam-exam2-report/blob/master/12/shot01.png)  
> [Успешный результат выполнения pipeline](https://github.com/a-chernov/epam-exam2-report/blob/master/12/shot02.png) — [текстовый лог](https://github.com/a-chernov/epam-exam2-report/blob/master/12/consoleText-1) и [появившийся образ на Docker Hub](https://github.com/a-chernov/epam-exam2-report/blob/master/12/shot03.png)  
> Сделаем коммит в master. Консоль jenkins:  
> ```
> jenkins_1  | Jan 31, 2019 8:32:01 PM hudson.triggers.SCMTrigger$Runner run
> jenkins_1  | INFO: SCM changes detected in ci-task. Triggering  #2
> ```
> [Успешный результат выполнения pipeline после нового коммита](https://github.com/a-chernov/epam-exam2-report/blob/master/12/shot04.png) — [текстовый лог](https://github.com/a-chernov/epam-exam2-report/blob/master/12/consoleText-2) и [появившийся образ на Docker Hub](https://github.com/a-chernov/epam-exam2-report/blob/master/12/shot05.png)
13. Создать Git репозиторий с Ansible. Добавить в него список используемых серверов в inventory, роль для установки вашего web-приложение и роль для установки балансировщика на базе Nginx.  
14. Роль для web-приложения должна устанавливать docker, скачивать образ web-приложения и запускать контейнер на определенном порту.  
15. Роль для Nginx должна устанавливать docker, скачивать образ Nginx, генерировать конфигурационный файл Nginx из Jinja2 шаблона для балансировки ваших web-приложений.  
16. В результате работы Ansible у вас должны задеплоиться в разных контейнерах балансировщик и 3 web-приложения к нему для балансировки на разных портах.  
17. Вынести роль для установки docker в отдельную роль и переиспользовать её в других ролях.  
18. Вынести роли в отдельные репозитории и скачивать их через ansible-galaxy.
> Ссылки на репозитории с ролями имеются в файле [requirements.yml](https://github.com/a-chernov/ansible-exam/blob/master/requirements.yml), который используется ansible-galaxy для установки этих ролей из их репозиториев.
19. Создать новую docker сеть и поместить в нее балансировщик и приложения. 
> Docker сеть *deploy* (172.18.0.0/16), в которую будут помещены все 4 контейнера, создаётся в роли [flask-deploy-role](https://github.com/a-chernov/flask-deploy-role/blob/master/tasks/main.yml) (3-8 строки).
20. Создать Jenkins pipeline для CD: 
    1) Jenkinsfile должен браться из репозитория с ansible 
    2) В Jenkinsfile должно быть 
    3) Запуск плейбука ansible для деплоя 
    4) Запуск интеграционных тестов (достаточно обычного http реквеста до web сервера и проверка что он возвращает статус 200)
21. Проверить работоспособность pipeline для CI и CD.
> [Jenkinsfile](https://github.com/a-chernov/ansible-exam/blob/master/Jenkinsfile)  
> 4 контейнера разворачиваются на хостовой ОС, где уже развернуты 2 контейнера (jenkins-мастер и jenkins-агент). В соответствии с заданием п. 14 для скачивания образа из приватного репозитория в роли flask-deploy-role для пароля от Docker Hub используется шифрование переменной с помощью ansible-vault. Для хранения vault-пароля используется [credentials](https://github.com/a-chernov/epam-exam2-report/blob/master/21/shot01.png)  
> [Успешный результат выполнения pipeline](https://github.com/a-chernov/epam-exam2-report/blob/master/21/shot02.png) — [текстовый лог](https://github.com/a-chernov/epam-exam2-report/blob/master/21/consoleText) и [список контейнеров на хостовой ОС](https://github.com/a-chernov/epam-exam2-report/blob/master/21/shot03.png)  
22. Проверить работоспособность приложения и балансировщика.
> Успешная работа балансировщика (вывод нескольких запросов к веб-серверу подряд) — [текстовый лог](https://github.com/a-chernov/epam-exam2-report/blob/master/22/curl_log). По выводимому веб-страницей hostname (25, 82, 139, 196, 253 и 310 строки) видно, что балансировщик переадресовывает запросы к разным контейнерам с веб-приложением.
