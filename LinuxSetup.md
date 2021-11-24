# Zeppelin Setup Guide By @Lando Calrissian#0001
![Zeppelin Banner](assets/zeppelinbanner.jpg)

**Consider putting a ⭐️ on this repo if you find it helpful!**

**Zeppelin self hosting. Its not too easy, but its also not impossible**

> THIS WILL BE GETTING REGULAR UPDATES, SO PLEASE BE PATIENT IF SOMETHING IS NOT RIGHT!

**Linux users, this is the guide for you!**
**if you are unsure on certain things, please send a messsage to the zeppelin support server (discord.gg/zeppelin)**

https://github.com/Dragory/ZeppelinBot/pull/232/files

## Check for Updates
`sudo apt update && sudo apt -y upgrade` 

## Install Necessary Software

1. `sudo apt -y install mariadb-server git nano curl build-essential nginx`
    - Mariadb for database (not MySQL)
    - Git for copying the bot code and future updates
    - Nano for editing text files
    - Curl for downloading certain installation scripts
    - build-essential is needed for building the bot
    - Nginx to serve web files for the dashboard, which is where the config is edited/built

It will probably say that some of the things are already installed, which is fine. Just make sure there are no errors.

2. Install NVM (Node Version Manager). Instead of installing Node directly (and run the high risk of installing the wrong version), NVM is used because the code contains a setting that tells NVM which node version to use. This reduces any chance of complications down the road.
  1. `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash`
  2. Log out and log back in. If you're connected by SSH, run `exit` and reconnect.

## Copy the Current Bot Code
`git clone https://github.com/Dragory/ZeppelinBot.git`

This creates a folder called ZeppelinBot and puts the current code in there.

## Install NodeJS
Now we need to get NVM to read the Node version and install the correct Node version to use.

1. `cd ZeppelinBot`
2. `nvm install`

## Set up Github SSH Access
Building the bot will access various Github repositories and download code from different places, code that the bot depends on. In order to do that, we need to set up a Github account if you don't have one yet, and set up SSH access via a key pair so your server can access these Github repositories as needed during the installation process.

1. `ssh-keygen -t ed25519 -C "your_email@example.com"` (substituting your email address in the quotes; keep the quotes)
2. `eval "$(ssh-agent)"`
3. `ssh-add ~/.ssh/id_ed25519`
4. `cat ~/.ssh/id_ed25519.pub` Copy all the text from the output
5. Install the key pair to Github:
  1. Log in to github (create an account if you need to)
  2. Click on profile picture on top right and click on Settings
  3. On the left, click on SSH and GPG Keys, then click New SSH Key
  4. Paste text (from step 4) into Key box and name it. Then click the green save button.
6. Back in the SSH shell, `ssh -T git@github.com`

## Install and Build the Bot and API

### Initial Installation

1. `cd backend`
2. `npm ci`
    - Make sure there are no errors. If there are errors, copy the entire output (not just the ERR parts) and ask in the Zeppelin Discord server.
