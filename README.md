# # Домашнее задание к занятию "Система мониторинга Zabbix" Муртазин Ильяс fops-42

### Задание 1 

Установите Zabbix Server с веб-интерфейсом.

#### Процесс выполнения
1. Выполняя ДЗ, сверяйтесь с процессом отражённым в записи лекции.
2. Установите PostgreSQL. Для установки достаточна та версия, что есть в системном репозитороии Debian 11.
3. Пользуясь конфигуратором команд с официального сайта, составьте набор команд для установки последней версии Zabbix с поддержкой PostgreSQL и Apache.
4. Выполните все необходимые команды для установки Zabbix Server и Zabbix Web Server.

#### Требования к результатам 
1. Прикрепите в файл README.md скриншот авторизации в админке.
2. Приложите в файл README.md текст использованных команд в GitHub.

---

### Решение 1

*Последовательность комманд для установки Zabbix Server:*

```bash
apt install postgresql apache2 -y
wget https://repo.zabbix.com/zabbix/7.4/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.4+ubuntu22.04_all.deb
dpkg -i zabbix-release_latest_7.4+ubuntu22.04_all.deb
apt update
apt install zabbix-server-pgsql zabbix-frontend-php php8.1-pgsql zabbix-apache-conf zabbix-sql-scripts -y
su - postgres -c 'psql --command "CREATE USER zabbix WITH PASSWORD '\'123456789\'';"'
su - postgres -c 'psql --command "CREATE DATABASE zabbix OWNER zabbix"'
zcat /usr/share/zabbix/sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
sed -i 's/# DBPassword=/DBPassword=123456789/g' /etc/zabbix/zabbix_server.conf
systemctl restart zabbix-server apache2
systemctl enable zabbix-server apache2
```

Результат:
![Вход в админку](https://github.com/murtazinilyas/monitoring/blob/main/scshots/zab_t1_admin.png)

---

### Задание 2 

Установите Zabbix Agent на два хоста.

#### Процесс выполнения
1. Выполняя ДЗ, сверяйтесь с процессом отражённым в записи лекции.
2. Установите Zabbix Agent на 2 вирт.машины, одной из них может быть ваш Zabbix Server.
3. Добавьте Zabbix Server в список разрешенных серверов ваших Zabbix Agentов.
4. Добавьте Zabbix Agentов в раздел Configuration > Hosts вашего Zabbix Servera.
5. Проверьте, что в разделе Latest Data начали появляться данные с добавленных агентов.

#### Требования к результатам
1. Приложите в файл README.md скриншот раздела Configuration > Hosts, где видно, что агенты подключены к серверу
2. Приложите в файл README.md скриншот лога zabbix agent, где видно, что он работает с сервером
3. Приложите в файл README.md скриншот раздела Monitoring > Latest data для обоих хостов, где видны поступающие от агентов данные.
4. Приложите в файл README.md текст использованных команд в GitHub

---

### Решение 2

Установка zabbix agent на машину с установленным zabbix server:

```bash
apt install zabbix-agent -y
systemctl restart zabbix-agent
systemctl enable zabbix-agent
```

Для установки zabbix agent написал такой ansible-плейбук:

```YAML
---
- hosts: "my"
  become: true

  tasks:
    - name: "Initialize repo"
      shell: |
        wget https://repo.zabbix.com/zabbix/7.4/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.4+ubuntu22.04_all.deb &&
        dpkg -i zabbix-release_latest_7.4+ubuntu22.04_all.deb

    - name: "Install zabbix agent"
      apt:
        name: zabbix-agent
        state: present
        update_cache: yes

    - name: "Change zabbix server address"
      lineinfile:
        path: /etc/zabbix/zabbix_agentd.conf
        regexp: '^Server=127.0.0.1'
        line: Server=192.168.56.21
      notify:
        - Restart zabbix agent

    - name: "Enable zabbix agent on boot"
      service:
        name: zabbix-agent
        enabled: yes

  handlers:
    - name: Restart zabbix agent
      service:
        name: zabbix-agent
        state: restarted
```

![Список агентов](https://github.com/murtazinilyas/monitoring/blob/main/scshots/zab_t2_conf.png)

![Лог файл одного из хостов](https://github.com/murtazinilyas/monitoring/blob/main/scshots/zab_t2_log.png)

![Метрики хостов](https://github.com/murtazinilyas/monitoring/blob/main/scshots/zab_t2_metrics.png)

---

## Задание 3 со звёздочкой*
Установите Zabbix Agent на Windows (компьютер) и подключите его к серверу Zabbix.

#### Требования к результатам
1. Приложите в файл README.md скриншот раздела Latest Data, где видно свободное место на диске C:

--- 

### Решение 3

![Метрики хоста на Win11](https://github.com/murtazinilyas/monitoring/blob/main/scshots/zab_t3.png)
