# **Описание**

В данном домашнем задании нам необходимо написать простой playbook, который будет запускать 2 виртуальные машины, устанавливать на них nginx, а также копировать на них стартовые странички.

---

# **Подготовка и запуск окружения** 

Для тестового стенда нам потребуется Vagrant файл из методички, который мы немного поравим, добавив в него секции для создания второй виртуальной машины, а также заменим ip адреса на адресацию из дефолтной сети - 192.168.56.0/24. Соотвественно - 192.168.56.150 и 192.168.56.151.

Описывать весь процесс создания файлов для запуска playbook на этих машинах не буду, так как он описан в методичке. Соотвественно для его выполнения, нам необходимо следующее:

1. Cклонировать себе на компьютер данный репозиторий:
```
git clone https://github.com/Dmitriy-Iv/Otus-Pr-linux-HW10.git hw10-git
```

2. Запустить Vagrantfile.
```
cd hw10-git/Ansible/
vagrant up
```

3. После этого проверить, что машины поднялись и их видит Ansible и имеет к ним доступ.
```
ansible-inventory --list
```

Вывод должен быть следующий:
```
{
    "_meta": {
        "hostvars": {
            "nginx": {
                "ansible_host": "192.168.56.150",
                "ansible_port": 22,
                "ansible_private_key_file": ".vagrant/machines/nginx/virtualbox/private_key",
                "nginx_index_file": "index_nginx.html",
                "nginx_listen_port": 80
            },
            "nginx2": {
                "ansible_host": "192.168.56.151",
                "ansible_port": 22,
                "ansible_private_key_file": ".vagrant/machines/nginx2/virtualbox/private_key",
                "nginx_index_file": "index_nginx2.html",
                "nginx_listen_port": 8080
            }
        }
    },
    "all": {
        "children": [
            "ungrouped",
            "web"
        ]
    },
    "web": {
        "hosts": [
            "nginx",
            "nginx2"
        ]
    }
}
```

4. Запустить playbook. 
```
ansible-playbook playbooks/nginx.yml
```
Вывод должен быть примерно следующий (в зависимости от того, первый раз запускалася playbook или уже нет, как в моём случае):
```

PLAY [NGINX | Install and configure NGINX] **************************************************************************************************************************************************************************************************

TASK [nginx : NGINX | Install EPEL Repo package from standart repo] *************************************************************************************************************************************************************************
ok: [nginx]
ok: [nginx2]

TASK [nginx : NGINX | Install NGINX package from EPEL Repo] *********************************************************************************************************************************************************************************
ok: [nginx2]
ok: [nginx]

TASK [nginx : NGINX | Create NGINX config file from template] *******************************************************************************************************************************************************************************
ok: [nginx2]
ok: [nginx]

TASK [nginx : NGINX | Copy index.html on target machines] ***********************************************************************************************************************************************************************************
ok: [nginx]
ok: [nginx2]

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
nginx                      : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
nginx2                     : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

5. Проверить доступность nginx.
```
curl http://192.168.56.150:80
curl http://192.168.56.151:8080
```

---

# **Нюансы при работе с хост машиной Windows** 

В моём случае - c Vagrant, VirtualBox и GitHub Desktop установленными на Windows 10, а Ansible установлен в WSL Ubuntu, я действую по следующему принципу:

1. Клонирую себе на компьютер данный репозиторий через GitHub Desktop - File - Clone Repository - URL (https://github.com/Dmitriy-Iv/Otus-Pr-linux-HW10.git).

2. Запускаю powershell, перехожу в данный каталог, в папку Ansible и там запускаю Vagrantfile.
```
PS D:\TEMP\OTUS\my-hw10\Ansible> vagrant up
```

3. После того, как машины создались и запустились (можно проверить выполнив vagrant ssh nginx и vagrant ssh nginx2), копирую папку Ansible из директории Windows в WSL (можно и не копировать, но тогда необходимо прописывать пути до ключей ssh через абсолютный путь на моём компьютере, плюс начинаются вопросы с правами, не только с ключами, но и ansible.cfg и т.д.).
```
dmitriy@spb:~$ cp -r /mnt/d/TEMP/OTUS/my-hw10/Ansible ./my-hw10/
```

4. Меняю права на данный каталог Ansible и на private.key виртуальных машин.
```
dmitriy@spb:~$ chmod -R 755 my-hw10/Ansible/
dmitriy@spb:~/my-hw10/Ansible$ chmod 700 my-hw10/Ansible/.vagrant/machines/nginx/virtualbox/private_key
dmitriy@spb:~/my-hw10/Ansible$ chmod 700 my-hw10/Ansible/.vagrant/machines/nginx2/virtualbox/private_key
```

5. После этого проверяю, что машины поднялись и их видит Ansible и имеет к ним доступ, и запускаю playbook.
```
dmitriy@spb:~$ cd my-hw10/Ansible/
dmitriy@spb:~/my-hw10/Ansible$ ansible-inventory --list
dmitriy@spb:~/my-hw10/Ansible$ ansible-playbook playbooks/nginx.yml
```





