## Пример использования ключей

### 1. Плейбук для проведения аудита (full)
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
### 1.1 Плейбук для проведения аудита (mini)
```yaml
#  ANSIBLE_STDOUT_CALLBACK=yaml необходимо писать данную перед запуском плейбука
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
        echo "ОБЩАЯ СТРУКТУРА БЛОЧНЫХ УСТРОЙСТВ (lsblk):"
        lsblk
        echo ""
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

### 8. Пояснение по переменным 
```yaml
# Документация по переменным роли infra/manage-disks

disks:
  - operation_type: create                # Тип операции: create | extend | stretch
    device: /dev/sdb                      # Устройство для создания раздела
    device_partition_number: 1            # Номер создаваемого раздела (по умолчанию 1)
    create_disklabel_if_missing: true     # Создать метку (GPT) если отсутствует
    disklabel_type: gpt                   # Тип метки диска (по умолчанию: gpt)
    add_to_existing_vg: false             # Добавить к существующей VG (если true)
    vg_name: data_vg                      # Имя новой VG (если create)
    vg_to_extend: data_vg                 # Имя существующей VG (если extend/stretch)

    partition_to_stretch: /dev/sda3       # Только для stretch — указание раздела

    lvs_to_create:                        # Только для create
      - name: data_lv
        size: 100%FREE                    # Размер тома
        fs_type: ext4                     # Тип файловой системы
        fs_label: data_lv_1               # Опционально: метка ФС
        mount_point: /data                # Куда монтировать
        preserve: true                    # Использовать safe-mount при наличии данных
        selinux_context: default_t        # SELinux-контекст
        state: present                    # Управление томом: present | absent

    lvs_to_modify:                        # Для extend или stretch
      - name: log_lv
        new_size: 30G                     # Новый размер (для изменения)
        fs_type: ext4                     # Тип ФС (если требуется для resizefs)
        mount_point: /var/log             # Точка монтирования
        preserve: true                    # Использовать safe-mount
        selinux_context: var_log_t        # SELinux-контекст
        allow_dangerous_resize: true      # Разрешить изменение критичной точки
        state: present                    # Управление томом: present | absent
```


### 9. Пример плейбука для нескольких машин
```yaml
# ANSIBLE_STDOUT_CALLBACK=yaml
- name: Управление разделами, LVM и ФС
  hosts: all
  become: true

  vars:
    reboot_if_changed: false

    disk_configs_by_host:
      '192.168.1.2': &db_850
        - operation_type: create
          device: /dev/sda
          device_partition_number: 4
          vg_name: rhel
          add_to_existing_vg: true
        - operation_type: extend
          vg_name: rhel
          lvs_to_modify:
            - name: root
              new_size: 50G
              mount_point: /
              fs_type: ext4
              allow_dangerous_resize: true
            - name: log_lv
              new_size: "100%FREE"
              mount_point: /var/log
              fs_type: ext4
              selinux_context: var_log_t
        - operation_type: create
          device: /dev/sdb
          device_partition_number: 1
          create_disklabel_if_missing: true
          disklabel_type: gpt
          vg_name: sdb_vg
          lvs_to_create:
            - name: data_lv
              size: "100%FREE"
              mount_point: /data
              fs_type: ext4
              selinux_context: default_t

      '192.168.1.3': *db_850
      '192.168.1.4': *db_850

      '192.168.1.5': &mq_450
        - operation_type: create
          device: /dev/sda
          device_partition_number: 4
          vg_name: rhel
          add_to_existing_vg: true
        - operation_type: extend
          vg_name: rhel
          lvs_to_modify:
            - name: root
              new_size: 50G
              mount_point: /
              fs_type: ext4
              allow_dangerous_resize: true
            - name: log_lv
              new_size: "100%FREE"
              mount_point: /var/log
              fs_type: ext4
              selinux_context: var_log_t
        - operation_type: create
          device: /dev/sdb
          device_partition_number: 1
          create_disklabel_if_missing: true
          disklabel_type: gpt
          vg_name: sdb_vg
          lvs_to_create:
            - name: data_lv
              size: "100%FREE"
              mount_point: /data
              fs_type: ext4
              selinux_context: default_t

      '192.168.1.6': *mq_450
      '192.168.1.7': *mq_450
```