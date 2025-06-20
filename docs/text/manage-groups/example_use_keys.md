## Пример использования ключей

### 1. Базовое создание группы
```yaml
group_list:
  - name: devs
    gid: 2001
    state: present
# Создаёт группу devs с GID 2001.
```

### 2. Удаление группы
```yaml
group_list:
  - name: oldgroup
    state: absent
# Удалит группу oldgroup, даже если у неё есть участники (предупредит через debug:).
```

### 3. Автоматическое назначение свободного GID
```yaml
group_list:
  - name: ci
    state: present
# Если gid не задан — будет найден первый свободный ≥ 1000.
```

### 4. Использование системной группы (`gid > 1000`) с разрешением
```yaml
group_list:
  - name: system-services
    gid: 999
    system: true
    group_allow_system_gid: true
    state: present
# Без group_allow_system_gid роль выдаст ошибку. Причина: защита от создания системных групп по ошибке. GID < 1000 — зарезервирован для системных групп: daemon, bin, mail, systemd-journal, и т.д.
```

### 5. Использование `group_gid_map`
```yaml
group_gid_map:
  metrics: 2500
  observability: 2501

group_list:
  - name: metrics
    state: present
  - name: observability
    state: present
```

### 6. Совмещение с ролью `infra/manage-users`
```yaml
- name: Управление группами и пользователями
  hosts: all
  become: true
  roles:
    - role: infra/manage-groups
    - role: infra/manage-users
  vars:
    group_list:
      - name: developers
        gid: 1500
        state: present

    user_list:
      - name: gacci
        groups: ["developers"]
        state: present
# Сначала создаёт группу, затем добавляет пользователя gacci в неё.
```

### 7. Совмещение с ролью `infra/manage-ssh` 
```yaml
- name: Создание группы с пользователями и SSH
  hosts: all
  become: true
  roles:
    - role: infra/manage-groups
    - role: infra/manage-users
    - role: infra/manage-ssh
  vars:
    group_list:
      - name: deploy
        gid: 1600
        state: present

    user_list:
      - name: deployer
        groups: ["deploy"]
        state: present

    users:
      - name: deployer
        generate_ssh_key: true
        ssh_key_type: ed25519
        ssh_key_comment: deploy@access
        ssh_key_force: false
        state: present
```