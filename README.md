# Offline Setup Guide – No Docker, Simple File Copy

This guide walks you through setting up the Forms application on an offline Windows PC using simple file copying. **No Docker required.**

---

## 📋 Prerequisites

Install these on the offline PC **before starting**:

### 1. PostgreSQL (Database)
- **Download**: https://www.postgresql.org/download/windows/
- **Version**: 14 or newer
- **Setup**:
  1. Run the installer
  2. Remember the password you set for the `postgres` user (default: `postgres`)
  3. Keep port as default `5432`
  4. Complete installation
  5. Verify PostgreSQL is running by opening PowerShell and testing:
     ```powershell
     psql -U postgres -c "SELECT version();"
     ```
  If successful, you'll see PostgreSQL version info.

### 2. Node.js + npm
- **Download**: https://nodejs.org/
- **Version**: 18 LTS or newer
- **Setup**:
  1. Run the `.msi` installer
  2. Accept defaults (ensure "Add to PATH" is checked)
  3. Complete installation
  4. Verify in PowerShell:
     ```powershell
     node --version
     npm --version
     ```

---

## 📁 Step 1: Copy Project Files

Copy the entire `22-2-FORMS` folder to your offline PC at a convenient location:

```
C:\Users\YourUsername\Projects\22-2-FORMS\
```

Or anywhere you prefer (just remember the path).

**The folder structure must remain exactly the same:**
```
22-2-FORMS/
├── client/
│   ├── src/
│   ├── package.json
│   ├── vite.config.js
│   └── ...
├── server/
│   ├── db/
│   │   ├── migrate.js
│   │   ├── pool.js
│   │   ├── seed.js
│   ├── routes/
│   ├── middleware/
│   ├── package.json
│   ├── index.js
│   └── ...
├── .env.example
├── docker-compose.yml
└── ...
```

---

## 🔧 Step 2: Create Configuration File

