# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
  :lesson10 => {
        :box_name => "centos/8",
        :ip_addr => '192.168.11.110',
  },
}

Vagrant.configure("2") do |config|

    MACHINES.each do |boxname, boxconfig|
        config.vbguest.no_install = true
        config.vm.define boxname do |box|

            box.vm.box = boxconfig[:box_name]
            box.vm.host_name = boxname.to_s

            #box.vm.network "forwarded_port", guest: 8080, host: 8080

            box.vm.network "private_network", ip: boxconfig[:ip_addr]

            box.vm.provider :virtualbox do |vb|
                    vb.customize ["modifyvm", :id, "--memory", "1024"]
            #        vb.gui = true
            end

        box.vm.provision "shell", privileged: false, inline: <<-SHELL
        echo -e "\n\nЗадание: собрать собственный rpm пакет и разместить его в собственном репозитории"
        echo -e "\nСборка кастомного nginx..."
        echo -e "Устанавливаем необходимый софт..."
        sudo yum install -y -q epel-release > /dev/null 2>&1

        sudo yum install -y -q git rpm-build rpmdevtools gcc make automake yum-utils > /dev/null 2>&1
        echo "Установим необходимые для компиляции nginx devel-файлы"
        sudo yum-builddep -y -q nginx
        echo "Скачиваем rpm с исходниками nginx..."
        echo "Добавим репозиторий nginx..."
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
        yumdownloader --source nginx
        rpmdev-setuptree
        rpm -ivh /home/vagrant/nginx-* > /dev/null 2>&1

        echo "Для примера уберем поддержку ipv6"
        sed -i '/--with-ipv6/d' ~/rpmbuild/SPECS/nginx.spec
        echo "Создаем rpm..."
        rpmbuild -bb ~/rpmbuild/SPECS/nginx.spec > /dev/null 2>&1
        echo "Готово"
        echo -e "\n\nУстановка собственного репозитория..."
        echo "Установим необходимый софт"
        sudo yum install -y -q createrepo
        echo "Создаем директорию для репозитория..."
        REPODIR=/usr/share/nginx/html/repos/x86_64
        sudo mkdir -p $REPODIR
        echo "Копируем готовые rpm и создаем репозиторий..."
        sudo cp -r ~/rpmbuild/RPMS/x86_64/* $REPODIR
        sudo createrepo $REPODIR
        echo -e "\nСоздаем файл repo..."
        cat <<'EOF' | sudo tee /etc/yum.repos.d/my.repo
[myrepo-x86_64]
name=my repo
baseurl=/usr/share/nginx/html/repos/x86_64
enabled=1
gpgcheck=0
EOF

        echo -e "\n\nУстановим nginx из локального репозитория..."
        sudo yum --disablerepo "nginx,nginx-source,AppStream" install -y -q nginx
        echo -e "\n\nПроверим из какого репозитория установился nginx..."
        yum info nginx |grep "From repo"
        echo -e "\n\nЗапустим и проверим nginx"
        sudo systemctl enable --now nginx.service
        systemctl status nginx.service
        echo "Поменяем файл репозитория с локального на сетевой..."
        sudo sed -i 's%/usr/share/nginx/html/repos/x86_64%http://192.168.11.110/repos/x86_64/%' /etc/yum.repos.d/my.repo
        echo "Проверим репозиторий"
        yum repoinfo "my repo"
        echo -e "Задание выполнено"
        echo "Спасибо за проверку!"
  SHELL

        end
    end
  end
