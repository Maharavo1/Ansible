# Ansible
 name: Create Root directory
  file:
    path: "/var/www/{{ domain_name.split('.')[1] }}/{{ domain_name.split('.')[0] }}"
    state: directory
    owner: www-data
    mode: '0755'

- name: Create virtual host configuration file
  template:
    src: virtualhost_template.j2
    dest: "/etc/apache2/sites-available/{{ domain_name }}.conf"

- name: Create symbolic link for virtual host configuration file
  command: "ln -s /etc/apache2/sites-available/{{ domain_name }}.conf /etc/apache2/sites-enabled/{{ domain_name }}.conf"

- name: Enable virtual host
  apache2_site:
    name: "{{ domain_name }}"
    state: enabled
  notify:
    - restart apache2
#untested


if [ $# -eq 0 ]; then
  echo "Usage: \\$0 <file_path>"
  exit 1
fi

file_path="\$1"
ansible-playbook virtualhost_setup.yml -e "domain_names_file_path=$file_path

name: Setup Virtual Hosts
  hosts: all
  become: true
  vars:
    domain_names_file: "{{ domain_names_file_path }}"
  tasks:
    - name: Install Apache2
      apt:
        name: "apache2"
        update_cache: yes
        state: latest
    - name: Enable mod_rewrite
      apache2_module:
        name: rewrite
        state: present
      notify:
        - restart apache2
    - name: Read domain names from file
      set_fact:
        domain_names: "{{ lookup('lines', domain_names_file) }}"
    - name: Create virtual hosts
      include_tasks: create_virtualhost.yml
      loop: "{{ domain_names }}"
      loop_control:
        loop_var: domain_name
  handlers:
    - name: restart apache2
      service:
        name: apache2
        state: restarted


<VirtualHost *:80>
  ServerName {{ domain_name }}
  ServerAlias {{ domain_name.split('.')[0] }}.{{ domain_name.split('.')[1] }}
  DocumentRoot "/var/www/{{ domain_name.split('.')[1] }}/{{ domain_name.split('.')[0] }}"
</VirtualHost>
