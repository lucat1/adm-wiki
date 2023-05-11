Host adm-saragozza
HostName 130.136.3.2
LocalForward 8200 localhost:8200
LocalForward 3000 localhost:3000
User luca

Host adm-pve1
ProxyJump adm-saragozza
HostName 130.136.3.3
User luca
LocalForward 8006 localhost:8006

Host adm-pve2
ProxyJump adm-saragozza
HostName 130.136.3.4
User luca
LocalForward 8006 localhost:8006

Host adm-pve3
ProxyJump adm-saragozza
HostName 130.136.3.5
User luca
LocalForward 8006 localhost:8006

Host adm-pve4
ProxyJump adm-saragozza
HostName 130.136.3.6
User luca
LocalForward 8006 localhost:8006

Host adm-pve5
ProxyJump adm-saragozza
HostName 130.136.3.7
User luca
LocalForward 8006 localhost:8006
