# lesson_16
ДЗ. Первые шаги с Ansible

# Домашнее задание
Первые шаги с Ansible

# Исполнитель
Павел Смирнов

# Цель
сделать первые шаги с Ansible, написать свой первый playbook;

Описание/Пошаговая инструкция выполнения домашнего задания:

🎯 Что нужно сделать?

Подготовить стенд на Vagrant как минимум с одним сервером. На этом сервере, используя Ansible, необходимо развернуть nginx со следующими условиями:
- необходимо использовать модуль yum/apt;
- конфигурационные файлы должны быть взяты из шаблона jinja2 с переменными;
- после установки nginx должен быть в режиме enabled в systemd;
- должен быть использован notify для старта nginx после установки;
- сайт должен слушать на нестандартном порту — 8080, для этого использовать переменные в Ansible.

# Среда выполнения
VirtualBox 7.0.10, ВМ1 - Ubuntu 24.04 (srv2 - сервер с Ansible), ВМ2 - Ubuntu 24.04 (nginx - целевая машина)

# Команды и описание действий

> На сервере srv2 где у нас установлен Ansible в домашней папке текущего пользователя создадим папку Ansible. В ней создадим файл - inventory.ini с именами хостов на которых будет происходить установка и настройка ПО и конфигурационный файл ansible.cfg в котором будет ссылка на inventory.ini.
> Так как целевая машина nginx развернута при помощи Vagrant то на ней уже есть подходящий пользователь vagrant, его и будем использовать в ansible, так как на целевой машине ssh под него уже настроен, единственно нужно настроить ssh соединения на стороне srv2. Возьмем закрытый ключ для этой машины от vagrant (E:\Vagrant\ansible\.vagrant\machines\nginx\virtualbox\private_key) и скопируем содержимое через буфер в файл ~/.ssh/nginx_pk. А в файл ~/.ssh/config добавим для хоста nginx информацию о подключении, как описано ниже.
```
usr1@srv2:~/ansible$ ls
ansible.cfg                 inventory.ini  site.yml
ansible-full-reference.cfg  nginx.yml      templates
usr1@srv2:~/ansible$ cat inventory.ini
[srv]
srv1
nginx
usr1@srv2:~/ansible$ cat ansible.cfg
[defaults]
inventory = ./inventory.ini
[ssh_connection]
pipelining = True
usr1@srv2:~/ansible$
usr1@srv2:~/ansible$
usr1@srv2:~/ansible$ cat ~/.ssh/config
Host srv1
        Hostname 192.168.0.111
        Port 22
        User ansible
        IdentityFile ~/.ssh/ansible_ed25519
        IdentitiesOnly yes
Host web-shell
        Hostname 192.168.0.107
        Port 22
        User vagrant
        IdentityFile ~/.ssh/vagrant_pk
        IdentitiesOnly yes
Host nginx
        Hostname 192.168.0.150
        Port 22
        User vagrant
        IdentityFile ~/.ssh/nginx_pk
        IdentitiesOnly yes

usr1@srv2:~/ansible$ cat ~/.ssh/nginx_pk
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAA
AAtzc2gtZWQyNTUxOQAAACD5ybBabLpe8YXHPd3ZQztYsZKk+q9sa2dBfaBu
CJ1uogAAAJA5ZAq7OWQKuwAAAAtzc2gtZWQyNTUxOQAAACD5ybBabLpe8YXH
Pd3ZQztYsZKk+q9sa2dBfaBuCJ1uogAAAEBmYurS+IxFpucpmd20FcWkdlrX
GBlZvCH+nB4bVd4WDPnJsFpsul7xhcc93dlDO1ixkqT6r2xrZ0F9oG4InW6i
AAAAB3ZhZ3JhbnQBAgMEBQY=
-----END OPENSSH PRIVATE KEY-----
usr1@srv2:~/ansible$
usr1@srv2:~/ansible$ cat inventory.ini
[web]
srv1
nginx
usr1@srv2:~/ansible$ nano inventory.ini
usr1@srv2:~/ansible$ cat inventory.ini
[srv]
srv1
nginx
usr1@srv2:~/ansible$ nano nginx.yml
usr1@srv2:~/ansible$ nano nginx.yml
```
> Делаем первый плейбук по частям, по этому создадим ~/ansible/nginx.yml в котором будет базовые обновления списков установленного ПО и установка nginx
<img width="663" height="419" alt="image" src="https://github.com/user-attachments/assets/a8264422-36ba-465e-9eac-ba8858d3413a" />

