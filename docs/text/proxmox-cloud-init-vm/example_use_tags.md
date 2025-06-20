## Пример использования тегов

### 1. Пример использования тегов в playbook
```yaml
- name: Выполнить только cloud-init
  hosts: localhost
  gather_facts: false
  vars_files:
    - examples/vm_list_full.yml

  roles:
    - role: infra/proxmox-cloud-init-vm
  tags: [cloud_init]
```

### 2. Примеры запуска с тегами из CLI
- Запуск только клонирования     
`ansible-playbook playbook.yml --tags clone`
- Пропустить удаление ВМ      
`ansible-playbook playbook.yml --skip-tags delete`   
- Выполнить cloud-init и запуск       
`ansible-playbook playbook.yml --tags "cloud_init,start"`