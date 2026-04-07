# Deployment Guide: Smart Student Behavior Analytics

This project consists of a React/Vite frontend and a Node.js/Express backend that connects to a MySQL database. Because the backend (`server.js`) is configured to serve the frontend static files, you can deploy this as a single unified application or split them up.

Here is a step-by-step guide to deploying your application for free or low cost using services like Render, Vercel, and Cloud MySQL.

---

## Step 1: Set Up Your MySQL Database

Most cloud providers (like Render or Vercel) don't provide native MySQL databases. You will need to host your MySQL database separately.

**Recommended Free Providers:**
1. **[Aiven](https://aiven.io/)** (Offers a free tier for MySQL)
2. **[Railway](https://railway.app/)** (Easy setup with a free trial/developer tier)

**Actions:**
1. Create a MySQL database instance on your chosen provider.
2. Note down your connection credentials: **Host**, **Port**, **User**, **Password**, and **Database Name**.
3. Connect to the remote database using your local MySQL Workbench or DBeaver.
4. Execute the contents of `database_setup.sql` on the remote database to create the required tables and seed data. *(Note: Since you use Sequelize, simply starting the server will also auto-generate the base tables, but running the SQL setups your seed data).*

---

## Step 2: Unified Deployment (Render.com)

Since your `server.js` serves the frontend code (lines 43-49 serving `../frontend/dist`), the easiest and most seamless way to deploy is to use **Render** as a single Web Service. 

1. Push your latest code to a Git repository (GitHub/GitLab).
2. Log into [Render.com](https://render.com/) and create a new **Web Service**.
3. Connect your Git repository.
4. Configure the Web Service with the following details:
   - **Root Directory:** *(leave blank or set to `.`)*
   - **Environment:** `Node`
   - **Build Command:** 
     ```bash
     cd backend && npm install && npm run build
     ```
     *(This installs backend dependencies, then triggers your package.json build script to jump into the frontend, install frontend dependencies, and compile your React app)*
   - **Start Command:**
     ```bash
     cd backend && node server.js
     ```

5. **Environment Variables:** Scroll down and add the following keys:
   - `NODE_ENV` = `production`
   - `DB_HOST` = *(Your remote DB host)*
   - `DB_USER` = *(Your remote DB user)*
   - `DB_PASSWORD` = *(Your remote DB password)*
   - `DB_NAME` = *(Your remote DB name)*
   - `DB_PORT` = `3306` *(or provided port)*
   - `JWT_SECRET` = *(Generate a random secure string)*
   - `VITE_API_URL` = `/api` *(This ensures the frontend correctly communicates with the unified backend)*

6. Click **Create Web Service**. Render will build both frontend and backend and deploy them to a single URL.

---

## Alternative: Split Deployment (Backend on Render, Frontend on Vercel)

If you prefer to host your frontend statically on Vercel or Netlify for better edge performance:

### 1. Deploy the Backend (Render)
- **Root Directory:** `backend`
- **Build Command:** `npm install`
- **Start Command:** `node server.js`
- **Environment Variables:** Set the DB variables, `JWT_SECRET`, and `NODE_ENV=production`.
- Once deployed, note the Render URL (e.g., `https://your-api.onrender.com`).

### 2. Deploy the Frontend (Vercel / Netlify)
- Create a new project pointing to your Github repo.
- **Root Directory:** `frontend`
- **Build Command:** `npm run build`
- **Output Directory:** `dist`
- **Environment Variable:** Add the exact Render backend URL you received in step 1:
  - `VITE_API_URL` = `https://your-api.onrender.com/api`

---

> [!TIP]
> **CORS Warning:** If you use the Split Deployment method, ensure your Render backend accepts cross-origin requests from your Vercel URL. Your current `cors()` middleware in `server.js` accepts all connections by default, but you may want to lock it down for production security.
> 
> **For the Unified Deployment:** CORS is not an issue since both frontend and backend live on the exact same domain.
