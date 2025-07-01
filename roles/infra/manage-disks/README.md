# Ansible Роль: `infra/manage-disks`

## Назначение:
Эта роль отвечает за конфигурирование дисков: создание, расширение, обнаружение и подготовку под монтирование.
Поддерживает операции:
- `create` - полноценное создание LV + FS + mount
- `extend` - увеличение LV
- `stretch` - расширение доступной емкости 

### Статус реализации: 

| Возможность               | Статус        |
|---------------------------|---------------|
| create                    |    Работает   |
| extend                    |    Работает   | 
| stretch                   |    Работает   |    


## Требуемые модули:
- Ansible Core = 2.14.8
- Proxmox VE 8.2.2 
- Python 3.11.2 на Ansible контроллере

## Таблица ключей из словаря

### Верхнеуровневые ключи 

| Ключ                           | Тип              | Описание                                                                 |
|--------------------------------|------------------|--------------------------------------------------------------------------|
| `operation_type`               | `string (enum)`  | Тип операции: `create`, `extend`, `stretch`                              |
| `device`                       | `string`         | Блочное устройство (например, `/dev/sdb`)                                |
| `device_partition_number`      | `integer`        | Номер создаваемого раздела (например, `1`)                               |
| `create_disklabel_if_missing`  | `bool`           | Создать метку (GPT), если отсутствует                                    |     
| `disklabel_type`               | `string`         | Тип метки: `gpt` (по умолчанию) или `msdos`                              |
| `add_to_existing_vg`           | `bool`           | Добавить к существующей Volume Group                                     |
| `vg_name`                      | `string`         | Название новой Volume Group                                              |
| `vg_to_extend`                 | `string`         | Существующая VG, которую нужно расширить                                 |
| `partition_to_stretch`         | `string`         | Раздел, который нужно "растянуть", например `/dev/sda3`                  |
| `lvs_to_create`                | `list(dict)`     | Список LVM томов, которые нужно создать                                  |
| `lvs_to_modify`                | `list(dict)`     | Список LVM томов, которые нужно изменить                                 |

### Вложенные ключи (`lvs_to_create` / `lvs_to_modify`)

| Ключ                     | Тип              | Описание                                                                 |
|--------------------------|------------------|--------------------------------------------------------------------------|
| `name`                   | `string`         | Имя Logical Volume                                                       |
| `size`                   | `string`         | Размер тома (например, `10G`, `100%FREE`)                                |
| `new_size`               | `string`         | Новый размер (только при `extend/stretch`)                               |
| `mount_point`            | `string`         | Точка монтирования                                                       |
| `fs_type`                | `string`         | Тип ФС: `ext4`, `xfs`, `btrfs`, `vfat`, `ntfs`, ...                      |
| `fs_label`               | `string`         | Метка ФС (опционально)                                                   |
| `preserve`               | `bool`           | Использовать safe-mount при наличии данных                               |
| `selinux_context`        | `string`         | Контекст SELinux, например `default_t`, `var_log_t` и т.д.               |
| `allow_dangerous_resize` | `bool`           | Разрешить изменение критичных точек (`/var`, `/home`, ...)               |
| `state`                  | `string`         | `present` или `absent`                                                   |

### Подробный [пример использования ключей](../../../docs/text/manage-disks/example_use_keys.md)

### Поддержка `resizefs` по типам файловых систем

| Файловая система | Пакет / утилита              | Поддержка `resizefs`          |
|------------------|------------------------------|-------------------------------|
| `xfs`            | `xfsprogs` (`xfs_growfs`)    | Да                            |
| `ext4`           | `e2fsprogs` (`resize2fs`)    | Да                            |
| `btrfs`          | `btrfs-progs`                | Да (`btrfs filesystem resize`)|
| `vfat`           | `dosfstools`                 | Нет                           |
| `ntfs`           | `ntfs-3g`, `ntfsresize`      | Частично                      |


### Подробно с **рекомендациями** и **предостережениями** можно ознакомится [здесь.](../../../docs/text/manage-disks/recommendations_and_warnings.md)

> Перед тем как использовать данную роль, рекомендую провести аудит дисков. Получив подробную информацию по дискам, легче будет написать данные в Ansible. Аудит можно провести с помощью плейбука который указан [здесь.](../../../docs/text/manage-disks/example_use_keys.md)