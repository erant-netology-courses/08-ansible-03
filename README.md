# 08-ansible-03

Задачу сделал на Ubuntu и нашел ошибку в 02 задаче. Разбил на несколько плейбуков, на мой взгляд такая изоляция лучше. Все линты пофиксил.

Картинки с результатами в корне репозитория.

# ClickHouse Playbook

Установка ClickHouse на Ubuntu/Debian.

## Что делает playbook

1. Скачивает DEB-пакеты ClickHouse (client, server, common-static) из официального репозитория
2. Устанавливает пакеты через `dpkg`
3. Запускает сервис `clickhouse-server`
4. Создаёт базу данных `logs`

## Параметры

| Переменная | Значение по умолчанию | Описание |
|-----------|----------------------|----------|
| `clickhouse_version` | `25.11.3.54` | Версия ClickHouse для установки |
| `clickhouse_packages` | `[clickhouse-client, clickhouse-server, clickhouse-common-static]` | Список пакетов |

## Теги

Теги не заданы.

## Запуск

```bash
# Полная установка
ansible-playbook -i inventory/prod.yml site.yml

# Проверка без применения
ansible-playbook -i inventory/prod.yml site.yml --check

# Только на одном хосте
ansible-playbook -i inventory/prod.yml site.yml --limit clickhouse-01
```

## Инвентори для ClickHouse

Хост должен находиться в группе `clickhouse`.

```yaml
---
clickhouse:
  hosts:
    clickhouse-01:
      ansible_host: <IP_адрес>
      ansible_user: <пользователь>
      ansible_ssh_private_key_file: <путь_к_ключу>
```

# Vector Playbook

Установка и настройка Vector — легковесного агента для сбора и передачи логов.

## Что делает playbook

1. Скачивает архив Vector из официального репозитория
2. Распаковывает бинарник в `/opt/vector`
3. Деплоит конфиг `vector.toml` из Jinja2-шаблона
4. Создаёт systemd unit-файл с автоматическим поиском пути к бинарнику
5. Запускает сервис и добавляет в автозагрузку

## Параметры

| Переменная | Значение по умолчанию | Описание |
|-----------|----------------------|----------|
| `vector_version` | `0.31.0` | Версия Vector |
| `vector_url` | `https://packages.timber.io/vector/{{ vector_version }}/vector-{{ vector_version }}-x86_64-unknown-linux-gnu.tar.gz` | URL для скачивания |
| `vector_install_dir` | `/opt/vector` | Директория установки |
| `vector_interval` | `1` | Интервал генерации демо-логов (в шаблоне) |

## Теги

Теги не заданы.

## Запуск

```bash
# Полная установка
ansible-playbook -i inventory/prod.yml vector.yml

# Только деплой конфига
ansible-playbook -i inventory/prod.yml vector.yml --tags config

# Проверка без применения
ansible-playbook -i inventory/prod.yml vector.yml --check
```

## Инвентори для Vector

Хост должен находиться в группе `vector` и ставится на локальную машину.

```yaml
---
vector:
  hosts:
    vector-01:
      ansible_connection: local
```

# Lighthouse Playbook

Установка и настройка Lighthouse — веб-интерфейса для ClickHouse.

## Что делает playbook

1. Устанавливает nginx и git
2. Клонирует репозиторий Lighthouse из GitHub в `/var/www/html/lighthouse`
3. Деплоит конфиг nginx для Lighthouse на порт 80
4. Включает сайт и удаляет дефолтный конфиг nginx
5. Перезапускает nginx при изменениях

## Параметры

| Переменная | Значение по умолчанию | Описание |
|-----------|----------------------|----------|
| `lighthouse_repo` | `https://github.com/VKCOM/lighthouse.git` | URL репозитория Lighthouse |
| `lighthouse_dest` | `/var/www/html/lighthouse` | Путь установки |
| `lighthouse_version` | `master` | Ветка/тег для клонирования |

## Теги

| Тег | Действие |
|-----|---------|
| `install` | Установка nginx и git |
| `nginx` | Всё, что связано с nginx: клонирование Lighthouse, деплой конфига, активация сайта, удаление default |

## Запуск

```bash
# Полная установка
ansible-playbook -i inventory/prod.yml lighthouse.yml

# Только установка пакетов
ansible-playbook -i inventory/prod.yml lighthouse.yml --tags install

# Только настройка nginx
ansible-playbook -i inventory/prod.yml lighthouse.yml --tags nginx
```
