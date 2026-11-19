# Snitch

Snitch is an Express and MongoDB e-commerce API with authentication, seller-managed products, ImageKit image uploads, and shopping cart support.

## Tech stack

- Node.js with ES modules
- Express 5
- MongoDB with Mongoose
- JSON Web Tokens
- bcryptjs
- express-validator
- Multer
- ImageKit

## Project structure

```text
server/
  src/
    app/          Express application setup
    config/       Environment variables and MongoDB connection
    controller/   HTTP request handlers
    middlewares/  Authentication and seller authorization
    models/       Mongoose models
    routes/       API route definitions
    services/     ImageKit storage integration
    utils/        JWT helpers
    validators/   Request validation
```

## Prerequisites

- Node.js 18 or newer
- npm
- A MongoDB database
- An ImageKit private key

## Environment variables

Create `server/.env` with:

```dotenv
MONGO_URI=mongodb://127.0.0.1:27017/snitch
ACCESS_TOKEN_SECRET=replace-with-a-long-random-access-secret
REFRESH_TOKEN_SECRET=replace-with-a-long-random-refresh-secret
IMAGEKIT_PRIVATE_KEY=your-imagekit-private-key
```

Do not commit real secrets or the `.env` file.

## Installation

```powershell
cd server
npm install
```

## Running the server

```powershell
cd server
npm run dev
```

The API starts at `http://localhost:3000`.

The development script uses nodemon. There is currently no production start script.

## Authentication

Send an access token in the `Authorization` header:

```text
Authorization: Bearer <access-token>
```

Login and registration return an access token and set an HTTP-only `refreshToken` cookie. The refresh endpoint reads that cookie and rotates the refresh token.

Access tokens expire after 15 minutes. Refresh tokens expire after 7 days.

## API endpoints

### Authentication

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| POST | `/api/auth/register` | Public | Register a user |
| POST | `/api/auth/login` | Public | Log in and receive tokens |
| POST | `/api/auth/refresh` | Refresh cookie | Rotate tokens |
| GET | `/api/auth/me` | Authenticated | Get the current user |

Register request:

```json
{
  "email": "user@example.com",
  "name": "Example User",
  "password": "secret123"
}
```

Login request:

```json
{
  "email": "user@example.com",
  "password": "secret123"
}
```

### Products

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| POST | `/api/products` | Seller | Create a product |
| GET | `/api/products` | Authenticated | List published products |
| GET | `/api/products/seller` | Seller | List all seller products |
| PATCH | `/api/products/unlist/:id` | Seller | Unlist a product |

Create a product using `multipart/form-data`. Send up to five image files in the `images` field, plus these text fields:

```text
title=Classic T-shirt
description=A comfortable everyday T-shirt made for regular wear.
price={"amount":999,"currency":"INR"}
sizes=[{"size":"M","stock":20},{"size":"XL","stock":40}]
```

Example:

```powershell
curl.exe -X POST "http://localhost:3000/api/products" `
  -H "Authorization: Bearer <access-token>" `
  -F "title=Classic T-shirt" `
  -F "description=A comfortable everyday T-shirt made for regular wear." `
  -F 'price={"amount":999,"currency":"INR"}' `
  -F 'sizes=[{"size":"M","stock":20},{"size":"XL","stock":40}]' `
  -F "images=@C:\path\to\front.jpg"
```

Supported product sizes are `XS`, `S`, `M`, `L`, `XL`, and `XXL`. Product currencies are `INR` and `USD`.

### Cart

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| POST | `/api/cart` | Authenticated | Add a product or increase its quantity |
| GET | `/api/cart` | Authenticated | Intended to retrieve the user's cart |

Cart request:

```json
{
  "productId": "64c123456789abcdef123456",
  "quantity": 2,
  "size": "M"
}
```

## Validation and response format

Request validation errors use HTTP `400` responses and include an `errors` array. Authentication and authorization failures use HTTP `401` or `403`.

Successful responses generally follow this shape:

```json
{
  "message": "Operation completed successfully",
  "data": {}
}
```

## Known implementation notes

- The cart GET route is registered but does not currently attach the `getCart` handler.
- The product route intended to publish a product uses the same `/unlist/:id` route path and therefore conflicts with the unlist route; add a distinct route such as `/publish/:id` before using that operation.
- MongoDB and ImageKit credentials must be valid before starting the server.
