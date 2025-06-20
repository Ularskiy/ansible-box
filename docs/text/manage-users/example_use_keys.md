## Пример использования ключей

### 1. Создание пользователей с различными параметрами
```yaml
- name: Создание пользователей с разными shell и sudo
  hosts: test_hosts
  become: true

  vars:
    users:
      - name: user_with_password
        manage_existing: true
        comment: "user_with_password_new"
        shell: /bin/bash
        groups: ["operator", "games"]
        sudo: no_password

      - name: user_no_login
        manage_existing: true
        shell: /sbin/nologin
        sudo: no_password

      - name: user_limited
        manage_existing: true
        shell: /bin/bash
        sudo: limited
        allowed_commands:
          - /usr/bin/systemctl restart sshd
          - /usr/bin/systemctl restart cron.service
          - /usr/bin/journalctl -u sshd
          - /usr/bin/journalctl -u cron.service
          
  roles:                 
    - infra/manage-users
```

### 2. Удаление пользователей с защитой
```yaml
- name: Удаление пользователей с защитой
  hosts: test_hosts
  become: true

  vars:
    protected_users:
      - root
      - ansible
    sensitive_accounts:
      - carol

    users:
      - name: carol
        state: absent
      - name: alice
        state: absent

  roles:
    - infra/manage-users
```

### 3. Запуск плейбука в режиме `DRY-RUN` без реальных изменений
```yaml
- name: Эмуляция создания пользователей
  hosts: test_hosts
  become: true

  vars:
    users_dry_run: true
    users:
      - name: testuser
        shell: /bin/bash
        comment: "тупо тест"
        sudo: no_password

  roles:
    - infra/manage-users
```

### 4. Сброс пароля вручную
```yaml
- name: Принудительный сброс пароля
  hosts: test_hosts
  become: true

  vars:
    users:
      - name: bob
        reset_password: true

  roles:
    - infra/manage-users
```

### 5. Управление существующими пользователями
```yaml
- name: Обновление shell и комментария существующего пользователя
  hosts: all
  become: true

  vars:
    users:
      - name: test_user
        manage_existing: true
        shell: /bin/zsh
        comment: "Обновлённый test_user"
        groups: ["wheel"]

  roles:
    - infra/manage-users
```