## Стенд для домашнего занятия "Управление пакетами. Дистрибьюция софта
### Сборка собственного rpm пакета и размещение его в собственном репозитории
#### Сборка кастомного nginx...
##### Установим необходимый софт
    sudo yum install -y -q epel-release # необязательно, нужно для mock, который здесь в итоге не использовался
    sudo yum install -y -q git rpm-build rpmdevtools gcc make automake yum-utils
##### Установим необходимые для компиляции nginx devel-файлы
    sudo yum-builddep -y -q nginx # хорошая команда, заменила мне ручную установку зависимостей
##### Скачиваем rpm с исходниками nginx. Добавим репозиторий nginx (https://nginx.org/ru/linux_packages.html)

    cat <<'EOF1' | sudo tee /etc/yum.repos.d/nginx.repo
     [nginx]
     name=nginx repo
     baseurl=http://nginx.org/packages/mainline/centos/8/$basearch/
     gpgcheck=0
     enabled=1
     [nginx-source]
     name=nginx source repo
     baseurl=http://nginx.org/packages/mainline/centos/8/SRPMS/
     gpgcheck=0
     enabled=1
    EOF1

##### Скачиваем исходники nginx, создаем в папке $HOME директории для сборки и распаковываем туда nginx.src.rpm
    yumdownloader --source nginx
    rpmdev-setuptree
    rpm -ivh /home/vagrant/nginx-* # команду можно подрихтовать, чтобы не все nginx-* файлы устанавливались

##### Для примера уберем поддержку ipv6 командой
    sed -i '/--with-ipv6/d' ~/rpmbuild/SPECS/nginx.spec
##### Создаем rpm
    rpmbuild -bb ~/rpmbuild/SPECS/nginx.spec

rpm пакет собран.


#### Установка репозитория
##### Установим необходимый софт
    sudo yum install -y -q createrepo
##### Создаем директорию для репозитория...
    REPODIR=/usr/share/nginx/html/repos/x86_64 # в дальнейшем будет директория через nginx
    sudo mkdir -p $REPODIR
##### Копируем готовые rpm и создаем сам репозиторий
    sudo cp -r ~/rpmbuild/RPMS/x86_64/* $REPODIR
    sudo createrepo $REPODIR
##### Создаем файл repo для нашего репозитория (пока локальный)
    cat <<'EOF' | sudo tee /etc/yum.repos.d/my.repo
     [myrepo-x86_64]
     name=my repo
     baseurl=/usr/share/nginx/html/repos/x86_64
     enabled=1
     gpgcheck=0
    EOF
##### Пробуем установить nginx из локального репозитория
    sudo yum --disablerepo "nginx,nginx-source,AppStream" install -y -q nginx # отключим на время другие репозитории с nginx
##### Проверим из какого репозитория установился nginx
    yum info nginx
###### Вывод команды:
    [vagrant@lesson10 ~]$ yum info nginx
    Last metadata expiration check: 0:09:16 ago on Thu 11 Jun 2020 06:17:05 PM UTC.
    Installed Packages
    Name         : nginx
    Epoch        : 1
    Version      : 1.19.0
    Release      : 1.el8.ngx
    Architecture : x86_64
    Size         : 3.7 M
    Source       : nginx-1.19.0-1.el8.ngx.src.rpm
    Repository   : @System
    From repo    : myrepo-x86_64
    Summary      : High performance web server
    URL          : http://nginx.org/
    License      : 2-clause BSD-like license
    Description  : nginx [engine x] is an HTTP and reverse proxy server, as well as
                 : a mail proxy server.

##### Запустим и проверим nginx (привет домашке по SystemD)
    sudo systemctl enable --now nginx.service
    systemctl status nginx.service
##### Поменяем файл репозитория с локального на сетевой
    sudo sed -i 's%/usr/share/nginx/html/repos/x86_64%http://192.168.11.110/repos/x86_64/%' /etc/yum.repos.d/my.repo
##### Проверим репозиторий командой
    yum repoinfo "my repo"
###### Вывод команды:
    [vagrant@lesson10 ~]$ yum repoinfo "my repo"
    Last metadata expiration check: 0:12:42 ago on Thu 11 Jun 2020 06:17:05 PM UTC.

    Repo-id      : myrepo-x86_64
    Repo-name    : my repo
    Repo-status  : enabled
    Repo-revision: 1591899397
    Repo-updated : Thu 11 Jun 2020 06:16:37 PM UTC
    Repo-pkgs    : 0
    Repo-size    : 0
    Repo-baseurl : http://192.168.11.110/repos/x86_64/
    Repo-expire  : 172,800 second(s) (last: Thu 11 Jun 2020 06:17:05 PM UTC)
    Repo-filename: /etc/yum.repos.d/my.repo


Задание выполнено
## Спасибо за проверку!
