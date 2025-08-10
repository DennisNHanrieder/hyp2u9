# Slim Framework App with Database & Twig Integration

## Overview  
**Slim Framework App with Database & Twig Integration** is a PHP web application built on the **Slim micro-framework**.  
Starting from the `slim/slim-skeleton` template, it adds **database connectivity** and **Twig templating**, for a small and structured foundation for building modern PHP applications.

## Why this project exists  
This project was created to:  
- Learn and demonstrate **Slim Framework** fundamentals.  
- Integrate **database access** into a Slim-based app.  
- Use **Twig** for clean and maintainable front-end rendering.  

## Features  
- **Slim Framework** — Lightweight routing and middleware handling.  
- **Database Access** — Connects to and queries a MySQL/MariaDB database.  
- **Twig Templating** — Separates presentation from business logic.  
- **Clean Project Structure** — Organized controllers, templates, and configuration.  
- **Docker Ready** — Prepped for containerized deployment.

## Technologies used  
- **Backend:** PHP 8+  
- **Framework:** Slim Framework  
- **Templating:** Twig  
- **Database:** MySQL/MariaDB  
- **Environment:** Docker

## How to run the project  
```bash
# 1) Clone the repository into the Docker container's webapp directory
git clone <https://github.com/DennisNHanrieder/hyp2u9.git> webapp/t1-ue09-YourTeamName

# 2) Install dependencies
composer install

# 3) Start the Slim app (inside Docker or PHP local server)
php -S localhost:8080 -t public

# 4) Open in browser
http://localhost:8080
```

## Dependencies & requirements  
- PHP 8+  
- Composer  
- MySQL/MariaDB  
- Docker (optional but recommended)  

## How to contribute  
1. Fork the repository and create a feature branch.  
2. Add new routes in the `routes/` folder.  
3. Create corresponding Twig templates for new views.  
4. Submit a pull request with a clear description.

## What powers the core functionality?  
- **Slim Framework** — Handles routing and middleware.  
- **PDO** — Manages database connections and queries.  
- **Twig** — Renders HTML views from templates.
