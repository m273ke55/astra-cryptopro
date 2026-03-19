# Коллекция Ansible для установки и базовой настройки КриптоПро CSP для Astra Automation

Ansible Collection для установки и базовой настройки КриптоПро CSP на Astra Linux Special Edition (SE), базовый уровень защищенности.

## Что делает коллекция

Коллекция содержит **одну runtime-роль** `cryptopro_csp` и отдельный Molecule scenario для верификации.

Runtime-роль выполняет:

- установку КриптоПро CSP из архива, который пользователь передаёт через `cryptopro_csp_archive_path` или `cryptopro_csp_archive_url`;
- установку зависимостей (`pcscd`, `libccid`) и опционально `libgost-astra`;
- системную настройку `PATH` через `/etc/profile.d/cryptopro.sh`;
- включение и запуск сервиса `pcscd`.

Runtime-роль **не выполняет** post-install verify-проверки (`dpkg`, `cpconfig -help`, `csptest -enum -info`, проверку лицензии). Эти проверки вынесены в `molecule/default/verify.yml`.

## Требования

- Astra Linux SE;
- Ansible >= 2.14;
- запуск на той же VM: `hosts: localhost`, `connection: local`, `become: true`;
- архив CryptoPro CSP пользователь получает самостоятельно с официального сайта CryptoPro после регистрации/авторизации.

## Поддерживаемые способы передачи архива

1. `cryptopro_csp_archive_path` — локальный путь на контроллере Ansible или путь на целевом хосте.
2. `cryptopro_csp_archive_url` — URL для скачивания архива на целевой хост.

Опционально поддерживаются:

- `cryptopro_csp_archive_checksum` — checksum в формате `sha256:...`;
- `cryptopro_csp_archive_headers` — HTTP headers для `get_url`;
- `cryptopro_csp_download_validate_certs` — включить/выключить проверку TLS-сертификата.

## Быстрый старт

### Вариант 1: локальный архив

```bash
ansible-playbook -i localhost, examples/install_cryptopro_local.yml -vv -K
```

### Вариант 2: архив по URL

Передайте URL через extra-vars или переменные inventory/playbook:

```bash
ansible-playbook -i localhost, examples/install_cryptopro_local.yml -vv -K \
  -e cryptopro_csp_archive_path='' \
  -e cryptopro_csp_archive_url='https://example.invalid/linux-amd64_deb.tgz'
```

### Вариант 3: форс-переустановка

```bash
ansible-playbook -i localhost, examples/install_cryptopro_local.yml -vv -K \
  -e cryptopro_csp_archive_path='/path/to/linux-amd64_deb.tgz' \
  -e cryptopro_csp_force_reinstall=true
```

## Поведение runtime-роли

Роль выполняет только установку и базовую настройку:

- `fail`, если не задан ни `cryptopro_csp_archive_url`, ни `cryptopro_csp_archive_path`;
- `fail`, если архив не скачался или не найден;
- `fail`, если после распаковки не найден `install.sh`;
- не переустанавливает CryptoPro CSP при повторном запуске, если `cpconfig` уже найден и `cryptopro_csp_force_reinstall=false`.

Файл `/etc/profile.d/cryptopro.sh` применяется только в новых shell-сессиях.

## Molecule verify

Проверки установки запускаются отдельно через Molecule:

```bash
export CRYPTOPRO_CSP_ARCHIVE_PATH='/path/to/linux-amd64_deb.tgz'
molecule test -s default
```

При необходимости можно использовать URL и дополнительные переменные окружения `CRYPTOPRO_CSP_ARCHIVE_URL`, `CRYPTOPRO_CSP_ARCHIVE_CHECKSUM`, `CRYPTOPRO_CSP_ARCHIVE_HEADERS`, `CRYPTOPRO_CSP_VERIFY_LICENSE`.

## Роль

- `astra.cryptopro.cryptopro_csp` — установка и базовая конфигурация КриптоПро CSP.

Подробности — в `roles/cryptopro_csp/README.md`.
