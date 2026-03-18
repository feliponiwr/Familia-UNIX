📝 Código para Copiar e Colar no GitHubMarkdown# 🛠️ Manual de Configuração de Servidores Softdesk
> **Responsável:** Sérgio Fischer | **Versão:** Rocky Linux 9.4 (Blue Onyx)

---

## 📑 Índice
1. [Sistema Operacional](#1-sistema-operacional)
2. [Monitoramento (Netdata)](#2-monitoramento-netdata)
3. [Área de SWAP](#3-área-de-swap)
4. [Apache (HTTPD)](#4-apache-httpd)
5. [MariaDB 10.6](#5-mariadb-106)
6. [PHP & Composer](#6-php--composer)
7. [E-mail (Relay Gmail)](#7-e-mail-relay-gmail)
8. [Clonagem de Servidor](#8-clonagem-de-servidor)
9. [Provisionamento de Clientes](#9-provisionamento-de-clientes)
10. [Gerenciamento de Disco (LVM)](#10-gerenciamento-de-disco-lvm)
11. [Migração de Dados](#11-migração-de-dados)
12. [Rotinas de Backup](#12-rotinas-de-backup)

---

## 1. 📦 Sistema Operacional
Instalação de dependências essenciais e ferramentas de compilação.

```bash
yum -y install yum-utils zip zlib epel-release wget bash gawk vim-enhanced sudo gcc autoconf automake make gettext pcre2 perl jq tar fuse langpacks-pt_BR.noarch logrotate telnet traceroute htop iftop mytop iotop ntpstat ntpsec postfix cyrus-sasl-plain httpd httpd-tools php83-php php83-php-opcache python3-certbot-apache mod_http2
2. 📊 Monitoramento (Netdata)Configuração do agente e acesso ao banco de dados.[!TIP]O Netdata pode ser acessado via navegador na porta 19999.Bash# Instalação automática
wget -O /tmp/netdata-kickstart.sh [https://get.netdata.cloud/kickstart.sh](https://get.netdata.cloud/kickstart.sh) && sh /tmp/netdata-kickstart.sh

# Acesso ao MariaDB
mysql -e "CREATE USER 'netdata'@'localhost' IDENTIFIED BY 'SUA_SENHA'; GRANT USAGE, REPLICATION CLIENT, PROCESS ON *.* TO 'netdata'@'localhost'; FLUSH PRIVILEGES;"
3. 💾 Área de SWAPCriação de arquivos de escape para garantir a estabilidade da memória.Bashfor n in 1 2; do 
  dd if=/dev/zero of=/.swapfile.$n count=512 bs=1M
  mkswap /.swapfile.$n
  swapon /.swapfile.$n
  chmod 600 /.swapfile.$n
done
4. 🌐 Apache (HTTPD)Otimização do motor de processamento (MPM Event).Apache# Local: /etc/httpd/conf/httpd.conf
<IfModule mpm_event_module>
    StartServers             2
    ServerLimit               96
    MaxRequestWorkers        256
    MaxConnectionsPerChild   512
</IfModule>
5. 🗄️ MariaDB 10.6Setup do repositório oficial e ativação do serviço.Bashcurl -LsS [https://r.mariadb.com/downloads/mariadb_repo_setup](https://r.mariadb.com/downloads/mariadb_repo_setup) | sudo bash
dnf install -y mariadb-server mariadb
systemctl enable --now mariadb
6. 🐘 PHP & ComposerConfiguração do ambiente de execução e dependências.[!IMPORTANT]Altere o UMask para 0002 no arquivo de serviço do PHP-FPM para evitar problemas de permissão de escrita.Bashphp -r "copy('[https://getcomposer.org/installer](https://getcomposer.org/installer)', 'composer-setup.php');"
php composer-setup.php --install-dir /usr/bin --filename composer
7. 📧 E-mail (Relay Gmail)Configuração do Postfix para envios autenticados.Bash# Configurar em /etc/postfix/sasl_passwd
[smtp.gmail.com]:587 seu-email@soft4.com.br:senha_de_aplicativo

# Comandos de ativação
chmod 600 /etc/postfix/sasl_passwd
postmap /etc/postfix/sasl_passwd
systemctl restart postfix
8. 👯 Clonagem de ServidorProcedimento para novos nós de produção.Bashsh checknewClone.sh unilnxpro99 nome_do_cliente
for s in softdesk mariadb netdata php83-php-fpm ntpd; do systemctl enable --now $s; done
9. 🚀 Provisionamento de ClientesAtivação de diretórios e VirtualHosts.Bashmkdir -p /var/www/vhosts/cliente.soft4.com.br/logs
ln -s /etc/httpd/sites-available/cliente.conf /etc/httpd/sites-enabled/
10. 🏗️ Gerenciamento de Disco (LVM)Preparação de volumes para armazenamento de dados.Bashpvcreate /dev/sdb
vgcreate VGDB /dev/sdb
lvcreate -l 100%FREE -n mariadb VGDB
mkfs.xfs /dev/VGDB/mariadb
11. 🔄 Migração de DadosMovendo o banco de dados para o novo disco de forma segura.[!CAUTION]Nunca mova os arquivos com o serviço do MariaDB rodando.Bashsystemctl stop mariadb
mv /var/lib/mysql /var/lib/mysql.old
mkdir /var/lib/mysql
mount /dev/VGDB/mariadb /var/lib/mysql
cp -rp /var/lib/mysql.old/* /var/lib/mysql/
chown -R mysql:mysql /var/lib/mysql
systemctl start mariadb
12. ☁️ Rotinas de BackupPolíticas de retenção de dados dos Vhosts.FrequênciaTipoHorário (Dom)RetençãoMensalFull (tgz)03:056 MesesSemanalIncremental03:054 SemanasDiárioBanco/Anexos22:207 Dias
---

### 💡 Dicas rápidas para quem está começando no Git:

1.  **Preview:** Antes de salvar, clique na aba **"Preview"** no topo do editor do GitHub para ver se a formatação ficou do seu agrado.
2.  **Alertas:** Note que usei `> [!TIP]`, `> [!IMPORTANT]` e `> [!CAUTION]`. Isso cria aquelas caixas coloridas que você viu no exemplo.
3.  **Tabelas:** O Passo 12 usa uma tabela Markdown. Se precisar adicionar colunas, basta seguir o padrão de barras `|`.

**Gostaria que eu te explicasse como criar esse arquivo diretamente pela interface do GitHub, passo a passo?**
