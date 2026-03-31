# EventReserve - Application Web de Gestion de Reservations d'Evenements

## Features

### User Side
- Register and login with username + password
- JWT token authentication (stored in cookie)
- Browse all upcoming events
- View event details (description, date, location, image)
- Reserve a seat with name, email, phone
- Confirmation page after reservation
- View personal reservation history

### Admin Side
- Secure login with ROLE_ADMIN privileges
- Dashboard with statistics (total events, reservations, available seats)
- Full CRUD on events with image upload
- View all reservations per event
- Delete reservations
- Secure logout

### Security
- JWT authentication via `LexikJWTAuthenticationBundle`
- WebAuthn / Passkeys support for passwordless login (custom service)
- Role-based access control: `ROLE_USER`, `ROLE_ADMIN`
- Protected admin routes
- CSRF protection on forms

---



## Project Structure

```
event-reservation/
├── config/
│   ├── packages/
│   │   ├── doctrine.yaml
│   │   ├── lexik_jwt_authentication.yaml
│   │   ├── security.yaml
│   │   └── ...
│   └── services.yaml
├── docker/
│   ├── nginx/default.conf
│   └── php/local.ini
├── migrations/
├── public/
│   └── uploads/events/          # Uploaded event images
├── src/
│   ├── Controller/
│   │   ├── AdminController.php
│   │   ├── ApiAuthController.php
│   │   ├── EventController.php
│   │   ├── HomeController.php
│   │   ├── PasskeyController.php
│   │   ├── ReservationController.php
│   │   └── SecurityController.php
│   ├── Entity/
│   │   ├── Event.php
│   │   ├── Reservation.php
│   │   └── User.php
│   ├── Repository/
│   └── Service/
│       └── PasskeyService.php
├── templates/
│   ├── admin/
│   ├── event/
│   ├── home/
│   ├── reservation/
│   ├── security/
│   └── base.html.twig
├── docker-compose.yml
├── Dockerfile
└── .env
```

---


2. **Start Docker services**
```bash
docker compose -f docker-compose.yml up -d --build
```

3. **Generate JWT keys**
```bash
docker compose -f docker-compose.yml exec app php bin/console lexik:jwt:generate-keypair
```

4. **Create the database schema**
```bash
docker compose -f docker-compose.yml exec app php bin/console doctrine:schema:create
```

5. **Create an admin user**

Hasher un mot de passe (remplace `monmotdepasse` par ce que tu veux) :
```bash
docker compose -f docker-compose.yml exec app php -r "echo password_hash('monmotdepasse', PASSWORD_BCRYPT);"
```

Copie le hash affiché, puis insère l'admin dans la base :
```bash
docker compose -f docker-compose.yml exec db psql -U event_user -d event_db -c \
  "INSERT INTO \"user\" (username, password, roles) VALUES ('admin', 'HASH_COPIE_ICI', '[\"ROLE_ADMIN\"]');"
```

6. **Access the application**
- App: http://localhost:8080
- Adminer (DB GUI): http://localhost:8081

---

### Manual Installation (without Docker)

**Requirements:** PHP 8.2+, Composer, PostgreSQL 15

1. **Install dependencies**
```bash
composer install
```

2. **Configure environment**
```bash
cp .env.example .env.local
# Set DATABASE_URL and other vars
```

3. **Generate JWT keys**
```bash
php bin/console lexik:jwt:generate-keypair
```

4. **Create database and schema**
```bash
php bin/console doctrine:database:create
php bin/console doctrine:schema:create
```

5. **Start development server**
```bash
symfony serve
# or
php -S localhost:8080 -t public/
```


---

## API Endpoints (JWT)

| Method | Endpoint                       | Auth        | Description                    |
|--------|--------------------------------|-------------|--------------------------------|
| POST   | `/api/login`                   | Public      | Get JWT token                  |
| POST   | `/api/register`                | Public      | Register new user              |
| GET    | `/api/profile`                 | ROLE_USER   | Get current user profile       |
| POST   | `/api/passkeys/register/options` | ROLE_USER | Start passkey registration     |
| POST   | `/api/passkeys/register/verify`  | ROLE_USER | Complete passkey registration  |
| POST   | `/api/passkeys/login/options`    | Public    | Start passkey authentication   |
| POST   | `/api/passkeys/login/verify`     | Public    | Complete passkey login + JWT   |

---


