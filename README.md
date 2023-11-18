Создание и использование роли

Предыдущая виртуалка была уничтожена и создана новая - так будет проще понять всё ли я делаю правильно.
Работа сводится к разделению моего предыдущего playbook.

1. mkdir roles

2. cd roles/

3. ansible-galaxy init install_configur_nginx

4. cd ..

5. Создаю файл шаблона nginx.conf.j2 в директории roles/install_configur_nginx/files

# {{ ansible_managed }}
events {
    worker_connections 1024;
}

http {
    server {
        listen       {{ nginx_listen_port }} default_server;
        server_name  default_server;
        root         /usr/share/nginx/html;

        location / {
        }
    }
}

6. В файле roles/install_configur_nginx/defaults/main.yml указываю vars
nginx_listen_port: 8080

7. В таком же файле main.yml, но в директории roles/install_configur_nginx/handlers, заполняю

- name: restart nginx
  systemd:
    name: nginx
    state: restarted
    enabled: yes

- name: reload nginx
  systemd:
    name: nginx
    state: reloaded

8. В файле main.yml в директории roles/install_configur_nginx/tasks заполняю

    - name: Install EPEL Repo package from standart repo
      yum:
        name: epel-release
        state: present
      tags:
        - epel-package
        - packages

    - name: Install NGINX
      yum:
        name: nginx
        state: latest
      notify:
       - restart nginx
      tags:
        - nginx-package
        - packages

    - name: Create NGINX config-file from template
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify:
        - reload nginx
      tags:
        - nginx-configuration

9. Файл playbook.yml будет выглядеть так

---
- name: NGINX
  hosts: nginx
  become: true

roles:
  - install_configur_nginx

10. Всё готово. Запускаю ansible-playbook nginx.yml

PLAY [NGINX] *************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************
ok: [nginx]

TASK [install_configur_nginx : Install EPEL Repo package from standart repo] *********************************************************
ok: [nginx]

TASK [install_configur_nginx : Install NGINX] ****************************************************************************************
ok: [nginx]

TASK [install_configur_nginx : Create NGINX config-file from template] ***************************************************************
changed: [nginx]

RUNNING HANDLER [install_configur_nginx : reload nginx] ******************************************************************************
changed: [nginx]

PLAY RECAP ***************************************************************************************************************************
nginx                      : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  


11. Проверка 

curl http://192.168.56.150:8080

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
<head>
  <title>Welcome to CentOS</title>
  <style rel="stylesheet" type="text/css"> 