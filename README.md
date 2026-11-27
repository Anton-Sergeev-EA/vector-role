# Ansible role: Vector

Устанавливает проверенный SHA-256 пакет Vector, формирует YAML-конфигурацию,
проверяет её через `vector validate` до замены и включает сервис systemd.
Изменение конфигурации вызывает handler перезапуска. Роль создана командой
`ansible-galaxy role init vector-role` на основе исходного playbook.

## Требования

Ubuntu 22.04 / Debian 12, x86_64, systemd, Python 3, Ansible Core 2.16.14.
Нужны root-права через `become` и исходящий HTTPS. Проверки CI выполняются на Ubuntu 22.04.
Версии закреплены для воспроизводимости учебного задания.

## Структура

`defaults/main.yml` — пользовательские параметры с низким приоритетом;
`vars/main.yml` — внутренние значения реализации; `tasks/main.yml` — действия;
`templates/` — шаблоны; `meta/main.yml` — метаданные и зависимости.
Внешние адреса и учётные данные не зашиты в роль.

## Параметры

| Переменная | По умолчанию | Назначение |
| --- | --- | --- |
| `vector_version` | `0.46.1` | Версия DEB-пакета |
| `vector_package_checksum` | SHA-256 из defaults | Контрольная сумма соответствующей версии; обновлять вместе с ней |
| `vector_clickhouse_endpoint` | `http://127.0.0.1:8123` | Адрес ClickHouse, доступный с хоста Vector |
| `vector_clickhouse_database` | `logs` | База назначения |
| `vector_clickhouse_table` | `events` | Таблица назначения |
| `vector_demo_interval` | `5` | Период генерации события в секундах |

Внутренние переменные: `vector_package_arch=amd64`, `vector_binary_path=/usr/bin/vector`.

## Зависимости и использование

Другие роли автоматически не вызываются. До запуска нужна таблица
`logs.events (message String, timestamp DateTime64(3))` в доступном ClickHouse.
Этот пример генерирует синтетические события, а не читает системные журналы.
Объём дискового буфера — 256 MiB; при заполнении используется backpressure.

```yaml
- name: Configure logging agent
  hosts: vector
  become: true
  roles:
    - role: vector
      vector_clickhouse_endpoint: http://10.30.0.10:8123
```

Установка закреплённого выпуска:

```bash
ansible-galaxy role install git+https://github.com/Anton-Sergeev-EA/vector-role.git,1.0.0,vector
ANSIBLE_ROLES_PATH=~/.ansible/roles ansible-playbook -i tests/inventory tests/test.yml --syntax-check
```

`--check` на пустом хосте пропускает установку DEB и конфигурирование отсутствующего
сервиса. Проверка не заменяет реальный запуск. Повторное применение должно давать `changed=0`.

Лицензия: MIT. [Полный стенд и проверки](https://github.com/Anton-Sergeev-EA/roles).
[Документация Vector](https://vector.dev/docs/).
