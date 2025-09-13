# Deploy an Express API (PostgreSQL & Prisma) to a Linux VPS

This article provides a detailed, step-by-step guide on how to deploy an Express.js API that uses Prisma to connect to a PostgreSQL database on a Linux VPS.

## 1. Initial Server Setup & Security

Before we deploy your application, we need to get your server ready and secure.

1.  **Log in to your VPS:** Use SSH to connect to your server with the root user.

    ```bash
    ssh root@your_vps_ip_address
    ```

2.  **Update your system:** Always start with an update to ensure all packages are current.

    ```bash
    apt update && apt upgrade -y
    ```

3.  **Create a new user:** It is bad practice to run applications as the root user. Create a new user with `sudo` privileges for your deployment.

    ```bash
    adduser anurag
    usermod -aG sudo anurag
    ```

    Now, log out of `root` and log in as your new user: `ssh anurag@your_vps_ip_address`.

4.  **Set up the firewall:** Use **UFW (Uncomplicated Firewall)** to allow only necessary traffic.
    ```bash
    sudo ufw allow 22/tcp
    sudo ufw enable
    ```
    This allows SSH access. We will open other ports later.

---

## 2. Install Required Software

Now we'll install Node.js, PostgreSQL, and Git on your server.

1.  **Install Node.js with NVM:** Using **NVM (Node Version Manager)** is the recommended way to manage Node.js versions.

    ```bash
    curl -o- [https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh](https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh) | bash
    source ~/.bashrc
    nvm install --lts
    nvm use --lts
    ```

    This installs the latest LTS version of Node.js.

2.  **Install PostgreSQL and Git:**
    ```bash
    sudo apt install postgresql postgresql-contrib git -y
    ```

---

## 3. Configure PostgreSQL

Next, we'll create a dedicated user and database for your API.

1.  **Switch to the `postgres` user:** This is the administrative user for the database.

    ```bash
    sudo -i -u postgres
    ```

2.  **Launch the `psql` shell:**

    ```bash
    psql
    ```

3.  **Create the database and user:** We will use double quotes to handle case-sensitive names, which is a good practice.

    ```sql
    CREATE DATABASE "AppDB";
    CREATE USER "AppUser" WITH ENCRYPTED PASSWORD 'a_strong_password';
    GRANT ALL PRIVILEGES ON DATABASE "AppDB" TO "AppUser";
    ```

    _Replace the password with a strong, unique one._

4.  **Grant permissions on the public schema:** This is a crucial step to prevent permission errors with Prisma.

    ```sql
    \c "AppDB"
    GRANT USAGE, CREATE ON SCHEMA public TO "AppUser";
    GRANT ALL ON ALL TABLES IN SCHEMA public TO "AppUser";
    GRANT ALL ON ALL SEQUENCES IN SCHEMA public TO "AppUser";
    ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO "AppUser";
    ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON SEQUENCES TO "AppUser";
    ```

    The `\c "AppDB"` command switches you to your new database.

5.  **Exit the `psql` shell and the `postgres` user:**
    ```sql
    \q
    exit
    ```

---

## 4. Deploy Your API with Prisma

Now, we'll get your application onto the server, configure it, and run the Prisma migrations.

1.  **Clone your project from Git:**

    ```bash
    git clone [https://github.com/your_username/your_repo.git](https://github.com/your_username/your_repo.git)
    cd your_repo
    ```

2.  **Install dependencies:**

    ```bash
    npm install
    ```

3.  **Configure environment variables:** Create a `.env` file and add your database URL.

    ```bash
    nano .env
    ```

    Add the following line, replacing the values with your credentials:

    ```
    DATABASE_URL="postgresql://AppUser:a_strong_password@localhost:5432/AppDB"
    ```

    Save and close the file.

4.  **Run Prisma migrations:** This will create the necessary tables in your database.
    ```bash
    npx prisma migrate deploy
    ```

---

## 5. Run Your Application with PM2

**PM2** is a process manager that keeps your application running in the background and restarts it automatically if it crashes.

1.  **Install PM2 globally:**

    ```bash
    npm install -g pm2
    ```

2.  **Start your application with PM2:**

    ```bash
    pm2 start your_main_file.js --name "my-express-api"
    ```

    _Replace `your_main_file.js` with your application's entry point (e.g., `dist/index.js` or `app.js`)._

3.  **Enable PM2 on startup:** This command ensures your application starts whenever the server reboots. Run the command that PM2 provides in the output.

    ```bash
    pm2 startup
    ```

4.  **Save the process list:**

    ```bash
    pm2 save
    ```

5.  **Open your application's port in the firewall:**
    ```bash
    sudo ufw allow 3000/tcp
    ```
    _Replace `3000` with the port your Express.js app is listening on._

Your API should now be running and accessible at `http://your_vps_ip_address:3000`.

---