1. Open the root folder (`22-2-FORMS\`)
2. Create a new file named `.env` (not `.env.example`)
3. Paste this content:

```
PORT=5000
DB_HOST=localhost
DB_PORT=5432
DB_NAME=formbuilder
DB_USER=postgres
DB_PASSWORD=postgres
JWT_SECRET=form-dashboard-jwt-secret-2026
```

> **If you set a different PostgreSQL password during installation**, replace the `DB_PASSWORD` value with your actual password.

---

## 💾 Step 3: Install Dependencies (if not already copied)

**Skip this step if you've already copied the `node_modules` folders from client and server.**

If you didn't copy `node_modules`, open **PowerShell** in the project root folder and run:

### Install Server Dependencies
```powershell
cd server
npm install
cd ..
```

### Install Client Dependencies
```powershell
cd client
npm install
cd ..
```

This downloads all required packages (React, Express, PostgreSQL driver, etc.) to `node_modules/` in each folder.

---

**Note**: `node_modules` can be large (500MB+) but if already copied, it will save significant time and bandwidth.

---

## 🗄️ Step 4: Set Up Database

This step has **two parts**: creating database structure (migrations) and populating it with data (seeding).

### What is Step 4 Doing?

Think of it like **building and stocking a filing cabinet**:
- **Migration** = Building empty drawers (tables with columns)
- **Seeding** = Filling drawers with starter documents (data)

#### The 7 Tables Being Created:

| Table | Purpose |
|-------|---------|
| **users** | User accounts (username, password hash, role) |
| **forms** | Form definitions (name, description, lock status) |
| **form_versions** | Different versions of each form |
| **form_fields** | Individual form fields (text, dropdown, email, etc.) |
| **submissions** | User form submissions |
| **submission_values** | Actual answers from submissions |
| **universities** | University names for autocomplete feature |

---

### Part A: Create Database Tables (Migrations)

Still in the project root, navigate to the server folder and run:

```powershell
cd server
npm run migrate
```

This creates all 7 empty tables with proper columns and data types.

**Expected output:**
```
Running migrations...
  ✓ pg_trgm extension enabled     ← Search optimization for universities
  ✓ users table
  ✓ forms table
  ✓ form_versions table
  ✓ form_fields table
  ✓ submissions table
  ✓ submission_values table
  ✓ universities table
```

**Important**: Migrations are safe to run multiple times (they use `IF NOT EXISTS`).

---

### Part B: Populate with Sample Data (Seeding)

```powershell
npm run seed
```

This fills the tables with initial data:
- **25 sample Indian universities** (for autocomplete)
- **Demo admin user** (username: `admin`, password: `admin@123`)
- **Search index** for fast university lookups

**Expected output:**
```
Seeding universities...
  ⚠ universities.xlsx not found (this is normal)
  Using sample development data...
  25 unique universities after dedup
```

---

### Optional: Use Your Own University List

If you have a custom `universities.xlsx` file:

1. Save your Excel file to: `server/data/universities.xlsx`
2. Format: Must have columns in this order:
   ```
   Column A: Name (e.g., "Harvard University")
   Column B: State (e.g., "Massachusetts")
   Column C: District (e.g., "Cambridge")
   ```
3. Run `npm run seed` – it will automatically read your file instead of sample data

Example:
```
universities.xlsx
├─ Name                          | State          | District
├─ Harvard University            | Massachusetts  | Cambridge
├─ Stanford University           | California     | Palo Alto
├─ MIT                          | Massachusetts  | Cambridge
```

---

### How It Works:

```
Your PowerShell (Step 4)
       ↓
PostgreSQL Server (localhost:5432)
       ↓
Migration Script Creates 7 Empty Tables
       ↓
Seeding Script Fills Tables with Data
       ↓
✅ Database Ready for Application (Step 5)
```

---

### 🐛 If Step 4 Fails:

| Error | Solution |
|-------|----------|
| `error: connect ECONNREFUSED` | PostgreSQL not running – start it first |
| `password authentication failed` | Wrong `DB_PASSWORD` in `.env` file |
| `no such file or directory` | Make sure you're in the `server/` folder |
| `XLSX file not found` | Normal – it will use sample data instead |

---

### ✅ After Step 4 Completes:

Your database is fully initialized:
- ✅ All tables exist and are ready
- ✅ Universities loaded (sample or custom)
- ✅ Admin user created – you can login with `admin`/`admin@123`
- ✅ Ready for Step 5 (running the application)

---

## ▶️ Step 5: Run the Application

### Start the Backend Server
Open a PowerShell window in the project root:
```powershell
cd server
npm start
```

You should see:
```
Server running on http://localhost:5000
```

### Start the Frontend Development Server
Open a **new PowerShell window** in the project root:
```powershell
cd client
npm run dev
```

You should see:
```
  VITE v7.3.1  ready in 150 ms

  ➜  Local:   http://localhost:5173/
  ➜  press h + enter to show help
```

---

## 🌐 Access the Application

1. Open your browser and go to: **http://localhost:5173**
2. You'll see the login page
3. Default credentials:
   - **Username**: `admin`
   - **Password**: `admin@123`

(These are created during the seed process)

---

## 🐛 Troubleshooting

### Frontend won't load?
- Make sure both server and client are running
- Server must be at `http://localhost:5000`
- Client must be at `http://localhost:5173`
- Check that no other apps are using these ports

### Database connection error?
- Verify PostgreSQL is running:
  ```powershell
  psql -U postgres -c "SELECT version();"
  ```
- Check `.env` file has correct `DB_PASSWORD` (must match PostgreSQL password)
- Make sure `DB_HOST=localhost` and `DB_PORT=5432`

### npm install fails?
- Make sure Node.js is installed: `node --version`
- Delete `node_modules` folder and `package-lock.json`, then retry:
  ```powershell
  rm -Recurse node_modules
  rm package-lock.json
  npm install
  ```

### Port already in use?
- **Port 5000** (backend): 
  ```powershell
  netstat -ano | findstr :5000
  # To kill the process: taskkill /PID <PID> /F
  ```
- **Port 5173** (frontend):
  ```powershell
  netstat -ano | findstr :5173
  ```

### Universities not showing in autocomplete?
- Ensure `npm run seed` completed without errors
- Check that `server/data/universities.xlsx` exists if using custom data
- Restart the server after seeding

---

## 🚀 Advanced: Running on a Different PC Later

To transfer the entire setup to another offline PC:

1. **Copy the entire folder again** (same structure, including `node_modules` if available)
2. **If you didn't copy `node_modules`**, run `npm install` in both `client` and `server` folders
3. Repeat **Step 4** (migrations and seeding)
4. Repeat **Step 5** (running the app)

**Pro tip**: Including `node_modules` in the copy makes setup faster on new PCs since `npm install` is skipped.

---

## 📝 Development Mode vs Production

Currently, you're running in **development mode** with hot-reload.

**For production**, after all testing:
```powershell
cd client
npm run build
```

This creates a `dist` folder that the backend will serve automatically.

---

## 🎯 Key Folders & Files

| Path | Purpose |
|------|---------|
| `client/src/` | React frontend code |
| `server/index.js` | Express backend entry point |
| `server/routes/` | API endpoints |
| `server/db/` | Database migrations and seeding |
| `.env` | Environment configuration |

---

## ✅ Success Checklist

- [ ] PostgreSQL installed and running
- [ ] Node.js installed
- [ ] Folder copied with exact structure
- [ ] `.env` file created in root
- [ ] `npm install` completed in both `client` and `server`
- [ ] `npm run migrate` completed successfully
- [ ] `npm run seed` completed successfully
- [ ] Backend running on `localhost:5000`
- [ ] Frontend running on `localhost:5173`
- [ ] Can login with `admin`/`admin@123`

---

## 💡 Tips

- **Keep both terminals open** – one for backend, one for frontend
- If you make changes to backend code, restart the server (Ctrl+C, then `npm start` again)
- Frontend auto-reloads on code changes (Vite feature)
- Database state persists between restarts (data is in PostgreSQL)
- All API requests go from client to `http://localhost:5000/api/`

---

## 🆘 Still Having Issues?

1. Check PostgreSQL is truly running: `psql -U postgres` (you should get a prompt)
2. Check `.env` file matches your PostgreSQL password
3. Look for error messages in PowerShell windows – they usually tell you what's wrong
4. Make sure firewall isn't blocking ports 5000 or 5173
5. Restart everything: close PowerShell windows, restart PostgreSQL, start fresh

Good luck! 🎉
