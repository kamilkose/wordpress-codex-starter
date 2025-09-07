# WordPress + "Codex" (ChatGPT Ajan) Akışı — Starter

Bu depo, **WordPress**’i Docker ile ayağa kaldırıp **GitHub üzerinden sürümlü yönetmek** ve “Codex” tarzı bir geliştirici ajan ile
PR/özellik akışını yürütmek için hazır bir iskelet sağlar.

## Bileşenler
- Docker Compose: **Nginx + PHP-FPM (wordpress:php8-fpm) + MySQL 8**
- WP-CLI (container içinden) ile ilk kurulum ve yönetim
- GitHub Actions ile **SSH** üzerinden sunucuya **deploy**
- Versiyon kontrolü: sadece `wp-content/` (özellikle `themes/codex-theme`) repoda tutulur
- Üretimde WordPress çekirdeği ve eklentiler WP-CLI ile kurulur/güncellenir

> **Neden böyle?** WordPress çekirdeği ve üçüncü parti eklentiler repoda taşınmazsa merge/upgrade süreçleri basitleşir.
> Kendi yazdıklarımız (`wp-content/`) ise tam sürüm kontrolünde kalır.

---

## Hızlı Başlangıç (Lokal)

1) `.env` oluşturun
```bash
cp .env.example .env
# .env içindeki ADMIN_*, DB_* ve DOMAIN_* değerlerini doldurun
```

2) Docker’ı kaldırın
```bash
docker compose up -d --build
```

3) İlk kurulum (WP-CLI)
```bash
# WP-CLI wrapper
./bin/wp core download --force
./bin/wp config create --dbname=$DB_NAME --dbuser=$DB_USER --dbpass=$DB_PASSWORD --dbhost=db --skip-check --force
./bin/wp core install --url="$SITE_URL" --title="$SITE_TITLE" --admin_user="$ADMIN_USER" --admin_password="$ADMIN_PASS" --admin_email="$ADMIN_EMAIL"
./bin/wp language core install tr_TR --activate
./bin/wp rewrite structure '/%postname%/' --hard
```

4) Tema geliştirme
```bash
# Kendi temanızı wp-content/themes/codex-theme içinde geliştirin
# Örnek index.php ve style.css mevcut
```

5) Site: http://localhost:8080

---

## Deploy (Sunucuya)

Sunucuda gereksinimler:
- Docker / Docker Compose
- Bir klasör (örn. `/var/www/wpapp`)

GitHub **Secrets**:
- `SSH_PRIVATE_KEY`, `DEPLOY_HOST`, `DEPLOY_USER`, `DEPLOY_PATH`

Adımlar:
1) Sunucuda `DEPLOY_PATH` klasörünü oluşturun ve kullanıcı izinlerini verin.
2) Repoya push → **Actions → Deploy** job’ı tetiklenir.
3) Job, dosyaları sunucuya `rsync` ile kopyalar ve `docker compose up -d` çalıştırır.

> **Domain ve HTTPS:** Nginx örnek konfig’i eklidir. Üretimde Let’s Encrypt için traefik/certbot ekleyebilirsiniz.

---

## Codex (ChatGPT ajan) ile çalışma önerisi

- Görev yaz: `feat(theme): header’da menü + arama; responsive`  
- Ajanın görevi: `wp-content/themes/codex-theme` içinde değişiklikler, gerekirse `functions.php`, `header.php`, `searchform.php`, SCSS/CSS; ardından PR ve açıklama.
- İnceleme → Merge → Actions deploy.

**Ajan komut şablonu:**
```
Repo: wordpress-codex-starter
Hedef: wp-content/themes/codex-theme
İş: feat(theme): header ve responsive navigasyon. Lighthouse en az 90/100. Gerekirse küçük JS.
Çıktı: Değişen dosyalar, test adımları, PR açıklaması.
```

Keyifli kurulumlar! 🚀
