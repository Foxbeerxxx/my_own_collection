# Домашнее задание к занятию "`Создание собственных модулей`" - `Татаринцев А.А.`


### Задание 1

1. `Делаю подготовку`

![1](https://github.com/Foxbeerxxx/my_own_collection/blob/main/img/img1.png)

2. `Начинаю выполнение основного задания, захожу в виртуальное окружение`
```
. venv/bin/activate && . hacking/env-setup
```
3. `В виртуальном окружении создаю файл my_own_module.py`
```
touch my_own_module.py
```
4. `Наполняю файл содержимым`
```
#!/usr/bin/python

# Copyright: (c) 2018, Your Name <your.email@example.org>
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)

from __future__ import (absolute_import, division, print_function)
__metaclass__ = type

DOCUMENTATION = '''
---
module: my_test

short_description: This module creates a text file with given content on a specified path.

version_added: "1.0.0"

description:
    - This module will create a text file on the remote host at a specified path with the given content.

options:
    path:
        description: The full path where the file should be created.
        required: true
        type: str
    content:
        description: The content that should be written into the file.
        required: true
        type: str

author:
    - Your Name (@yourGitHubHandle)
'''

EXAMPLES = '''
- name: Create a text file with content
  my_namespace.my_collection.my_test:
    path: /tmp/test_file.txt
    content: "Hello, Ansible!"
'''

RETURN = '''
path:
    description: The path to the file created.
    type: str
    returned: always
    sample: "/tmp/test_file.txt"
content:
    description: The content written into the file.
    type: str
    returned: always
    sample: "Hello, Ansible!"
'''

from ansible.module_utils.basic import AnsibleModule

def run_module():
    module_args = dict(
        path=dict(type='str', required=True),
        content=dict(type='str', required=True)
    )

    result = dict(
        changed=False,
        path='',
        content=''
    )

    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    # Check if file already exists
    try:
        with open(module.params['path'], 'r') as file:
            existing_content = file.read()
        if existing_content == module.params['content']:
            module.exit_json(changed=False, path=module.params['path'], content=existing_content)
    except FileNotFoundError:
        pass  # No file exists, will create a new one

    # Create or update the file
    with open(module.params['path'], 'w') as file:
        file.write(module.params['content'])

    result['changed'] = True
    result['path'] = module.params['path']
    result['content'] = module.params['content']

    module.exit_json(**result)

def main():
    run_module()

if __name__ == '__main__':
    main()

```

5. `Теперь запускаю команду Ansible для выполнения своего модуля:`
![2](https://github.com/Foxbeerxxx/my_own_collection/blob/main/img/img2.png)

6. `Для начала создадаю коллекцию, в которой будет храниться мой модуль и роль`

```
(venv) alexey@dellalexey:~/ansible_new$ ansible-galaxy collection init my_own_namespace.yandex_cloud_elk
[WARNING]: You are running the development version of Ansible. You should only run Ansible from "devel" if you are modifying the Ansible engine, or trying out features under development. This is a rapidly changing source of code and can become unstable at any point.
- Collection my_own_namespace.yandex_cloud_elk was created successfully
```

7. `Создаю роль для моей коллекции, которая будет использовать мой модуль.`

![3](https://github.com/Foxbeerxxx/my_own_collection/blob/main/img/img3.png)

```
ansible-galaxy init my_own_namespace.yandex_cloud_elk.roles.my_role
```
8. `Создаю файл playbook.yml с наполнением`

```
touch ~/ansible_new/ansible/playbook.yml
наполнение

---
- name: Create text file using custom role
  hosts: localhost
  roles:
    - my_own_namespace.yandex_cloud_elk.my_role

```

9. `Когда файл playbook.yml создан, перемещаю его в соответствующую директорию внутри роли:`
```
mv ~/ansible_new/ansible/playbook.yml my_own_namespace/yandex_cloud_elk/roles/my_role/tasks/main.yml

```
10. `Перехожу в корневой каталог коллекции `

```
cd ~/ansible_new/ansible/my_own_namespace/yandex_cloud_elk/
 создаю файл galaxy.yml

touch galaxy.yml
nano galaxy.yml
 и добавляю наполнение 

namespace: my_own_namespace
name: yandex_cloud_elk
version: 1.0.0
readme: README.md
author: AlexeyT
authors:
  - AlexeyT
  - Cat Kokos
description: Custom collection for managing Yandex Cloud ELK resources
license: MIT

```
11. `Собираю коллекцию в архив`
```
(venv) alexey@dellalexey:~/ansible_new/ansible/my_own_namespace/yandex_cloud_elk$ ansible-galaxy collection build ~/ansible_new/ansible/my_own_namespace/yandex_cloud_elk
[WARNING]: You are running the development version of Ansible. You should only run Ansible from "devel" if you are modifying the Ansible engine, or trying out features under development. This is a rapidly changing source of code and can become unstable at any point.
[WARNING]: Found unknown keys in collection galaxy.yml at '/home/alexey/ansible_new/ansible/my_own_namespace/yandex_cloud_elk/galaxy.yml': author
Created collection for my_own_namespace.yandex_cloud_elk at /home/alexey/ansible_new/ansible/my_own_namespace/yandex_cloud_elk/my_own_namespace-yandex_cloud_elk-1.0.0.tar.gz
```
12. `Создаю каталог mkdir ~/ansible_new/ansible/playbook_and_collection`
13. `Перемещаю плейбук и архив `
```
mv ~/ansible_new/ansible/playbook.yml ~/ansible_new/ansible/yandex_cloud_elk-1.0.0.tar.gz /home/alexey/ansible_new/my_own_collection
```
14. `Устанавливаю коллекцию из локального архива`
```
ansible-galaxy collection install ./my_own_namespace-yandex_cloud_elk-1.0.0.tar.gz
```
![4](https://github.com/Foxbeerxxx/my_own_collection/blob/main/img/img4.png)


15. `Запускаю playbook`
```
ansible-playbook playbook.yml
```
![5](https://github.com/Foxbeerxxx/my_own_collection/blob/main/img/img5.png)
