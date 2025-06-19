## Пример использования ключей 

### Полный набор ключей
```yaml
vm_list:
  - name: demo-vm-full
    id: 150
    state: present              # можно: present, started, stopped, absent, snapshot, backup, rollback, migrate
    ip: 192.168.50.150
    cidr: 24
    gw: 192.168.50.1
    nameserver: 8.8.4.4
    searchdomain: corp.local
    ci_user: ansible
    memory: 4096                # в MB
    cores: 4
    onboot: 1                   # или true

    target_node: pve2          # нода куда будет клонироваться
    source_node: pve1          # используется при миграции
    source_vmid: 9000          # шаблон VM 

    bridge: br-vlan2875
    nic_model: virtio

    disk_add:
      storage: local-lvm
      size: 20G
      disk_type: scsi

    disk_resize: 10            # увеличить диск на 10 (в гигабайтах)
    disk_name: scsi0           # имя существующего диска, который будем увеличивать

    snapshot_name: before-patch
    backup_storage: backup-nfs
```

### Создание новой ВМ
```yaml
vm_list:
  - name: debian-test-01
    id: 110
    state: present
    ip: 192.168.10.110
    cidr: 24
    gw: 192.168.10.1
    ci_user: ansible
    memory: 2048
    cores: 2
    onboot: true
    bridge: br-vlan2875
    nic_model: virtio
    disk_add:
      storage: local-lvm
      size: 10G
```

### Удаление ВМ
```yaml
vm_list:
  - name: debian-old-01
    id: 111
    state: absent
    target_node: pve1
```

### Расширение диска и запуск
```yaml
vm_list:
  - name: debian-vm-prod
    id: 120
    state: started
    disk_resize: 20
    disk_name: scsi0
    target_node: pve2
```

### Создание снапшота 
```yaml
vm_list:
  - name: backend-db
    id: 130
    state: snapshot
    snapshot_name: pre-upgrade
    target_node: pve2
```

### Миграция VM между нодами
```yaml
vm_list:
  - name: win-tools
    id: 140
    state: migrate
    source_node: pve1
    target_node: pve2
```