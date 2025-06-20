## Пример использования ключей

### 1. Плейбук для проведения аудита 
```yaml
#  ANSIBLE_STDOUT_CALLBACK=yaml необходимо писать данную команду перед запуском плейбука
- name: Аудит дисковой подсистемы 
  hosts: all 
  gather_facts: false
  become: true

  tasks:
    - name: Собрать информацию о дисках системными командами
      ansible.builtin.shell: |
        echo "==============================================================="
        echo "ОТЧЕТ ПО ХОСТУ: $(hostname)"
        echo "==============================================================="
        echo ""
        echo "ПАМЯТЬ (RAM):"
        free -h
        echo ""
        echo "FSTAB (конфигурация постоянного монтирования):"
        cat /etc/fstab
        echo ""
        echo "ОБЩАЯ СТРУКТУРА БЛОЧНЫХ УСТРОЙСТВ (lsblk):"
        lsblk
        echo ""
        echo "ДИСКИ, РАЗДЕЛЫ И ФАЙЛОВЫЕ СИСТЕМЫ (lsblk -f):"
        lsblk -f
        echo ""
        echo "LVM - PHYSICAL VOLUMES (pvs):"
        pvs
        echo ""
        echo "LVM - VOLUME GROUPS (vgs):"
        vgs
        echo ""
        echo "LVM - LOGICAL VOLUMES (lvs):"
        lvs
        echo ""
        echo "ИСПОЛЬЗОВАНИЕ ДИСКОВОГО ПРОСТРАНСТВА (df -h):"
        df -h
        echo ""
        echo "SELINUX КОНТЕКСТЫ КЛЮЧЕВЫХ ДИРЕКТОРИЙ (ls -Zd)"
        ls -Zd / /var /var/log /home /opt /usr /etc /boot /var/www /var/lib/mysql 2>/dev/null || true
        echo "==============================================================="
      args:
        executable: /bin/bash
      register: audit_report
      changed_when: false

    - name: Показать сводный отчет
      ansible.builtin.debug:
        msg: "{{ audit_report.stdout }}"
```
### 2. Пример структуры:
```yaml
disks:
  - operation_type: create        #  верхнеуровневый ключ
    device: /dev/sdb              #  верхнеуровневый ключ
    vg_name: sdb_vg               #  верхнеуровневый ключ
    lvs_to_create:                #  верхнеуровневый ключ
      - name: data_lv             #  вложенный ключ
        size: 100%FREE            #  вложенный ключ
        mount_point: /data        #  вложенный ключ
        fs_type: xfs              #  вложенный ключ
        
# Верхнеуровневые ключи описывают, что нужно сделать с диском/томами — создать, расширить, удалить.
# Вложенные ключи относятся к каждому отдельному логическому тому (LV), и управляют его параметрами: имя, размер, файловая система, точка монтирования и т.д.
 ```

### 3. Использование в плейбуке
```yaml
- name: manage_disks playbook
  hosts: all
  become: true
  
  vars:
    reboot_if_changed: false
    disk_configs_by_host:
      '192.168.':
        - operation_type: stretch
          disk_device: /dev/sda
          partition_to_stretch: /dev/sda3
          vg_to_extend: rhel
          lvs_to_modify:
            - name: root
              new_size: 50G
              mount_point: /
              allow_dangerous_resize: true
            - name: log_lv
              new_size: "100%FREE"
              mount_point: /var/log
              fs_type: xfs
              selinux_context: var_log_t
        - operation_type: create
          device: /dev/sdb
          vg_name: sdb_vg
          lvs_to_create:
            - name: data_lv
              size: "100%FREE"
              mount_point: /data
              fs_type: xfs
              selinux_context: default_t

              
    disks: "{{ disk_configs_by_host[inventory_hostname] | default([]) }}"


  roles:
    - role: infra/fix-repository
      fix_repository_exit_on_success: false
    - role: infra/manage-disks
```

### 4. Создание нового тома на новом устройстве
```yaml
disks:
  - operation_type: create
    device: /dev/sdb
    vg_name: sdb_vg
    lvs_to_create:
      - name: data_lv
        size: "100%FREE"
        mount_point: /data
        fs_type: xfs
        selinux_context: default_t
```
### 5. Расширение существующих разделов и томов
```yaml
disks:
  - operation_type: stretch
    disk_device: /dev/sda
    partition_to_stretch: /dev/sda3
    vg_to_extend: rhel
    lvs_to_modify:
      - name: root
        new_size: 50G
        mount_point: /
        allow_dangerous_resize: true

      - name: log_lv
        new_size: "100%FREE"
        mount_point: /var/log
        fs_type: xfs
        selinux_context: var_log_t
```

### 6. Увеличение размера одного логического тома
```yaml
disks:
  - operation_type: extend
    vg_to_extend: rhel
    lvs_to_modify:
      - name: tmp_lv
        new_size: 5G
        mount_point: /tmp
```

### 7. Удаление логического тома
```yaml
disks:
  - operation_type: extend
    vg_to_extend: rhel
    lvs_to_modify:
      - name: old_lv
        state: absent
```

