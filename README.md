# Домашнее задание к занятию "`Ansible.Часть 2`" - `Дудин Сергей Васильевич`

---

### Задание 1

**Выполните действия, приложите файлы с плейбуками и вывод выполнения.**

Напишите три плейбука. При написании рекомендуем использовать текстовый редактор с подсветкой синтаксиса YAML.

Плейбуки должны: 

1. Скачать какой-либо архив, создать папку для распаковки и распаковать скаченный архив. Например, можете использовать [официальный сайт](https://kafka.apache.org/downloads) и зеркало Apache Kafka. При этом можно скачать как исходный код, так и бинарные файлы, запакованные в архив — в нашем задании не принципиально.
2. Установить пакет tuned из стандартного репозитория вашей ОС. Запустить его, как демон — конфигурационный файл systemd появится автоматически при установке. Добавить tuned в автозагрузку.
3. Изменить приветствие системы (motd) при входе на любое другое. Пожалуйста, в этом задании используйте переменную для задания приветствия. Переменную можно задавать любым удобным способом.

Решение 1.1
```
---
- name: Download and unpacking Apache Kafka
  hosts: my
  become: yes

  tasks:
    - name: create directory
      file:
        path: "/netology/kafka"
        state: directory
        mode: '0754'

    - name: download Kafka
      get_url:
        url: "https://dlcdn.apache.org/kafka/4.0.0/kafka-4.0.0-src.tgz"
        dest: "/tmp/kafka.tgz"
        timeout: 300

    - name: unpacking arhive
      unarchive:
        src: "/tmp/kafka.tgz"
        dest: "/netology/kafka"
        remote_src: yes
```

![Решение 1.1](https://github.com/noisy441/7-01.part_2/blob/main/img/1.1.png)


Решение 1.2
```
---
- name: Install package
  hosts: my
  become: yes
  vars:
    package_name: "tuned"
  
  tasks:
    - name: Installing package
      apt:
        state: present
        name: "{{ package_name }}"
    
    - name: Launch and add to startup
      service:
        name: "{{ package_name }}"
        state: started
        enabled: yes

```

![Решение 1.2](https://github.com/noisy441/7-01.part_2/blob/main/img/1.2.png)

Решение 1.3
```
---
- name: New MOTD using variable
  hosts: all
  become: yes
  
  vars:
    motd_message: |
      Добро пожаловать в систему!
      Дата: {{ ansible_date_time.date }}
      Время: {{ ansible_date_time.time }}
      
  tasks:
    - name: Create a temporary file with a message
      copy:
        content: "{{ motd_message }}"
        dest: /tmp/new_motd.txt
        
    - name: Installing a new MOTD
      command: cp /tmp/new_motd.txt /etc/motd
      
    - name: Deleting a temporary file
      file:
        path: /tmp/new_motd.txt
        state: absent
```

![Решение 1.3](https://github.com/noisy441/7-01.part_2/blob/main/img/1.3.1.png)
![Решение 1.3](https://github.com/noisy441/7-01.part_2/blob/main/img/1.3.2.png)




---

### Задание 2

**Выполните действия, приложите файлы с модифицированным плейбуком и вывод выполнения.** 

Модифицируйте плейбук из пункта 3, задания 1. В качестве приветствия он должен установить IP-адрес и hostname управляемого хоста, пожелание хорошего дня системному администратору. 

Решение 2

```
---
- name: New MOTD using variable
  hosts: all
  become: yes
  
  #В качестве приветствия устанавливаем IP-адрес и hostname управляемого хоста, пожелание хорошего дня системному администратору. 

  vars:
    motd_message: |
      Доброго дня системный администратор. Ваш IP: {{ ansible_default_ipv4.address }} hostname: {{ ansible_hostname }}
      
  tasks:
    - name: Create a temporary file with a message
      copy:
        content: "{{ motd_message }}"
        dest: /tmp/new_motd.txt
        
    - name: Installing a new MOTD
      command: cp /tmp/new_motd.txt /etc/motd
      
    - name: Deleting a temporary file
      file:
        path: /tmp/new_motd.txt
        state: absent
```

![Решение 2](https://github.com/noisy441/7-01.part_2/blob/main/img/2.1.png)
![Решение 2](https://github.com/noisy441/7-01.part_2/blob/main/img/2.2.png)


---

### Задание 3

**Выполните действия, приложите архив с ролью и вывод выполнения.**

Ознакомьтесь со статьёй [«Ansible - это вам не bash»](https://habr.com/ru/post/494738/), сделайте соответствующие выводы и не используйте модули **shell** или **command** при выполнении задания.

Создайте плейбук, который будет включать в себя одну, созданную вами роль. Роль должна:

1. Установить веб-сервер Apache на управляемые хосты.
2. Сконфигурировать файл index.html c выводом характеристик каждого компьютера как веб-страницу по умолчанию для Apache. Необходимо включить CPU, RAM, величину первого HDD, IP-адрес.
Используйте [Ansible facts](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html) и [jinja2-template](https://linuxways.net/centos/how-to-use-the-jinja2-template-in-ansible/). Необходимо реализовать handler: перезапуск Apache только в случае изменения файла конфигурации Apache.
4. Открыть порт 80, если необходимо, запустить сервер и добавить его в автозагрузку.
5. Сделать проверку доступности веб-сайта (ответ 200, модуль uri).

В качестве решения:
- предоставьте плейбук, использующий роль;
- разместите архив созданной роли у себя на Google диске и приложите ссылку на роль в своём решении;
- предоставьте скриншоты выполнения плейбука;
- предоставьте скриншот браузера, отображающего сконфигурированный index.html в качестве сайта.

`Решение`

1. `playbook-apache.yml`
```
---
- name: Install Apache
  hosts: my
  become: yes
  gather_facts: yes

  roles:
    - apache_setup
```
2. `roles/tasks/main.yml`
```
---
- name: Gather facts
  setup:
    filter: "ansible_devices,ansible_default_ipv4"

- name: Install Apache
  package:
    name: apache2
    state: present

- name: Check Apache is running and enabled
  service:
    name: apache2
    state: started
    enabled: yes

- name: Open tcp port 80
  ufw:
    rule: allow
    port: '80'
    proto: tcp

- name: Find first disk
  set_fact:
    first_disk: >-
      {{
        ansible_devices
        | dict2items
        | selectattr('key', 'match', '^(sd|vd|nvme)[a-z]+$')
        | list
        | map(attribute='key')
        | first | default('not_found')
      }}

- name: 
  template:
    src: index.html.j2
    dest: '/var/www/html/index.html'
  notify:
    - restart apache2

- name: Check website availability
  uri:
    url: http://{{ inventory_hostname }}
    status_code: 200
  delegate_to: localhost
  run_once: yes
```
3. `Все остальные файлы проекта представлены в репозитории`


![Решение 3](https://github.com/noisy441/7-01.part_2/blob/main/img/3.png)
![Решение 3](https://github.com/noisy441/7-01.part_2/blob/main/img/3.1.png)

