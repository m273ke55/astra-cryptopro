# astra.cryptopro

Ansible Collection для установки и базовой настройки КриптоПро CSP на Astra Linux Special Edition (SE), базовый уровень защищенности.

## Что делает коллекция

Коллекция содержит **одну роль** `cryptopro_csp` и реализует:

- установку КриптоПро CSP из локального архива `artifacts/linux-amd64_deb.tgz`;
- установку зависимостей (`pcscd`, `libccid`) и опционально `libgost-astra`;
- системную настройку `PATH` через `/etc/profile.d/cryptopro.sh`;
- включение и запуск сервиса `pcscd`;
- базовые проверки установки (пакеты, доступность `cpconfig`, опциональная проверка лицензии).

## Требования

- Astra Linux SE;
- Ansible >= 2.14;
- запуск на той же VM: `hosts: localhost`, `connection: local`, `become: true`;
- архив КриптоПро находится в `artifacts/linux-amd64_deb.tgz` (или переопределён переменной роли).

## Быстрый старт

```bash
ansible-playbook -i localhost, examples/install_cryptopro_local.yml
```

## Роль

- `astra.cryptopro.cryptopro_csp` — установка и базовая конфигурация КриптоПро CSP.

Подробности — в `roles/cryptopro_csp/README.md`.
