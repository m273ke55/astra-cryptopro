# Role: cryptopro_csp

Роль устанавливает и выполняет базовую настройку КриптоПро CSP на Astra Linux SE.

## Требования

- Astra Linux Special Edition (SE);
- запуск локально на целевой VM с правами `become: true`;
- архив CryptoPro CSP пользователь получает самостоятельно с официального сайта CryptoPro после регистрации/авторизации.

## Supported input

Роль поддерживает два способа передачи архива:

1. `cryptopro_csp_archive_path` — путь к архиву на контроллере Ansible или на целевом хосте.
2. `cryptopro_csp_archive_url` — URL, по которому роль скачает архив через `ansible.builtin.get_url`.

Дополнительно можно передать:

- `cryptopro_csp_archive_checksum` — checksum архива (`sha256:...`);
- `cryptopro_csp_archive_headers` — словарь HTTP-заголовков;
- `cryptopro_csp_download_validate_certs` — проверка TLS-сертификатов при скачивании.

## Поведение

1. Проверяет, что задан `cryptopro_csp_archive_path` или `cryptopro_csp_archive_url`.
2. Устанавливает зависимости: `pcscd`, `libccid`, и опционально `libgost-astra`.
3. При `cryptopro_csp_archive_url` скачивает архив во временный каталог `cryptopro_csp_remote_dir`.
4. При `cryptopro_csp_archive_path` использует архив с контроллера или целевого хоста.
5. Распаковывает архив в `cryptopro_csp_remote_dir`.
6. Находит `install.sh` (или другой скрипт из `cryptopro_csp_install_script`) и запускает установку.
7. Создаёт профиль PATH в `/etc/profile.d/cryptopro.sh` (опционально).
8. Включает и запускает `pcscd`.

Post-install verify-проверки не выполняются в runtime-роли. Они вынесены в `molecule/default/verify.yml`.

## Variables

См. `defaults/main.yml`.

## Примеры

### Запуск с локальным архивом

```yaml
- name: Install CryptoPro CSP locally
  hosts: localhost
  connection: local
  become: true
  gather_facts: true

  vars:
    cryptopro_csp_archive_path: "/path/to/linux-amd64_deb.tgz"

  roles:
    - role: astra.cryptopro.cryptopro_csp
```

Запуск:

```bash
ansible-playbook -i localhost, examples/install_cryptopro_local.yml -vv -K
```

### Запуск с URL

```yaml
- name: Install CryptoPro CSP from URL
  hosts: localhost
  connection: local
  become: true
  gather_facts: true

  vars:
    cryptopro_csp_archive_path: ""
    cryptopro_csp_archive_url: "https://example.invalid/linux-amd64_deb.tgz"
    cryptopro_csp_archive_checksum: "sha256:..."
    cryptopro_csp_archive_headers:
      Authorization: "Bearer <token>"

  roles:
    - role: astra.cryptopro.cryptopro_csp
```

### Форс-переустановка

```yaml
- name: Force reinstall CryptoPro CSP
  hosts: localhost
  connection: local
  become: true
  gather_facts: true

  vars:
    cryptopro_csp_archive_path: "/path/to/linux-amd64_deb.tgz"
    cryptopro_csp_force_reinstall: true

  roles:
    - role: astra.cryptopro.cryptopro_csp
```

Файл `/etc/profile.d/cryptopro.sh` применяется в новых shell-сессиях.
