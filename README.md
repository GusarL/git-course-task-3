# Последовательность действий:

1. Копирую чистый репозиторий, без рабочей папки, на локальную машину
   git clone --bare https://github.com/MaksymSemenykhin/git-course-task.git
2. Захожу на сервер
   ```
   cd .ssh
   ssh -i "aws_user.ssh" lhus@devopstest.pp.ua.com
   ```
3. Делаю папку git в srv на сервере
   ```
   cd ..
   cd ..
   cd srv
   mkdir git 
   ```
4. Делаю доступ на запись 
   ```
   sudo chmod -R 777 /srv 
   ```
5. В папке, куда склонился репозиторий на локальной машине, копирую склоненный репозиторий на сервер
   ```
   scp -r -i ~/.ssh/aws_user.ssh git-course-task.git lhus@devopstest.pp.ua:/srv/git
   ```
6. На сервере в папке srv
  ```
  git init --bare --shared
  ```
7. Настраиваю возможность заходить на сервер не указвая ключ (ключ прописываю в config) 
   ```
   ls -a
   cd home/.ssh/config
   nano config
   Добавила 
   Host devopstest.pp.ua
   IdentityFile ~/.ssh/aws_user.ssh
   ```
8. Добавила ключ task3.pub на сервер
   ```
   ssh lhus@devopstest.pp.ua:/srv/git
   cd .ssh/authorized_keys  
   ```
   скопировала содержимое файла task3.pub и сохранила
9. Добавила пользователя git
   sudo adduser git
   su git
   cd
   mkdir .ssh && chmod 700 .ssh
   touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys 
   пароль 1111 
10. Добавила ключ task3.pub для пользователя git в .ssh/authorized_keys.                   
11. Инициализировала голый репозиторий в srv/git
    ```
    cd /srv/git
    mkdir git-course-task.git
    git-course-task.git
    git init --bare
    ```
12. Ограничила права пользователя git
    ```
    cat /etc/shells   
    which git-shell   
    sudo -e /etc/shells
    sudo chsh git -s $(which git-shell)
    ```
13. Дала права записи в папку var/www/task3
    ```
    sudo chown -R `whoami`:`id -gn` /var/www/task3
    ```
14. Перехожу в srv/git/git-course-task.git
    ```
    nano .git/hooks/post-receive
    ```
15. Пишу bash-script, который как только сервер получить пуш, перебирает рефы и ищет изменения в master, 
    если находит, обновляет папку /var/www/task
    ```
    #!/bin/bash
    while read oldrev newrev ref
    do
        if [[ $ref =~ .*/master$ ]];
        then
            echo "Master ref received.  Deploying master branch to production..."
            git --work-tree=/var/www/task3 --git-dir=/srv/git/git-course-task checkout -f
        else
            echo "Ref $ref successfully received.  Doing nothing: only the master branch may be deployed on this server."
        fi
    done
    ```
16. Делаю файл исполняемым
    ```
    chmod +x hooks/post-receive
    ```
17. Добавляю удалённую вертку production 
    ```
     git remote add production git@devopstest.pp.ua:/srv/git/git-course-task.git
     ```
18. Из локальной папки
    ```
    git push production master
    ```
    Сообщение remote: Master ref received.  Deploying master branch... появляется, папка var/www/task3 обновляется{         

