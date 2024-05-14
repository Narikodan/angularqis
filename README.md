# XPB_API2.0

XPB_API2.0 is a FastAPI project designed to handle various endpoints related Loyalty xchange and business suite. This guide will help you set up and run the project on your machine.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Database Setup](#database-setup)
- [Running the Application](#running-the-application)
- [Cron Jobs](#cron-jobs)
- [Service Management](#service-management)
- [Log Rotation](#log-rotation)
- [Project Structure](#project-structure)
- [Endpoints](#endpoints)
  - [User Endpoints](#user-endpoints)
  - [Admin Endpoints](#admin-endpoints)
  - [Merchant Endpoints](#merchant-endpoints)

## Prerequisites

- Python 3.10
- PostgreSQL
- MongoDB

## Installation

1. **Clone the repository:**
    ```bash
    git clone https://github.com/Code-XPB/XPB_API2.0.git
    ```

2. **Navigate to the project directory:**
    ```bash
    cd XPB_API2.0
    ```

3. **Install Python 3.10 and necessary development packages:**
    ```bash
    sudo apt install python3.10 python3.10-dev python3.10-venv
    ```

4. **Create a virtual environment:**
    ```bash
    python3.10 -m venv ~/path_to_venv/venv_name
    ```

    Replace `~/path_to_venv/venv_name` with the desired path and name for your virtual environment.

5. **Activate the virtual environment:**
    ```bash
    source ~/path_to_venv/venv_name/bin/activate
    ```

    Once activated, your shell prompt will change to indicate that you are now working inside the virtual environment.

6. **Upgrade `pip` and install the required Python packages:**
    ```bash
    pip install --upgrade pip
    pip install -r requirements/stage.txt
    ```

    This will install all the packages listed in the `requirements/stage.txt` file.

## Configuration

1. **Copy and configure `hash_policy.ini`:**
    ```bash
    cp hash_policy.ini.example hash_policy.ini
    ```

    Update the `hash_policy.ini` file:
    ```ini
    [bcrypt]
    default_rounds = 12  # You can set this between 12 and 15
    ```

2. **Copy and configure `logging.conf`:**
    ```bash
    cp logging.conf.example logging.conf
    mkdir logs
    ```

3. **Copy and configure `alembic.ini`:**
    ```bash
    cp alembic.ini.example alembic.ini
    ```

    Update the `alembic.ini` file with your database URL:
    ```ini
    sqlalchemy.url = postgresql://user:password@localhost/dbname
    ```

4. **Place necessary JSON files in the project root:**
    - `apple-app-site-association`
    - `firebase.json`

5. **Add M2P key files:**
    ```bash
    mkdir -p core/api/m2p/keys/
    # Place public and private keys in the above directory
    ```

    Update the paths in the `.env` file to point to these keys.

6. **Configure environment variables:**
    ```bash
    cp .env.example .env
    ```

    Update the values in the `.env` file with your specific configurations.

## Database Setup

1. **Initialize the database schema:**
    ```bash
    alembic upgrade head
    ```

2. **Load initial data:**
    ```bash
    mkdir data_csv
    # Copy required CSV files to the data_csv folder
    psql -U <username> -d <database_name> -h <host> -p <port> -f load_data.sql
    ```

    Example `load_data.sql` content:
    ```sql
    \copy roles FROM 'data_csv/roles.csv' DELIMITER ',' CSV HEADER;
    \copy country FROM 'data_csv/country.csv' DELIMITER ',' CSV HEADER;
    # Repeat for other tables as necessary
    ```

## Running the Application

1. **Create a systemd service for the FastAPI application:**
    ```bash
    sudo nano /etc/systemd/system/xpayback.service
    ```

    Add the following content:
    ```ini
    [Unit]
    Description=Gunicorn Daemon for FastAPI Xpayback Application
    After=network.target

    [Service]
    Restart=on-failure
    RestartSec=5s
    User=<SYSTEM_USER>
    Group=www-data
    WorkingDirectory=/path_to_project/XPB_API2.0
    ExecStart=/path_to_venv/venv_name/bin/gunicorn -c gunicorn_conf.py main:app

    [Install]
    WantedBy=multi-user.target
    ```

2. **Start and enable the service:**
    ```bash
    sudo systemctl start xpayback
    sudo systemctl enable xpayback
    sudo systemctl status xpayback
    ```

## Cron Jobs

Add the following cron jobs:
```bash
# Example cron jobs
30 0 * * * cd /path_to_project/XPB_API2.0/cron_jobs/ && /path_to_venv/venv_name/bin/python xoxoday_cron.py >> /path_to_project/XPB_API2.0/logs/xoxoday_cron.log 2>&1
# Repeat for other cron jobs as necessary