3. `cd ..`
4. `cp .env.example .env`
5. `nano .env`
    - Fill in 32-char key, letters and numbers only.
    - You can use a password generator set to letters and numbers only to help you (https://passwordsgenerator.net/?length=32&symbols=0&numbers=1&lowercase=1&uppercase=1&similar=0&ambiguous=0&client=1&autoselect=0)
    - Press **Ctrl-X**, then **Y**, then **Enter** to save the file and close Nano.
6. `cd backend`
7. `cp bot.env.example bot.env`
8. `cp api.env.example api.env`

We'll fill in these env files later. First we need to set up the database

### Initial Database Setup

1. `sudo mariadb`
    - If you have a root passsword set, it won't let you in. Run `sudo mariadb -p` instead and type in the password when asked. It won't display, so make sure you type it correctly then just press enter.
2. Check what version of Mariadb is running. **If it is 10.2 or below, you need to upgrade Mariadb before you can continue**
    - `exit;`
    - `sudo apt remove mariadb-server`
    - `curl -LsS -O https://downloads.mariadb.com/MariaDB/mariadb_repo_setup`
    - `sudo bash mariadb_repo_setup --mariadb-server-version=10.6`
    - `sudo apt update`
    - `sudo apt install mariadb-server`
    - `sudo mariadb` or `sudo mariadb -p`
4. `CREATE USER 'zeppelin'@'localhost' IDENTIFIED BY 'ENTER_DATABASE_PASSWORD';`
    - Replace **ENTER_DATABASE_PASSWORD** with a password of your choosing, but keep the single quotes.
5. `grant all on *.* to 'root'@'localhost' identified by 'ENTER_YOUR_ROOT_PASSWORD_HERE' with grant option;`
    - Replace **ENTER_YOUR_ROOT_PASSWORD_HERE** with your root MySQL password, or leave it blank if you don't have one. Either way, make sure to keep the single quotes.
6. `create database zeppelin;`
7. `grant all on zeppelin.* to 'zeppelin'@'localhost' identified by 'ENTER_DATABASE_PASSWORD' with grant option;`
    - Replace **ENTER_DATABASE_PASSWORD** with the same password you chose in step 2, but keep the single quotes.
8. `flush privileges;`
9. `exit;`

### Setting up the Bot
1. In your browser, go to https://discord.com/developers/applications/ and log in as needed.
2. Click on the button **Create new application** and name it.
3. On the left, click on **Oauth2**
    - Under **Redirects**, put `http://localhost:8800/auth/oauth-callback` into the text box.
        - If there is no text box, click **Add Another**
        - If the bot will be used in a production environment, or you will otherwise be accessing the bot/dashboard from outside the server computer, put `http://[domain|ip]:8800/auth/oauth-callback)` into the box intead. (e.g. http://8.8.8.8:8800/auth/oauth-callback or http://google.com:8800/auth/oauth-callback)
        - Make sure there is no slash at the end (e.g. /oauth-callback not /oauth-callback/)
    - Copy **Client ID** and **Client Secret**
    - On the bottom, click the green **Save** button.
4. On the left, click on **Bot** and add a bot.
    - Copy the token
    - Under **Privileged Gateway Intents**, enable all 3 toggles.
5. Invite the bot to the server but do not try to run any commands.
    - https://discord.com/api/oauth2/authorize?client_id=CLIENT-ID-HERE&permissions=8&scope=bot
    - Replace **CLIENT-ID-HERE** with the client ID you copied in step 3.

### Fill in the Bot and API settings
1. `nano bot.env`
    - TOKEN= *(fill in the token from step 4 under Setting up the Bot)*
    - DB_HOST=localhost
    - DB_USER=zeppelin
    - DB_PASSWORD= *(fill in the password you set in step 2 under Initial Database Setup)*
    - DB_DATABASE=zeppelin
2. Press **Ctrl-X**, then **Y**, then **Enter** to save the file and close Nano.
3. `nano api.env`
    - PORT=8800
    - CLIENT_ID= *(fill in the token from step 3 under Setting up the Bot)*
    - CLIENT_SECRET= *(fill in the token from step 3 under Setting up the Bot)*
    - OAUTH_CALLBACK_URL= *(put in the same URL you did in the Discord Application settings, step 3 under Setting up the Bot)*
    - DASHBOARD_URL=http://localhost:1234
      - Use the same domain/IP as you did for OAUTH_CALLBACK_URL
      - Make sure there is no slash at the end
    - DB_HOST=localhost
    - DB_USER=zeppelin
    - DB_PASSWORD= *(fill in the password you set in step 2 under Initial Database Setup)*
    - DB_DATABASE=zeppelin
    - STAFF= *(put your Discord User ID here; attach any additional bot adminstrators' Discord user IDs separated by commas)*
4. Press **Ctrl-X**, then **Y**, then **Enter** to save the file and close Nano.

### Build the Bot and API

1. `npm run build`
    - Make sure there are no errors. If there are errors, copy the entire output (not just the ERR parts) and ask in the Zeppelin Discord server.
2. Run migrations. This will actually set up the database structure:
    - For a development instance (for testing and development): `npm run migrate-dev`
    - For a production instance (for use on actual servers): `npm run migrate-prod`

### Set up Initial Database Entries

Initial configurations and entries in the database need to be set up to use the bot on the first server:

1. `sudo mariadb`
    - or `sudo mariadb -p` as applicable
2. `use zeppelin;`
3. `INSERT INTO allowed_guilds (id, name, icon, owner_id) VALUES ("SERVER_ID", "SERVER_NAME", null, "OWNER_ID");`
    - Modify SERVER_ID, SERVER_NAME, OWNER_ID
4. `INSERT INTO configs (id, `key`, config, is_active, edited_by) VALUES (1, "global", "{\"prefix\": \"!\", \"url\": \"http://localhost:8800\" ,\"owners\": [\"YOUR_ID\"]}", true, "YOUR_ID");`
    - Modify YOUR_ID X2; replace localhost with domain|ip as applicable
5. `INSERT INTO configs (id, `key`, config, is_active, edited_by) VALUES (2, "guild-GUILD_ID", "{\"prefix\": \"!\", \"levels\": {\"YOUR_ID\": 100}, \"plugins\": { \"utility\": {}}}", true, "YOUR_ID");`
    - Modify GUILD_ID, YOUR_ID X2
6. `INSERT INTO api_permissions (guild_id, target_id, type, permissions) VALUES (GUILD_ID, YOUR_ID, "USER", "OWNER");`
    - Modify GUILD_ID, YOUR_ID
7. `SET GLOBAL time_zone = '+0:00';`
8. `exit;`

### Start Bot and API

#### In Production

For production use (most cases), use pm2 to manage your bot instances. Zeppelin already coems with "process files" for pm2, which are files that contain instructions telling pm2 how to start the bot and api.

1. `npm i -g pm2`
2. `cd ..`
3. `pm2 start process-bot.json`
4. `pm2 start process-api.json`

#### In Development

To start the bot in development, run `npm run watch`. This will start build and start both the bot and api, and will also check for file changes and update the bot automatically.

## Install and Build the Dashboard

1. cd to the dashboard folder:
    - If you are in the backend folder: `cd ../dashboard`
    - If you are in the ZeppelinBot folder: `cd dashboard`
2. `npm ci`
3. `cp .env.example .env`
4. `nano .env`
    - API_URL=http://localhost:8800
      - As before, replace localhost with a domain or IP if you did so in the api.env file.
      - As before, make sure there is no slash at the end.
5. If you are setting up a production bot: `npm run build`
6. If you are setting up a development bot: `npm run watch`
    - This will build and set up a temporary webserver that hosts the dashboard.

## Set up Nginx for Production Bots

1. `sudo nano /etc/nginx/sites-available/zeppelin`
2. Copy the following:
```
server {
    listen 1234; # replace with dashboard port
    listen [::]:1234; # replace with dashboard port

    server_name zeppelin; #or domain on a live server

    root /home/ubuntu/ZeppelinBot/dashboard/dist; #replace ubuntu with actual account name
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
````
3. In Nano, **right click** to paste the text. Then press **Ctrl-X** then **Y** then **Enter** to save and close the file.
4. `sudo ln -s /etc/nginx/sites-available/zeppelin /etc/nginx/sites-enabled/zeppelin`
5. `sudo systemctl restart nginx`
    - Make sure there are no errors. If there are, run `systemctl journal nginx.service` (or whatever command it tells you to run) to view the error log and ask in the Zeppelin Discord server.

That's it! The bot should be fully functional. The dashboard should be accessible at http://[localhost|domain|ip]:1234. If there are any issues, or to see sample configs, please visit the Zeppelin Discord Server
