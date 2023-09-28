# Switch

Lo switch si trova nella sottorete `130.136.0.0/24`. Per accedere è necessario
collegarsi in ssh ad una macchina connessa con lo swtich (es. saragozza) e 
successivamente darsi un ip nella sottorete:
```bash
# ip a add 130.136.0.n/24 dev {device}
```
`n` dovrebbe essere un ip non ancora in uso nella sottorete. `{device}` è 
l'interfaccia di rete a cui volete assegnare l'ip. Si possono elencare le 
interfacce con `ip a`.

Lo switch ha ip `130.136.0.253`.

Nonostante abbia un'interfaccia web per il managment non siamo riusciti a trovare
il modo di farla funzionare. Per configurarlo è quindi necessario utilizzare 
`telnet`.

```bash
telnet 130.136.0.253
```


