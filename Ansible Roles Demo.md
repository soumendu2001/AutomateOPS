🧩 STEP 1 — Keep Role Structure


ansible-galaxy init apache_webserver

🧩 STEP 2 — Tasks


touch tasks/main.yml.”

---
- name: Install Apache package

ansible.builtin.package:

- name: "{{ apache_package }}"
  state: present

- name: Deploy Apache index.html
  ansible.builtin.template:
    src: index.html.j2
    dest: "{{ apache_docroot }}/index.html"
  notify: Restart Apache

🧩 STEP 3 — Template 

touch roles/apache_webserver/templates/index.html.j2
vim roles/apache_webserver/templates/index.html.j2 with below content


<html>

<head>

     <title> AutomteOps {{ apache_site_name}} </title>

  </head>

  <body>

    <h1>Welcome to {{apache_site_name}} </h1>

  </body>

</html>



🧩 STEP 4 — Handler (Still generic ✅)


vim roles/apache_webserver/handlers/main.yml
 

---

# handlers file for apache_webserver

- name: Restart Apache

 ansible.builtin.service:

   name: "{{ apache_service}}"

   state: restarted



🧩 STEP 5 — Defaults (Common values only)

vim roles/apache_webserver/defaults/main.yml

---

apache_docroot: "/var/www/html"

apache_site_name: "AutomateOps Demo Site"



🧩 STEP 6 — OS‑Specific Variables ✅✅✅ (Key Change)


Create OS vars directory:


🔹 Ubuntu / Debian

Touch roles/apache_webserver/vars/Debian.yml


vim roles/apache_webserver/vars/Debian.yml

Update the file as below
apache_package: apache2

apache_service: apache2

---

🔹 RHEL / Amazon Linux
Update the file as below

touch roles/apache_webserver/vars/RedHat.yml

vim roles/apache_webserver/vars/RedHat.yml


---

apache_package: httpd

apache_service: httpd



🧩 STEP 7 — Load OS-Specific Vars Automatically 


Add this as FIRST task in tasks/main.yml

- name: Load OS-specific variables

 ansible.builtin.include_vars: "{{ ansible_os_family }}.yml"



✅ Final tasks/main.yml


---

# tasks file for apache_webserver

- name: Load OS Specific Variable

 ansible.builtin.include_vars: "{{ ansible_os_family}}.yml"


- name: Install Apache Package

 ansible.builtin.package:

   name: "{{ apache_package }}"

   state: present



- name: Deploy Apache Index file index.html

 ansible.builtin.template:

   src: index.html.j2

   dest: "{{ apache_docroot}}/index.html"

 notify: Restart Apache


...

🧩 STEP 8 — Playbook (Recommended Method ✅)

touch roles/ site.yml

---

- name: Configure Web Server (Ubuntu + RHEL)

 hosts: web

 become: yes

 roles:

   - apache_webserver



✅ STEP  9 — Run


ansible-playbook site.yml


✅ Verification (OS-Aware)


Ubuntu

Login into Ubuntu Server using ssh

systemctl status apache2

dpkg -l | grep apache2



“On RHEL or Amazon Linux…

Login into Amazon Linux  Server using ssh

Run

systemctl status httpd

rpm -qa | grep httpd



“And from any server…”



Common

curl http://localhost

grep -i "AutomateOps" /var/www/html/index.html


