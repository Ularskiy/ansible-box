## Пример использования переменных

### 1. Установка Node Exporter с созданием пользователя/группы используя 
```yaml
- name: Установка Node Exporter и зависимостей
  hosts: exporters
  become: true

  vars:
    node_exporter_state: "present"
    node_exporter_user: "node_user"
    node_exporter_group: "node_user"
    node_exporter_port: 9100

    node_exporter_groups:
      - name: "node_user"
        system: true
        state: present

    node_exporter_users:
      - name: "node_user"
        group: "node_user"
        system: true
        shell: /sbin/nologin
        create_home: false
        state: present

    group_list: "{{ node_exporter_groups }}"
    users: "{{ node_exporter_users }}"

  roles:
    - infra/manage-groups
    - infra/manage-users
    - monitoring/node-exporter
```


### 2. Только установка Node Exporter (например если юзер и группы уже созданы)
```yaml
- name: Установка Node Exporter без создания пользователя
  hosts: exporters
  become: true

  vars:
    node_exporter_state: "present"
    node_exporter_user: "node_user"
    node_exporter_group: "node_user"
    node_exporter_port: 9100

  roles:
    - monitoring/node-exporter
```

### 3. Принудительное обновление версии Node Exporter
```yaml
- name: Обновление Node Exporter
  hosts: exporters
  become: true

  vars:
    node_exporter_version: "1.9.1"
    update_node_exporter: true
    node_exporter_state: "present"
    node_exporter_user: "nodeusr"
    node_exporter_group: "nodeusr"

  roles:
    - monitoring/node-exporter
```


### 4. Удаление Node Exporter
```yaml
- name: Удалить Node Exporter
  hosts: exporters
  become: true

  vars:
    node_exporter_state: "absent"

  roles:
    - monitoring/node-exporter
```