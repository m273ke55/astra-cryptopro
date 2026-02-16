# Role: cryptopro_csp

Роль устанавливает и выполняет базовую настройку КриптоПро CSP на Astra Linux SE из локального архива `linux-amd64_deb.tgz`.

## Требования

- Astra Linux Special Edition (SE);
- запуск локально на целевой VM с правами `become: true`;
- архив КриптоПро размещён в коллекции: `artifacts/linux-amd64_deb.tgz` (или задан альтернативный путь).

## Поведение

1. Проверяет наличие локального архива установки.
2. Устанавливает зависимости: `pcscd`, `libccid`, и опционально `libgost-astra`.
3. Распаковывает архив в `cryptopro_remote_dir`.
4. Находит `install.sh` (или другой скрипт из `cryptopro_install_script`) и запускает установку.
5. Создаёт профиль PATH в `/etc/profile.d/cryptopro.sh` (опционально).
6. Включает и запускает `pcscd`.
7. Выполняет проверки: наличие пакетов `cprocsp*`, запуск `cpconfig`, опциональную проверку лицензии.

## Variables

См. `defaults/main.yml`.

## Пример запуска

```yaml
- name: Install CryptoPro CSP locally
  hosts: localhost
  connection: local
  become: true
  gather_facts: true
  roles:
    - role: astra.cryptopro.cryptopro_csp
```

Запуск:

```bash
ansible-playbook -i localhost, examples/install_cryptopro_local.yml
```
