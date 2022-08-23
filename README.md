# Ansible
https://docs.ansible.com

## 1. Установка Ansible
Добавление архива Ansible
```bash
sudo apt-add-repository ppa:ansible/ansible
```

Установка Ansible из добавленного репозитария
```bash
sudo apt install ansible -y
```

## 2. Генерация ssh ключей
Созадем каталог, назначаем владельца и разрешения, генерируем ключ
```bash
mkdir ~/.ssh
chown -R user:user ~/.ssh/
chmod 700 ~/.ssh/
ssh-keygen
```

Проброс root ssh ключей на узлы
```bash
ansible all -m authorized_key -a "user=root key='{{ lookup('file', '/root/.ssh/id_rsa.pub') }}' path=/root/.ssh/authorized_keys manage_dir=no" --ask-pass -i ./inventory/inventory_hosts
```
или
```bash
cd .ssh
ssh-copy-id -i id_rsa.pub user@host
```
или
```bash
ssh-copy-id '-p 2202 -i ~/.ssh/id_dsa  user@host'
```
или
```bash
cat ~/.ssh/id_rsa.pub | ssh user@host 'cat >> ~/.ssh/authorized_keys'
```
или
```bash
 cat ~/.ssh/id_rsa.pub | ssh {user_name}@{hostname_or_ip} 'mkdir -p .ssh;touch .ssh/authorized_keys; cat >> .ssh/authorized_keys;chmod 700 ~/.ssh;chmod 600 ~/.ssh/authorized_keys'
```


## 3. Команды Ansible
### Команды проверки
Проверка версии Ansible
```bash
ansible --version
```
Проверяем доступные управляемые хосты (по умолчанию они указываются /etc/ansible/hosts)
```bash
ansible all --list-hosts
```
Если хотим использовать хосты из другого файла, то указываем параметр -i
```bash
ansible all --list-hosts -i ./inventory/inventory_hosts
```

### Команды разовые
Проверка связи с удаленными хостами по ключу
```bash
ansible -m ping web_servers_group -i ./inventory/inventory_hosts
```
или по паролю
```bash
ansible -m ping web_servers_group -i ./inventory/inventory_hosts --ask-pass
```
Может потребовать установить тулзу
```bash
sudo apt install sshpass
```
Чтобы каждый раз не вводить пароль можно указать учетные данные в файле ./inventory/inventory_hosts
```bash
192.168.0.1 ansible_user=ubuntu ansible_password=ubuntu
```
Повторяем команду, но уже без необходимости ввода пароля
```bash
ansible all -i ./inventory/inventory_hosts -m ping
```
Поверка времени, аптайма, загрузки хостов
```bash
ansible -a uptime web_servers_group -i ./inventory/inventory_hosts
```
Выполнение произвольной команды в shell всех хостах
```bash
ansible -m shell -a 'cat ~/.ssh/authorized_keys | grep rsa'  web_servers_group -i ./inventory/inventory_hosts
```

### Команды для работы с плейбуками 
Проверка синтаксиса плейбука перед запуском
```bash
ansible-playbook ./playbooks/install_web_server.yml --syntax-check
```
Запуск плейбука на хостах указанных в файле инвентаризации
```bash
ansible-playbook ./playbooks/install_web_server.yml -i ./inventory/inventory_hosts
```
Запуск плейбука с определенного шага на хостах указанных в файле инвентаризации
```bash
ansible-playbook ./playbooks/install_web_server.yml -i ./inventory/inventory_hosts --step --start-at-task="start nginx"
```




