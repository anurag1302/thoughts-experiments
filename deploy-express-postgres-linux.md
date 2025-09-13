# Deploy an Express API (PostgreSQL & Prisma) to a Linux VPS

This write-up provides a detailed, step-by-step guide on how we can deploy our Express.js API that uses Prisma to connect to a PostgreSQL database on a Linux VPS.

---

## 1. Initial Server Setup & Security

Before we deploy our application, we need to get our server ready and secure.

1.  **Log in to our VPS:** We'll use SSH to connect to our server with the root user.

    ```bash
    ssh root@our_vps_ip_address
    ```

2.  **Update our system:** We should always start with an update to ensure all packages are current.

    ```bash
    apt update && apt upgrade -y
    ```

3.  **Create a new user:** It's not a good practice to run applications as the root user. We will create a new user with `sudo` privileges for our deployment.

    ```bash
    adduser anurag
    usermod -aG sudo anurag
    ```

    Now, let's log out of `root` and log in as our new user: `ssh anurag@our_vps_ip_address`.

4.  **Set up the firewall:** We'll use **UFW (Uncomplicated Firewall)** to allow only necessary traffic.
    ```bash
    sudo ufw allow 22/tcp
    sudo ufw enable
    ```
    This allows SSH access. We will open other ports later.

---

## 2. Install Required Software

Now we'll install Node.js, PostgreSQL, and Git on our server.

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

Next, we'll create a dedicated user and database for our API.

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

    _Let's be sure to replace the password with a strong, unique one._

4.  **Grant permissions on the public schema:** This is a crucial step to prevent permission errors with Prisma.

    ```sql
    \c "AppDB"
    GRANT USAGE, CREATE ON SCHEMA public TO "AppUser";
    GRANT ALL ON ALL TABLES IN SCHEMA public TO "AppUser";
    GRANT ALL ON ALL SEQUENCES IN SCHEMA public TO "AppUser";
    ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO "AppUser";
    ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON SEQUENCES TO "AppUser";
    ```

    The `\c "AppDB"` command switches us to our new database.

5.  **Exit the `psql` shell and the `postgres` user:**
    ```sql
    \q
    exit
    ```

---

## 4. Deploy Our API with Prisma

Now, we'll get our application onto the server, configure it, and run the Prisma migrations.

1.  **Clone our project from Git:**

    ```bash
    git clone [https://github.com/our_username/our_repo.git](https://github.com/our_username/our_repo.git)
    cd our_repo
    ```

2.  **Install dependencies:**

    ```bash
    npm install
    ```

3.  **Configure environment variables:** We'll create a `.env` file and add our database URL.

    ```bash
    nano .env
    ```

    Add the following line, replacing the values with our credentials:

    ```
    DATABASE_URL="postgresql://AppUser:a_strong_password@localhost:5432/AppDB"
    ```

    Save and close the file.

4.  **Run Prisma migrations:** This will create the necessary tables in our database.
    ```bash
    npx prisma migrate dev
    npx prisma migrate deploy
    ```

---

## 5. Run Our Application with PM2

**PM2** is a process manager that keeps our application running in the background and restarts it automatically if it crashes.

1.  **Install PM2 globally:**

    ```bash
    npm install -g pm2
    ```

2.  **Start our application with PM2:**

    ```bash
    pm2 start our_main_file.js --name "my-express-api"
    ```

    _We'll replace `our_main_file.js` with our application's entry point (e.g., `src/index.js` or `app.js`)._

3.  **Enable PM2 on startup:** This command ensures our application starts whenever the server reboots. We will run the command that PM2 provides in the output.

    ```bash
    pm2 startup
    ```

4.  **Save the process list:**

    ```bash
    pm2 save
    ```

5.  **Open our application's port in the firewall:**
    ```bash
    sudo ufw allow 3000/tcp
    ```
    _We'll replace `3000` with the port our Express.js app is listening on._

Our API should now be running and accessible at `http://our_vps_ip_address:3000`.
