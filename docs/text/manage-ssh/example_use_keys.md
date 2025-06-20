## Пример использования ключей

### 1. Удаление SSH-ключей пользователя
```yaml
users:
  - name: node_exporter
    state: absent
    generate_ssh_key: true
    ssh_key_type: ed25519
    ssh_key_bits: 256
    ssh_key_force: true
```

### 2. Генерация ключей и установка на хост
```yaml
users:
  - name: ansible
    generate_ssh_key: true
    ssh_key_type: ed25519
    ssh_key_comment: ansible@mgmt
    ssh_key_force: false
    state: present
```

### 3. Установка уже существующих ключей (без генерации)
```yaml
users:
  - name: deploy
    generate_ssh_key: false
    ssh_state: present
    state: present
```

### 4. Отправка ключей по email
```yaml
users:
  - name: devops
    generate_ssh_key: true
    ssh_key_type: rsa
    ssh_key_bits: 2048
    ssh_key_comment: devops@corp
    send_keys_to_email: true
    email: devops@example.com
    state: present
```

### 5. Комбинирование с другими ролями (например, установка Node Exporter)
```yaml
- name: Установить node_exporter и настроить SSH
  hosts: monitoring
  become: true
  roles:
    - role: monitoring/node-exporter
    - role: infra/manage-ssh
  vars:
    users:
      - name: node_exporter
        generate_ssh_key: true
        ssh_key_type: ed25519
        ssh_key_comment: exporter@telemetry
        ssh_key_force: false
        state: present
```

### 6. Массовая генерация ключей для нескольких пользователей
```yaml
users:
  - name: gacci
    generate_ssh_key: true
    ssh_key_comment: gacci@lab
    state: present

  - name: mucci
    generate_ssh_key: true
    ssh_key_comment: mucci@lab
    ssh_key_force: true
    ssh_key_type: ecdsa
    state: present
```