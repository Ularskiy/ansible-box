# Ansible Роль: `infra/manage-ssh`

## Назначение:
Роль предназначена для управления SSH-ключами пользователей на хостах. Она может:
- создавать SSH-ключи локально;
- записывать публичные ключи в `authorized_keys` на удалённых машинах;
- удалять ключи при необходимости;
- логировать действия через роль `shared/log-events` (в планах);
- использовать самописанные параметры генерации ключей (тип, длина, комментарий);
- работать с пользователями в цикле (`loop: "{{ users }}"`).

### Статус реализации

| Возможность                         | Статус                   |
|-------------------------------------|--------------------------|
| Генерация SSH-ключей                | Работает              |
| Установка ключей на целевой хост    | Работает              |
| Удаление ключей с целевого хоста    | Работает              |
| Локальное хранение ключей           | Работает              |
| Поддержка разных типов ключей       | Работает              |
| Настройка комментариев в ключах     | Работает              |
| Повторная генерация (`force`)       | Работает              |
| Удаление локального ключа           | Работает              |
| Логирование действий                | Не дописан            |
| Автоматическое определение `~user`  | Работает              |
| Отправка ключей по e-mail           | Не дописан            |

## Требуемые модули:
- Ansible Core = 2.14.8
- Proxmox VE 8.2.2 
- Python 3.11.2 на Ansible контроллере

## Таблица ключей из словаря `users`
| Ключ                   | Тип     | Описание                                                                 |
|------------------------|---------|--------------------------------------------------------------------------|
| `name`                 | string  | Имя пользователя, для которого управляются ключи                         |
| `generate_ssh_key`     | bool    | Генерировать ли новый SSH-ключ локально                                  |
| `ssh_key_type`         | string  | Тип ключа (`rsa`, `ed25519`, `ecdsa`, и т.д.)                            |
| `ssh_key_bits`         | int     | Длина ключа (например, 2048 для RSA, 256 для ED25519)                    |
| `ssh_key_comment`      | string  | Комментарий, добавляемый в конец публичного ключа                        |
| `ssh_key_force`        | bool    | Перегенерировать ключ, если он уже существует                            |
| `ssh_state`            | string  | `present` или `absent`, для контроля записи ключа на хост                |
| `state`                | string  | `present` или `absent`, для полной логики создания/удаления              |
| `ssh_key_storage_path` | string  | Локальный путь для хранения ключей. По умолчанию: `/etc/ansible/ssh`     |
| `ssh_key_type_default` | string  | Тип ключа по умолчанию (`ed25519`, если не указано иное)                 |
| `ssh_key_bits_default` | int     | Размер ключа по умолчанию (например, `256` для `ed25519`)                |

### Подробный [пример использования ключей](../../../docs/text/manage-ssh/example_use_keys.md)

## Поддерживаемые типы SSH-ключей
| Тип ключа   | Значение в `ssh_key_type`  | Поддержка в роли | Размер ключа       (`ssh_key_bits`)    |
|-------------|----------------------------|------------------|----------------------------------------|
| RSA         | `rsa`                      |   Да             | 2048, 3072, 4096                       |
| ED25519     | `ed25519`                  |   Да             | 256 (игнорируется, т.к. фиксированный) |
| ECDSA       | `ecdsa`                    |   Да             | 256, 384, 521                          |
| DSA         | `dsa`                      |   Устарело       | 1024 (не рекомендуется)                |

### Подробный [пример использования типов ключей](../../../docs/text/manage-ssh/example_use_ssh_key.md)


### Структура подзадач в (utils/)

| Файл                            | Назначение                                                                |
|---------------------------------|---------------------------------------------------------------------------|
| `detect_home_path.yml`          | Определяет домашний каталог пользователя через `getent`                   |
| `remove_ssh_key_after_send.yml` | Удаляет ключи после отправки по почте                                     |
| `send_keys_to_email.yml`        | Читает ключи и отправляет их на email с использованием `mail:`            |
