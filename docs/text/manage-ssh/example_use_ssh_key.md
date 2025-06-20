## Примеры добавления ключей

### 1. Тип ключей: ED25519 (по умолчанию)
```yaml
users:
  - name: ansible
    generate_ssh_key: true
    ssh_key_type: ed25519
    ssh_key_bits: 256  
    state: present
```
### 2. Тип ключей: RSA
```yaml
users:
  - name: ci-user
    generate_ssh_key: true
    ssh_key_type: rsa
    ssh_key_bits: 2048
    state: present

users:
  - name: ci-user
    generate_ssh_key: true
    ssh_key_type: rsa
    ssh_key_bits: 3072
    state: present

users:
  - name: ci-user
    generate_ssh_key: true
    ssh_key_type: rsa
    ssh_key_bits: 4096
    state: present
```

### 3. Тип ключей: ECDSA
```yaml
users:
  - name: сi_user
    generate_ssh_key: true
    ssh_key_type: ecdsa
    ssh_key_bits: 256
    state: present

users:
  - name: сi_user
    generate_ssh_key: true
    ssh_key_type: ecdsa
    ssh_key_bits: 384
    state: present

users:
  - name: сi_user
    generate_ssh_key: true
    ssh_key_type: ecdsa
    ssh_key_bits: 521
    state: present