> теперь проверим что целевая машина nginx доступна для ansible, путем запуска команды ping для хоста nginx. Так же при помощи ansible проверим версию ядра и состояние службы firewalld.

```
usr1@srv2:~/ansible$ ansible nginx -m ping
nginx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
usr1@srv2:~/ansible$ ansible nginx -m command -a "uname -r"
nginx | CHANGED | rc=0 >>
5.15.0-71-generic
usr1@srv2:~/ansible$ ansible nginx -m systemd -a name=firewalld
nginx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "name": "firewalld",
    "status": {
        "ActiveEnterTimestamp": "n/a",
        "ActiveEnterTimestampMonotonic": "0",
        "ActiveExitTimestamp": "n/a",
        "ActiveExitTimestampMonotonic": "0",
        "ActiveState": "inactive",
        "AllowIsolate": "no",
     ...
        "TimeoutAbortUSec": "1min 30s",
        "TimeoutCleanUSec": "infinity",
        "TimeoutStartFailureMode": "terminate",
        "TimeoutStartUSec": "1min 30s",
        "TimeoutStopFailureMode": "terminate",
        "TimeoutStopUSec": "1min 30s",
        "TimerSlackNSec": "50000",
        "Transient": "no",
        "UID": "[not set]",
        "UMask": "0022",
        "UtmpMode": "init",
        "WatchdogSignal": "6",
        "WatchdogTimestamp": "n/a",
        "WatchdogTimestampMonotonic": "0",
        "WatchdogUSec": "infinity"
    }
}
usr1@srv2:~/ansible$ ansible-playbook nginx.yml

PLAY [NGINX | Install and configure NGINX] *************************************

TASK [Gathering Facts] *********************************************************
ok: [nginx]

TASK [Update apt cache] ********************************************************
changed: [nginx]

TASK [NGINX | Install NGINX] ***************************************************
changed: [nginx]

PLAY RECAP *********************************************************************
nginx                      : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

usr1@srv2:~/ansible$
usr1@srv2:~/ansible$
usr1@srv2:~/ansible$
usr1@srv2:~/ansible$
usr1@srv2:~/ansible$ ls
ansible.cfg  ansible-full-reference.cfg  inventory.ini  nginx.yml  site.yml
usr1@srv2:~/ansible$ mkdir templates
usr1@srv2:~/ansible$ nano templates/nginx.conf.j2
usr1@srv2:~/ansible$ nano templates/nginx.conf.j2
usr1@srv2:~/ansible$ nano nginx.yml
usr1@srv2:~/ansible$ ansible-playbook nginx.yml

PLAY [NGINX | Install and configure NGINX] *************************************

TASK [Gathering Facts] *********************************************************
ok: [nginx]

TASK [Update apt cache] ********************************************************
changed: [nginx]

TASK [NGINX | Install NGINX] ***************************************************
ok: [nginx]

TASK [NGINX | Create NGINX config file from template] **************************
changed: [nginx]

RUNNING HANDLER [reload nginx] *************************************************
changed: [nginx]

PLAY RECAP *********************************************************************
nginx                      : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

usr1@srv2:~/ansible$ nano nginx.yml
usr1@srv2:~/ansible$ ansible-playbook nginx.yml

PLAY [NGINX | Install and configure NGINX] *************************************

TASK [Gathering Facts] *********************************************************
ok: [nginx]

TASK [Update apt cache] ********************************************************
changed: [nginx]

TASK [NGINX | Install NGINX] ***************************************************
ok: [nginx]

TASK [NGINX | Create NGINX config file from template] **************************
changed: [nginx]

RUNNING HANDLER [reload nginx] *************************************************
changed: [nginx]

PLAY RECAP *********************************************************************
nginx                      : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

usr1@srv2:~/ansible$ curl http://192.168.0.150:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
usr1@srv2:~/ansible$
```









