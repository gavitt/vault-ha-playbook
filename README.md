Инсталляция HashiCorp Vault в HA режиме с integrated storage, с созданием сертификатам. 
**Playbook тестировался на Debian 11**

1. Для работы кластера необходимо минимум 3 сервера, в inventory нужно указать ip адреса и hostname серверов.
2. В первую очередь необходимо сгенерировать CA сертификат, для этого используется playbook  `ca-sert.yml`
3. Для установки и конфигурирования Vault необходимо запустить playbook `vault.yml`. Перед запуском необходимо отредактировать файл переменных `roles/vault/vars/main.yml`. В нем необходимо задать версию Vault и hostname серверов.
4. Далее необходимо инициировать хранилище Vault, для этого необходимо выполнить следующую команду на первой ноде кластера:

```bash
  vault operator init  
```

В результате вам будет сгенерировано пять ключей для распечатывания хранилища и root-токен для доступа.
Распечатать хранилище можно через UI или CLI 

```bash
vault operator unseal
vault operator unseal
vault operator unseal
```

> **_INFO:_**  На второй и третьей ноде распечатываем хранилище, используя полученные ключи при инициализации первой ноды.

5. Для балансировки доступа можно использовать nginx, минимально рабочий конфиг:

```
upstream vault{
        ip_hash;
        server 192.168.77.25:8200 max_fails=0 fail_timeout=5s;
        server 192.168.77.26:8200 max_fails=0 fail_timeout=5s;
        server 192.168.77.27:8200 max_fails=0 fail_timeout=5s;
}

server {
        listen  *:8200;

        location / {
          proxy_pass https://vault/;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto "https";
      }
}

```