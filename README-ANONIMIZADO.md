# Como Criar um Ambiente de Desenvolvimento para o Site do Cliente

> Este guia detalha o processo de criação de um ambiente de desenvolvimento baseado em uma cópia do site principal, usando Nginx e WordPress.

## 1. Copiar o arquivo de configuração do Nginx
```bash
cp dominio.com.conf dev.dominio.com.conf
sed -i "s/dominio.com/dev.dominio.com/g" dev.dominio.com.conf
```
Não esqueça de ajustar as seguintes linhas do dev.dominio.com.conf:

root  /var/www/dev.portalaltadefinicao; # Essa linha define o diretório onde está o Wordpress do domínio

# Comentar as duas linhas do certificado LetsEncrypt pois pertence ao domínio principal do arquivo que copiamos, você instalar o certificado do ambiente dev posteriormente.
ssl_certificate /etc/letsencrypt/live/www.homologacao.dominio.com/fullchain.pem; # managed by Certbot
ssl_certificate_key /etc/letsencrypt/live/www.homologacao.dominio.com/privkey.pem; # managed by Certbot

Verifique se existe alguma regra no Nginx que possa redirecionar para o site principal, caso tenha, comente ou apague ela.

## 2. Copiar os arquivos do WordPress
```bash
rsync -av --exclude 'wp-content/uploads' /var/www/portalaltadefinicao/ /var/www/dev.portalaltadefinicao/
```

## 3. Ativar nova configuração do Nginx
```bash
ln -s /etc/nginx/sites-available/dev.dominio.com.conf /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

## 4. Backup do banco de dados
```bash
bash /opt/serverdoin/scripts/backup-bancos
cd /var/www/mysql/
gzip -d portalaltadefinicao-Thursday.sql.gz
```

## 5. Criar novo banco e usuário
```sql
mysql -u root -p
CREATE DATABASE dev_portal;
CREATE USER 'dev_user'@'localhost' IDENTIFIED BY 'senha_segura';
GRANT ALL PRIVILEGES ON dev_portal.* TO 'dev_user'@'localhost';
FLUSH PRIVILEGES;
exit;
```

## 6. Importar banco de dados
```bash
mysql -u root -p dev_portal < portalaltadefinicao-Thursday.sql
```

## 7. Configurar `wp-config.php`
```php
define('DB_NAME', 'dev_portal');
define('DB_USER', 'dev_user');
define('DB_PASSWORD', 'senha_segura');
define('WP_HOME', 'https://dev.dominio.com');
define('WP_SITEURL', 'https://dev.dominio.com');
```

> Verifique e comente/ajuste qualquer redirecionamento para o domínio principal.

## 8. Atualizar a zona DNS
Adicione o seguinte registro:
```
dev.dominio.com. 900 A 190.89.238.155
```

## 9. Gerar certificado SSL com Certbot
```bash
sudo certbot --nginx -d www.dev.dominio.com -d dev.dominio.com
```

## 10. Desativar temporariamente o plugin CDN
```bash
mv wp-content/plugins/serverdoin-cdn wp-content/plugins/serverdoin-cdn.old
```

---

**Pronto!** O ambiente de desenvolvimento está configurado.
