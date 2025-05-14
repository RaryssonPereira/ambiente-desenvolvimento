# Como Criar um Ambiente de Desenvolvimento para o Site do Cliente

> Este guia detalha o processo de criação de um ambiente de desenvolvimento baseado em uma cópia do site principal, usando Nginx e WordPress.

## 1. Copiar o arquivo de configuração do Nginx

No diretório `/etc/nginx/sites-available`, execute:
```bash
cp dominio.com.conf dev.dominio.com.conf
sed -i "s/dominio.com/dev.dominio.com/g" dev.dominio.com.conf
```

Não esqueça de ajustar a seguinte linha no `dev.dominio.com.conf`:
```nginx
root /var/www/dev.dominio; # Essa linha define o diretório onde está o WordPress do domínio
```

### Comentar certificados SSL antigos (serão regenerados para o domínio dev)
Comente as linhas que apontam para os certificados do domínio original, pois eles não funcionarão no subdomínio de desenvolvimento:

```nginx
# ssl_certificate /etc/letsencrypt/live/www.dominio.com/fullchain.pem; # managed by Certbot
# ssl_certificate_key /etc/letsencrypt/live/www.dominio.com/privkey.pem; # managed by Certbot
```

Verifique também se há alguma regra de redirecionamento para o domínio principal e comente ou remova para evitar redirecionamentos indesejados.

## 2. Ativar nova configuração do Nginx

Crie o link simbólico no diretório `/etc/nginx/sites-enabled` e reinicie o Nginx:
```bash
ln -s /etc/nginx/sites-available/dev.dominio.com.conf /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

## 3. Copiar os arquivos do WordPress

Copie os arquivos do WordPress, exceto a pasta de uploads, para evitar transferir arquivos de mídia grandes e agilizar o processo:
```bash
rsync -av --exclude 'wp-content/uploads' /var/www/dominio/ /var/www/dev.dominio/
```

## 4. Backup do banco de dados
```bash
bash /opt/serverdoin/scripts/backup-bancos
cd /var/www/mysql/
gzip -d dominio-Thursday.sql.gz
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
mysql -u root -p dev_portal < dominio-Thursday.sql
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

Adicione o seguinte registro A na sua zona DNS:
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
