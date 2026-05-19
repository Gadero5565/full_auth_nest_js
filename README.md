# NestJS JWT Auth API

A simple authentication and authorization API built with **NestJS**, **PostgreSQL**, **TypeORM**, **Passport JWT**, and **Role-Based Access Control (RBAC)**.

The application supports user registration, login with JWT access tokens, protected profile access, user listing for managers/admins, and role updates for admins.

---

## Features

- User registration
- User login with JWT token generation
- Password hashing using bcrypt
- JWT authentication using Passport
- Protected routes with `JwtAuthGuard`
- Role-based access control using `RolesGuard`
- Supported roles:
  - `user`
  - `manager`
  - `admin`
- Admin user seeding on application startup
- PostgreSQL database integration with TypeORM
- DTO validation using `class-validator`
- Global validation pipe
- CORS enabled

---

## Tech Stack

- [NestJS](https://nestjs.com/)
- [TypeORM](https://typeorm.io/)
- [PostgreSQL](https://www.postgresql.org/)
- [Passport JWT](https://www.passportjs.org/packages/passport-jwt/)
- [bcrypt](https://www.npmjs.com/package/bcrypt)
- [class-validator](https://www.npmjs.com/package/class-validator)
- [@nestjs/config](https://docs.nestjs.com/techniques/configuration)

---

## Project Structure

```text
src/
├── auth/
│   ├── dtos/
│   │   ├── login.dto.ts
│   │   ├── register.dto.ts
│   │   ├── update-role.dto.ts
│   │   └── users.dto.ts
│   ├── auth.controller.ts
│   ├── auth.module.ts
│   ├── auth.service.ts
│   ├── jwt-auth.guard.ts
│   ├── jwt.strategy.ts
│   ├── roles.decorator.ts
│   ├── roles.guard.ts
│   └── user.entity.ts
├── seeding/
│   └── user.seeder.ts
├── app.module.ts
└── main.ts
```

---

## Environment Variables

Create a `.env` file in the project root.

```env
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=postgres
DB_NAME=auth_db

JWT_SECRET=replace_with_a_strong_secret
JWT_EXPIRES_IN=3600s
```

You can also copy the provided example file:

```bash
cp .env.example .env
```

> Do not commit your real `.env` file to GitHub.

---

## Installation

```bash
npm install
```

---

## Database Setup

Make sure PostgreSQL is running and create a database named:

```text
auth_db
```

Example using PostgreSQL CLI:

```bash
createdb auth_db
```

Or using Docker:

```bash
docker run --name auth-postgres \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=auth_db \
  -p 5432:5432 \
  -d postgres
```

---

## Running the Application

### Development

```bash
npm run start:dev
```

### Production

```bash
npm run build
npm run start:prod
```

The API will run on:

```text
http://localhost:3000
```

---

## Default Admin User

When the application starts, it checks if an admin user already exists.

If no admin exists, it creates one:

```text
Username: admin
Email: admin@example.com
Password: admin123
Role: admin
```

> This account is useful for development only. Change the default credentials before using the app in production.

---

## API Endpoints

### Register User

```http
POST /auth/register
```

Request body:

```json
{
  "username": "john",
  "email": "john@example.com",
  "password": "password123"
}
```

Response:

```json
{
  "message": "User registered successfully"
}
```

---

### Login

```http
POST /auth/login
```

Request body:

```json
{
  "username": "john",
  "password": "password123"
}
```

Response:

```json
{
  "accessToken": "jwt_token_here"
}
```

Use the returned token in protected routes:

```http
Authorization: Bearer jwt_token_here
```

---

### Get Profile

```http
GET /auth/profile
```

Access: authenticated users

Headers:

```http
Authorization: Bearer jwt_token_here
```

Response:

```json
{
  "id": 1,
  "username": "john",
  "email": "john@example.com",
  "role": "user"
}
```

---

### Get All Users

```http
GET /auth/users
```

Access: `manager`, `admin`

Headers:

```http
Authorization: Bearer jwt_token_here
```

Response:

```json
[
  {
    "id": 1,
    "username": "john",
    "email": "john@example.com"
  }
]
```

---

### Update User Role

```http
PATCH /auth/users/role
```

Access: `admin`

Headers:

```http
Authorization: Bearer jwt_token_here
```

Request body:

```json
{
  "userId": 1,
  "role": "manager"
}
```

Response:

```json
{
  "message": "User role updated successfully",
  "user": {
    "id": 1,
    "username": "john",
    "role": "manager"
  }
}
```

---

## Roles and Permissions

| Endpoint | User | Manager | Admin |
|---|---:|---:|---:|
| `POST /auth/register` | ✅ | ✅ | ✅ |
| `POST /auth/login` | ✅ | ✅ | ✅ |
| `GET /auth/profile` | ✅ | ✅ | ✅ |
| `GET /auth/users` | ❌ | ✅ | ✅ |
| `PATCH /auth/users/role` | ❌ | ❌ | ✅ |

---

## Authentication Flow

1. A user registers using `/auth/register`.
2. The password is hashed before being stored in the database.
3. The user logs in using `/auth/login`.
4. The server validates the username and password.
5. The server returns a JWT access token.
6. The client sends the token in the `Authorization` header.
7. Protected routes validate the token using `JwtAuthGuard`.
8. Role-protected routes also check the user role using `RolesGuard`.

---

## License

This project is open-source and available under the MIT License.
