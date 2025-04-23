# TypeScript REST API with Node.js, Express & MongoDB

This guide helps you set up a basic REST API using TypeScript, Express, and MongoDB.

---

## 1. Initialize Your Project

Create a new directory for your project and navigate into it:

```bash
mkdir ts-rest-api
cd ts-rest-api
```

Initialize your project with npm (it will create a `package.json`):

```bash
npm init -y
```

---

## 2. Install Dependencies

Install the necessary dependencies:

```bash
npm install express mongoose dotenv
```

Install development dependencies for TypeScript and the necessary type definitions:

```bash
npm install -D typescript ts-node-dev @types/express @types/node
```

---

## 3. Configure TypeScript

Initialize TypeScript configuration:

```bash
npx tsc --init
```

Open `tsconfig.json` and update it to:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "CommonJS",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true
  }
}
```

---

## 4. Create the Project Structure

```bash
mkdir -p src/{models,routes,controllers,config}
touch src/app.ts src/server.ts src/models/user.model.ts src/controllers/user.controller.ts src/routes/user.routes.ts src/config/db.ts .env
```

---

## 5. MongoDB Configuration (`src/config/db.ts`)

```ts
import mongoose from "mongoose";
import dotenv from "dotenv";

dotenv.config();

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGO_URI || "");
    console.log("MongoDB connected");
  } catch (err) {
    console.error("MongoDB connection failed:", err);
    process.exit(1);
  }
};

export default connectDB;
```

---

## 6. Create User Model (`src/models/user.model.ts`)

```ts
import mongoose, { Schema, Document } from "mongoose";

export interface IUser extends Document {
  name: string;
  email: string;
}

const UserSchema: Schema = new Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
});

export default mongoose.model<IUser>("User", UserSchema);
```

---

## 7. User Controller (`src/controllers/user.controller.ts`)

```ts
import { Request, Response } from "express";
import User from "../models/user.model";

export const getUsers = async (_: Request, res: Response) => {
  const users = await User.find();
  res.json(users);
};

export const createUser = async (req: Request, res: Response) => {
  const { name, email } = req.body;
  try {
    const user = new User({ name, email });
    await user.save();
    res.status(201).json(user);
  } catch (err) {
    res.status(400).json({ error: "User creation failed", details: err });
  }
};
```

---

## 8. User Routes (`src/routes/user.routes.ts`)

```ts
import express from "express";
import { getUsers, createUser } from "../controllers/user.controller";

const router = express.Router();

router.get("/", getUsers);
router.post("/", createUser);

export default router;
```

---

## 9. Express App Setup (`src/app.ts`)

```ts
import express from "express";
import userRoutes from "./routes/user.routes";

const app = express();

app.use(express.json());
app.use("/api/users", userRoutes);

export default app;
```

---

## 10. Server Setup (`src/server.ts`)

```ts
import app from "./app";
import connectDB from "./config/db";
import dotenv from "dotenv";

dotenv.config();
const PORT = process.env.PORT || 5000;

connectDB();

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

---

## 11. Environment File (`.env`)

```
MONGO_URI=mongodb://localhost:27017/ts-rest-api
PORT=5000
```

---

## 12. Scripts in `package.json`

```json
"scripts": {
  "dev": "ts-node-dev --respawn --transpile-only src/server.ts",
  "build": "tsc",
  "start": "node dist/server.js"
}
```

---

## 13. Running the Project

Make sure MongoDB is running:

```bash
mongod
```

Run the server in development mode:

```bash
npm run dev
```

Or use Docker:

```bash
docker run --name mongodb -d -p 27017:27017 mongo:6.0
ps aux | grep mongod
docker logs mongodb
docker exec -it mongodb mongo
```

---

## 14. Testing with CURL

Create a user:

```bash
curl -X POST http://localhost:5000/api/users \
-H "Content-Type: application/json" \
-d '{"name": "Alice", "email": "alice@example.com"}'
```

Get all users:

```bash
curl http://localhost:5000/api/users
```

---

## Summary

- `GET /api/users`: Lists all users.
- `POST /api/users`: Creates a new user.

---

## Optional: Install MongoDB on Ubuntu

Import MongoDB public key:

```bash
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-archive-keyring.gpg
```

Add MongoDB repository:

```bash
echo "deb [signed-by=/usr/share/keyrings/mongodb-archive-keyring.gpg] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
```

Update package list:

```bash
sudo apt update
```

Install MongoDB:

```bash
sudo apt install -y mongodb-org
```

Start MongoDB:

```bash
sudo systemctl start mongod
```

Enable MongoDB to start on boot:

```bash
sudo systemctl enable mongod
```

Verify Installation:

```bash
mongod --version
```