# Протокол работы

```
usr1@srv2:~/ansible$ ls
ansible.cfg                 inventory.ini  site.yml
ansible-full-reference.cfg  nginx.yml      templates
usr1@srv2:~/ansible$ cat inventory.ini
[srv]
srv1
nginx
usr1@srv2:~/ansible$ cat ansible.cfg
[defaults]
inventory = ./inventory.ini
[ssh_connection]
pipelining = True
usr1@srv2:~/ansible$
usr1@srv2:~/ansible$
usr1@srv2:~/ansible$ cat ~/.ssh/config
Host srv1
        Hostname 192.168.0.111
        Port 22
        User ansible
        IdentityFile ~/.ssh/ansible_ed25519
        IdentitiesOnly yes
Host web-shell
        Hostname 192.168.0.107
        Port 22
        User vagrant
        IdentityFile ~/.ssh/vagrant_pk
        IdentitiesOnly yes
Host nginx
        Hostname 192.168.0.150
        Port 22
        User vagrant
        IdentityFile ~/.ssh/nginx_pk
        IdentitiesOnly yes

usr1@srv2:~/ansible$ cat ~/.ssh/nginx_pk
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAA
AAtzc2gtZWQyNTUxOQAAACD5ybBabLpe8YXHPd3ZQztYsZKk+q9sa2dBfaBu
CJ1uogAAAJA5ZAq7OWQKuwAAAAtzc2gtZWQyNTUxOQAAACD5ybBabLpe8YXH
Pd3ZQztYsZKk+q9sa2dBfaBuCJ1uogAAAEBmYurS+IxFpucpmd20FcWkdlrX
GBlZvCH+nB4bVd4WDPnJsFpsul7xhcc93dlDO1ixkqT6r2xrZ0F9oG4InW6i
AAAAB3ZhZ3JhbnQBAgMEBQY=
-----END OPENSSH PRIVATE KEY-----
usr1@srv2:~/ansible$
usr1@srv2:~/ansible$ cat inventory.ini
[web]
srv1
nginx
usr1@srv2:~/ansible$ nano inventory.ini
usr1@srv2:~/ansible$ cat inventory.ini
[srv]
srv1
nginx
usr1@srv2:~/ansible$ nano nginx.yml
usr1@srv2:~/ansible$ nano nginx.yml
usr1@srv2:~/ansible$ ansible nginx -m ping
nginx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
usr1@srv2:~/ansible$ ansible nginx -m command -a "uname -r"
nginx | CHANGED | rc=0 >>
5.15.0-71-generic
usr1@srv2:~/ansible$ ansible nginx -m systemd -a name=firewalld
nginx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "name": "firewalld",
    "status": {
        "ActiveEnterTimestamp": "n/a",
        "ActiveEnterTimestampMonotonic": "0",
        "ActiveExitTimestamp": "n/a",
        "ActiveExitTimestampMonotonic": "0",
        "ActiveState": "inactive",
        "AllowIsolate": "no",
        "AssertResult": "no",
        "AssertTimestamp": "n/a",
        "AssertTimestampMonotonic": "0",
        "BlockIOAccounting": "no",
        "BlockIOWeight": "[not set]",
        "CPUAccounting": "yes",
        "CPUAffinityFromNUMA": "no",
        "CPUQuotaPerSecUSec": "infinity",
        "CPUQuotaPeriodUSec": "infinity",
        "CPUSchedulingPolicy": "0",
        "CPUSchedulingPriority": "0",
        "CPUSchedulingResetOnFork": "no",
        "CPUShares": "[not set]",
        "CPUUsageNSec": "[not set]",
        "CPUWeight": "[not set]",
        "CacheDirectoryMode": "0755",
        "CanFreeze": "yes",
        "CanIsolate": "no",
        "CanReload": "no",
        "CanStart": "no",
        "CanStop": "yes",
        "CapabilityBoundingSet": "cap_chown cap_dac_override cap_dac_read_search cap_fowner cap_fsetid cap_kill cap_setgid cap_setuid cap_setpcap cap_linux_immutable cap_net_bind_service cap_net_broadcast cap_net_admin cap_net_raw cap_ipc_lock cap_ipc_owner cap_sys_module cap_sys_rawio cap_sys_chroot cap_sys_ptrace cap_sys_pacct cap_sys_admin cap_sys_boot cap_sys_nice cap_sys_resource cap_sys_time cap_sys_tty_config cap_mknod cap_lease cap_audit_write cap_audit_control cap_setfcap cap_mac_override cap_mac_admin cap_syslog cap_wake_alarm cap_block_suspend cap_audit_read cap_perfmon cap_bpf cap_checkpoint_restore",
        "CleanResult": "success",
        "CollectMode": "inactive",
        "ConditionResult": "no",
        "ConditionTimestamp": "n/a",
        "ConditionTimestampMonotonic": "0",
        "ConfigurationDirectoryMode": "0755",
        "ControlPID": "0",
        "CoredumpFilter": "0x33",
        "DefaultDependencies": "yes",
        "DefaultMemoryLow": "0",
        "DefaultMemoryMin": "0",
        "Delegate": "no",
        "Description": "firewalld.service",
        "DevicePolicy": "auto",
        "DynamicUser": "no",
        "ExecMainCode": "0",
        "ExecMainExitTimestamp": "n/a",
        "ExecMainExitTimestampMonotonic": "0",
        "ExecMainPID": "0",
        "ExecMainStartTimestamp": "n/a",
        "ExecMainStartTimestampMonotonic": "0",
        "ExecMainStatus": "0",
        "FailureAction": "none",
        "FileDescriptorStoreMax": "0",
        "FinalKillSignal": "9",
        "FreezerState": "running",
        "GID": "[not set]",
        "GuessMainPID": "yes",
        "IOAccounting": "no",
        "IOReadBytes": "18446744073709551615",
        "IOReadOperations": "18446744073709551615",
        "IOSchedulingClass": "2",
        "IOSchedulingPriority": "4",
        "IOWeight": "[not set]",
        "IOWriteBytes": "18446744073709551615",
        "IOWriteOperations": "18446744073709551615",
        "IPAccounting": "no",
        "IPEgressBytes": "[no data]",
        "IPEgressPackets": "[no data]",
        "IPIngressBytes": "[no data]",
        "IPIngressPackets": "[no data]",
        "Id": "firewalld.service",
        "IgnoreOnIsolate": "no",
        "IgnoreSIGPIPE": "yes",
        "InactiveEnterTimestamp": "n/a",
        "InactiveEnterTimestampMonotonic": "0",
        "InactiveExitTimestamp": "n/a",
        "InactiveExitTimestampMonotonic": "0",
        "JobRunningTimeoutUSec": "infinity",
        "JobTimeoutAction": "none",
        "JobTimeoutUSec": "infinity",
        "KeyringMode": "private",
        "KillMode": "control-group",
        "KillSignal": "15",
        "LimitAS": "infinity",
        "LimitASSoft": "infinity",
        "LimitCORE": "infinity",
        "LimitCORESoft": "0",
        "LimitCPU": "infinity",
        "LimitCPUSoft": "infinity",
        "LimitDATA": "infinity",
        "LimitDATASoft": "infinity",
        "LimitFSIZE": "infinity",
        "LimitFSIZESoft": "infinity",
        "LimitLOCKS": "infinity",
        "LimitLOCKSSoft": "infinity",
        "LimitMEMLOCK": "94044160",
        "LimitMEMLOCKSoft": "94044160",
        "LimitMSGQUEUE": "819200",
        "LimitMSGQUEUESoft": "819200",
        "LimitNICE": "0",
        "LimitNICESoft": "0",
        "LimitNOFILE": "1048576",
        "LimitNOFILESoft": "1048576",
        "LimitNPROC": "2712",
        "LimitNPROCSoft": "2712",
        "LimitRSS": "infinity",
        "LimitRSSSoft": "infinity",
        "LimitRTPRIO": "0",
        "LimitRTPRIOSoft": "0",
        "LimitRTTIME": "infinity",
        "LimitRTTIMESoft": "infinity",
        "LimitSIGPENDING": "2712",
        "LimitSIGPENDINGSoft": "2712",
        "LimitSTACK": "infinity",
        "LimitSTACKSoft": "8388608",
        "LoadError": "org.freedesktop.systemd1.NoSuchUnit \"Unit firewalld.service not found.\"",
        "LoadState": "not-found",
        "LockPersonality": "no",
        "LogLevelMax": "-1",
        "LogRateLimitBurst": "0",
        "LogRateLimitIntervalUSec": "0",
        "LogsDirectoryMode": "0755",
        "MainPID": "0",
        "ManagedOOMMemoryPressure": "auto",
        "ManagedOOMMemoryPressureLimit": "0",
        "ManagedOOMPreference": "none",
        "ManagedOOMSwap": "auto",
        "MemoryAccounting": "yes",
        "MemoryAvailable": "infinity",
        "MemoryCurrent": "[not set]",
        "MemoryDenyWriteExecute": "no",
        "MemoryHigh": "infinity",
        "MemoryLimit": "infinity",
        "MemoryLow": "0",
        "MemoryMax": "infinity",
        "MemoryMin": "0",
        "MemorySwapMax": "infinity",
        "MountAPIVFS": "no",
        "NFileDescriptorStore": "0",
        "NRestarts": "0",
        "NUMAPolicy": "n/a",
        "Names": "firewalld.service",
        "NeedDaemonReload": "no",
        "Nice": "0",
        "NoNewPrivileges": "no",
        "NonBlocking": "no",
        "NotifyAccess": "none",
        "OOMScoreAdjust": "0",
        "OnFailureJobMode": "replace",
        "OnSuccessJobMode": "fail",
        "Perpetual": "no",
        "PrivateDevices": "no",
        "PrivateIPC": "no",
        "PrivateMounts": "no",
        "PrivateNetwork": "no",
        "PrivateTmp": "no",
        "PrivateUsers": "no",
        "ProcSubset": "all",
        "ProtectClock": "no",
        "ProtectControlGroups": "no",
        "ProtectHome": "no",
        "ProtectHostname": "no",
        "ProtectKernelLogs": "no",
        "ProtectKernelModules": "no",
        "ProtectKernelTunables": "no",
        "ProtectProc": "default",
        "ProtectSystem": "no",
        "RefuseManualStart": "no",
        "RefuseManualStop": "no",
        "ReloadResult": "success",
        "RemainAfterExit": "no",
        "RemoveIPC": "no",
        "Restart": "no",
        "RestartKillSignal": "15",
        "RestartUSec": "100ms",
        "RestrictNamespaces": "no",
        "RestrictRealtime": "no",
        "RestrictSUIDSGID": "no",
        "Result": "success",
        "RootDirectoryStartOnly": "no",
        "RuntimeDirectoryMode": "0755",
        "RuntimeDirectoryPreserve": "no",
        "RuntimeMaxUSec": "infinity",
        "SameProcessGroup": "no",
        "SecureBits": "0",
        "SendSIGHUP": "no",
        "SendSIGKILL": "yes",
        "StandardError": "inherit",
        "StandardInput": "null",
        "StandardOutput": "inherit",
        "StartLimitAction": "none",
        "StartLimitBurst": "5",
        "StartLimitIntervalUSec": "10s",
        "StartupBlockIOWeight": "[not set]",
        "StartupCPUShares": "[not set]",
        "StartupCPUWeight": "[not set]",
        "StartupIOWeight": "[not set]",
        "StateChangeTimestamp": "n/a",
        "StateChangeTimestampMonotonic": "0",
        "StateDirectoryMode": "0755",
        "StatusErrno": "0",
        "StopWhenUnneeded": "no",
        "SubState": "dead",
        "SuccessAction": "none",
        "SyslogFacility": "3",
        "SyslogLevel": "6",
        "SyslogLevelPrefix": "yes",
        "SyslogPriority": "30",
        "SystemCallErrorNumber": "2147483646",
        "TTYReset": "no",
        "TTYVHangup": "no",
        "TTYVTDisallocate": "no",
        "TasksAccounting": "yes",
        "TasksCurrent": "[not set]",
        "TasksMax": "813",
        "TimeoutAbortUSec": "1min 30s",
        "TimeoutCleanUSec": "infinity",
        "TimeoutStartFailureMode": "terminate",
        "TimeoutStartUSec": "1min 30s",
        "TimeoutStopFailureMode": "terminate",
        "TimeoutStopUSec": "1min 30s",
        "TimerSlackNSec": "50000",
        "Transient": "no",
        "UID": "[not set]",
        "UMask": "0022",
        "UtmpMode": "init",
        "WatchdogSignal": "6",
        "WatchdogTimestamp": "n/a",
        "WatchdogTimestampMonotonic": "0",
        "WatchdogUSec": "infinity"
    }
}
usr1@srv2:~/ansible$ ansible-p
ansible-playbook  ansible-pull
usr1@srv2:~/ansible$ ansible-playbook nginx.yml
ansible.cfg                 -k
ansible-full-reference.cfg  -K
--ask-become-pass           --key-file
--ask-pass                  -l
--ask-vault-pass            --limit
--ask-vault-password        --list-hosts
-b                          --list-tags
--become                    --list-tasks
--become-method             -M
--become-pass-file          --module-path
--become-password-file      nginx.yml
--become-user               --private-key
-c                          --scp-extra-args
-C                          --sftp-extra-args
--check                     site.yml
--connection                --skip-tags
--connection-password-file  --ssh-common-args
--conn-pass-file            --ssh-extra-args
-D                          --start-at-task
--diff                      --step
-e                          --syntax-check
--extra-vars                -t
-f                          -T
--flush-cache               --tags
--force-handlers            --timeout
--forks                     -u
-h                          --user
--help                      -v
-i                          --vault-id
--inventory                 --vault-pass-file
--inventory-file            --vault-password-file
inventory.ini               --verbose
-J                          --version
usr1@srv2:~/ansible$ ansible-playbook nginx.yml

PLAY [NGINX | Install and configure NGINX] *************************************

TASK [Gathering Facts] *********************************************************
ok: [nginx]

TASK [Update apt cache] ********************************************************
changed: [nginx]

TASK [NGINX | Install NGINX] ***************************************************
changed: [nginx]

PLAY RECAP *********************************************************************
nginx                      : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

usr1@srv2:~/ansible$
usr1@srv2:~/ansible$
usr1@srv2:~/ansible$
usr1@srv2:~/ansible$
usr1@srv2:~/ansible$ ls
ansible.cfg  ansible-full-reference.cfg  inventory.ini  nginx.yml  site.yml
usr1@srv2:~/ansible$ mkdir templates
usr1@srv2:~/ansible$ nano templates/nginx.conf.j2
usr1@srv2:~/ansible$ nano templates/nginx.conf.j2
usr1@srv2:~/ansible$ nano nginx.yml
usr1@srv2:~/ansible$ ansible-playbook nginx.yml

PLAY [NGINX | Install and configure NGINX] *************************************

TASK [Gathering Facts] *********************************************************
ok: [nginx]

TASK [Update apt cache] ********************************************************
changed: [nginx]

TASK [NGINX | Install NGINX] ***************************************************
ok: [nginx]

TASK [NGINX | Create NGINX config file from template] **************************
changed: [nginx]

RUNNING HANDLER [reload nginx] *************************************************
changed: [nginx]

PLAY RECAP *********************************************************************
nginx                      : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

usr1@srv2:~/ansible$ nano nginx.yml
usr1@srv2:~/ansible$ ansible-playbook nginx.yml

PLAY [NGINX | Install and configure NGINX] *************************************

TASK [Gathering Facts] *********************************************************
ok: [nginx]

TASK [Update apt cache] ********************************************************
changed: [nginx]

TASK [NGINX | Install NGINX] ***************************************************
ok: [nginx]

TASK [NGINX | Create NGINX config file from template] **************************
changed: [nginx]

RUNNING HANDLER [reload nginx] *************************************************
changed: [nginx]

PLAY RECAP *********************************************************************
nginx                      : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

usr1@srv2:~/ansible$ curl http://192.168.0.150:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
usr1@srv2:~/ansible$

```

