# Коллекция Ansible для установки КриптоПро CSP

Коллекция `astra.cryptopro` предназначена для установки и базовой настройки КриптоПро CSP на Astra Linux SE.

Основной сценарий использования — remote-установка на целевой хост по SSH. Основная роль коллекции — `astra.cryptopro.cryptopro_csp`. Проверки установки вынесены отдельно в `molecule/default/verify.yml` и не выполняются внутри runtime-роли.

## Что делает коллекция

Роль `astra.cryptopro.cryptopro_csp`:

- устанавливает КриптоПро CSP из архива, предоставленного пользователем;
- устанавливает необходимые зависимости;
- добавляет путь к утилитам КриптоПро CSP через `/etc/profile.d/cryptopro.sh`;
- включает и запускает сервис `pcscd`;
- не выполняет verify-проверки после установки.

## Требования

- Astra Linux SE на целевом хосте;
- Ansible 2.14 или новее на управляющей машине;
- SSH-доступ к целевому хосту;
- права `become` для установки пакетов и системной настройки;
- архив КриптоПро CSP, полученный пользователем самостоятельно с официального сайта КриптоПро после регистрации и авторизации.

Архив КриптоПро CSP не хранится в репозитории и не должен добавляться в коллекцию.

## Передача архива

Поддерживаются два способа передачи архива:

1. `cryptopro_csp_archive_path` — путь к архиву.
   - локальный файл на управляющей машине;
   - файл, уже размещённый на целевом хосте.
2. `cryptopro_csp_archive_url` — URL для скачивания архива на целевой хост.

Дополнительные переменные:

- `cryptopro_csp_archive_checksum` — контрольная сумма архива, например `sha256:...`;
- `cryptopro_csp_archive_headers` — HTTP-заголовки для скачивания;
- `cryptopro_csp_download_validate_certs` — проверка TLS-сертификата при загрузке.

## Быстрый старт

### Remote

Это основной сценарий использования.

Скопируйте пример inventory и заполните его своими значениями:

```bash
cp examples/inventory.ini.example inventory.ini
```

Пример содержимого `inventory.ini`:

```ini
[cryptopro_targets]
target1 ansible_host=YOUR_HOST_IP ansible_user=YOUR_USERNAME

[cryptopro_targets:vars]
ansible_become=true
```

Замените `YOUR_HOST_IP` и `YOUR_USERNAME` на реальные значения.

Запуск установки из локального архива:

```bash
ansible-playbook -i inventory.ini examples/install_cryptopro_remote.yml -k -K \
  -e cryptopro_csp_archive_path=/path/to/linux-amd64_deb.tgz
```

Используемый пример playbook: `examples/install_cryptopro_remote.yml`.

### Local

Локальная установка доступна как дополнительный сценарий.

```bash
ansible-playbook -i localhost, examples/install_cryptopro_local.yml -K
```

Если путь к архиву нужно передать явно:

```bash
ansible-playbook -i localhost, examples/install_cryptopro_local.yml -K \
  -e cryptopro_csp_archive_path=/path/to/linux-amd64_deb.tgz
```

### URL

Если архив нужно скачать на целевой хост, используйте `cryptopro_csp_archive_url`:

```bash
ansible-playbook -i inventory.ini examples/install_cryptopro_remote.yml -k -K \
  -e cryptopro_csp_archive_url=https://example.invalid/linux-amd64_deb.tgz
```

Пример с дополнительными параметрами:

```bash
ansible-playbook -i inventory.ini examples/install_cryptopro_remote.yml -k -K \
  -e cryptopro_csp_archive_url=https://example.invalid/linux-amd64_deb.tgz \
  -e cryptopro_csp_archive_checksum=sha256:YOUR_CHECKSUM \
  -e cryptopro_csp_download_validate_certs=true
```

При необходимости передайте HTTP-заголовки через `cryptopro_csp_archive_headers`.

### Force reinstall

По умолчанию повторная установка не выполняется, если КриптоПро CSP уже установлен. Для принудительной переустановки используйте `cryptopro_csp_force_reinstall=true`.

```bash
ansible-playbook -i inventory.ini examples/install_cryptopro_remote.yml -k -K \
  -e cryptopro_csp_archive_path=/path/to/linux-amd64_deb.tgz \
  -e cryptopro_csp_force_reinstall=true
```

## Поведение роли

Роль `astra.cryptopro.cryptopro_csp` выполняет только установку и базовую настройку.

- завершится с ошибкой, если не задан `cryptopro_csp_archive_path` и не задан `cryptopro_csp_archive_url`;
- завершится с ошибкой, если архив не найден или не был скачан;
- завершится с ошибкой, если после распаковки не найден `install.sh`;
- не выполняет повторную установку, если КриптоПро CSP уже установлен и `cryptopro_csp_force_reinstall=false`.

Файл `/etc/profile.d/cryptopro.sh` применяется только в новых shell-сессиях. Для текущей сессии может потребоваться повторный вход или запуск новой оболочки.

## Molecule

Verify-проверки вынесены в `molecule/default/verify.yml` и запускаются отдельно от runtime-роли.

```bash
export CRYPTOPRO_CSP_ARCHIVE_PATH="/path/to/linux-amd64_deb.tgz"
molecule test -s default
```

В контейнерных окружениях возможны ограничения, связанные с `systemd` и `pcscd`. Такие ограничения не обязательно указывают на проблему в самой роли.

## Роль

- `astra.cryptopro.cryptopro_csp` — установка и базовая настройка КриптоПро CSP.

Дополнительные примеры находятся в каталоге `examples/`.
