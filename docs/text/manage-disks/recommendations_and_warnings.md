### Рекомендации и предостережения
- Никогда не выполняй `create` для системных путей (`/`, `/usr`, `/etc`, ...) — роль проверяет это, но всё равно требует осторожности.
- Для критических путей (`/var`, `/home`, и т.д.) при `extend/stretch`  необходимо указывать `allow_dangerous_resize: true`
- Перед запуском **всегда делай audit** и **бэкап** или **снэпшот** машины.
- Точки из списка `safe_mount_prohibited_mounts` (например, `/`, `/usr`, `/etc`, ...) автоматически исключаются из "safe mount" — даже если задать `preserve: true`.
- В режиме `stretch` используется `growpart`, `partprobe` и `pvresize`. После увеличения размера LV может быть выполнено расширение файловой системы.
- При `extend` и `stretch`, если не указан `fs_type`, он определяется автоматически через `blkid` (`blkid -s TYPE`), и используется как `effective_fs_type`.
- При наличии SELinux:
  - для стандартных путей (`/var/log`, `/home`, ) вызывается `restorecon -Rv`.
  - для кастомных путей вызывается `sefcontext` + `setype={{ selinux_context }}` + `restorecon` через `reload: true`.

#### Встроенные предохранители
Следующие пути считаются критичными и никогда не обрабатываются в режиме safe_mount, даже если явно задано `preserve: true`:
```yaml
### roles/infra/manage-disks/defaults/main
safe_mount_prohibited_mounts:
  - /
  - /boot
  - /boot/efi
  - /dev
  - /proc
  - /sys
  - /run
  - /etc
  - /var/lib

# Это защита от перезаписи или порчи системных путей, при которых может нарушиться загрузка или работа хоста.
```

Для следующих путей при включённом SELinux применяется автоматическое восстановление системного контекста через restorecon -Rv, без необходимости вручную задавать `selinux_context`:
```yaml
### roles/infra/manage-disks/defaults/main
selinux_protected_mount_points:
  - /var/log
  - /home
  - /var/www
  - /var/lib/mysql
```


### Поведение при монтировании и SELinux
- Все монтируемые точки автоматически прописываются в `/etc/fstab` с опциями: `defaults 0 2`.
- Монтирование выполняется немедленно после записи (`state: mounted`).
- Если SELinux включён:
  - Для путей `/var/log`, `/home`, `/var/www`, `/var/lib/mysql` вызывается `restorecon -Rv`.
  - Для остальных путей вызывается `community.general.sefcontext` и настраивается `setype`, затем вызывается `restorecon` (через `reload: true`).
- Для путей с `preserve: true` сначала сохраняются контексты (`stat -c '%C %n'`), затем восстанавливаются вручную через `chcon`.