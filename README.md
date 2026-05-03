# EcoBudget — Personal Finance Tracker

A full-stack personal finance web application built with vanilla HTML/CSS/JavaScript on the frontend and PHP + SQLite on the backend. Includes expense tracking, budgeting, recurring expenses, visual reports, and an AI financial advisor powered by OpenAI.

---

## Screenshots

| Dashboard | Expenses | Reports |
|-----------|----------|---------|
| Overview with spending summary and budget status | Filterable transaction list | Pie, bar, and trend charts |

| AI Advisor | Monthly Summary | Categories |
|------------|-----------------|------------|
| Chat-based financial advice | AI-generated end-of-month review | Custom colour-coded categories |

---

## Features

- **Dashboard** — Monthly spending hero card, today/week/month stats, recent transactions, and budget status at a glance
- **Expense Tracking** — Add, search, filter by category/month, and delete expenses; grouped by date
- **Categories** — Create custom colour-coded categories; 8 defaults seeded on registration
- **Budgets** — Set per-category monthly spending limits with visual progress bars and over-budget alerts
- **Recurring Expenses** — Manage subscriptions and regular bills; one-click generation into expenses for the current month
- **Reports** — Spending by category (donut chart), 6-month trend (bar chart), payment method breakdown (pie chart), and a detailed table
- **Monthly Summary** — Stats overview with AI-generated end-of-month analysis, pie chart, bar chart, and category table
- **AI Financial Advisor** — GPT-powered chat for saving tips, investment basics, and budget advice tailored to Botswana (BWP); falls back to curated responses if no API key is set
- **Dark Mode** — Full dark/light theme toggle, persisted via cookie
- **Collapsible Sidebar** — Minimises to icon-only view; state persisted in localStorage

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML5, CSS3 (custom design system), Vanilla JavaScript (ES2020) |
| Backend | PHP 8.0+ |
| Database | SQLite 3 (via PHP PDO) |
| Charts | Chart.js 4 |
| Icons | Lucide Icons |
| AI | OpenAI GPT-3.5-turbo (with local fallback) |
| Font | DM Sans + DM Mono (Google Fonts) |

---

## Project Structure

```
ecobudget/
├── index.php                   # Redirects to login or dashboard
├── login.php                   # Login page
├── register.php                # Registration page
├── dashboard.php               # Main overview page
├── expenses.php                # Transaction list with filters
├── add-expense.php             # Add new expense form
├── categories.php              # Manage categories
├── budgets.php                 # Set and track budgets
├── recurring.php               # Recurring expenses manager
├── reports.php                 # Visual spending reports
├── monthly-summary.php         # Monthly review with AI
├── ai-advisor.php              # AI chat advisor
│
├── api/                        # Backend REST endpoints
│   ├── login.php               # POST — authenticate user
│   ├── register.php            # POST — create account
│   ├── logout.php              # GET  — destroy session
│   ├── dashboard.php           # GET  — stats and recent data
│   ├── expenses.php            # GET / POST / PUT / DELETE
│   ├── categories.php          # GET / POST / DELETE
│   ├── budgets.php             # GET / POST / DELETE
│   ├── recurring.php           # GET / POST / PUT / DELETE + generate
│   ├── reports.php             # GET  — analytics data
│   └── ai.php                  # POST — OpenAI chat and summaries
│
├── config/
│   ├── database.php            # PDO connection, schema creation, seeding
│   └── config.php              # OpenAI key, currency, app settings
│
├── includes/
│   ├── auth_check.php          # Session guards for pages and API
│   ├── sidebar.php             # Reusable navigation partial
│   ├── page_head.php           # HTML head + sidebar open
│   └── page_foot.php           # Script tags + HTML close
│
├── assets/
│   ├── css/style.css           # Full design system with dark mode
│   └── js/app.js               # All frontend logic and API calls
│
└── data/
    └── ecobudget.db            # SQLite database (auto-created on first run)
```

---

## Requirements

- PHP 8.0 or higher
- `php-sqlite3` extension enabled
- `php-curl` extension enabled (for OpenAI API calls)
- A web server (Apache, Nginx) **or** PHP's built-in server for local development
- No Composer, no npm, no build tools needed

---

## Installation

### Option 1 — PHP Built-in Server (quickest)

```bash
# Unzip the project
unzip ecobudget.zip
cd ecobudget

# Start the server
php -S localhost:8000

# Open in browser
http://localhost:8000
```

### Option 2 — Apache / XAMPP / WAMP

1. Copy the `ecobudget/` folder into your web root:
   - **XAMPP**: `C:/xampp/htdocs/ecobudget`
   - **WAMP**: `C:/wamp64/www/ecobudget`
   - **Linux Apache**: `/var/www/html/ecobudget`

2. Ensure `mod_rewrite` is enabled (optional, not strictly required).

3. Visit `http://localhost/ecobudget/`

### Option 3 — Nginx

```nginx
server {
    listen 80;
    server_name ecobudget.local;
    root /var/www/ecobudget;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/run/php/php8.0-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

---

## Configuration

Open `config/config.php` and update the following:

```php
// Your OpenAI API key — get one at https://platform.openai.com/api-keys
define('OPENAI_API_KEY', 'sk-...');

// OpenAI model (gpt-3.5-turbo is cost-effective; use gpt-4o for better quality)
define('OPENAI_MODEL', 'gpt-3.5-turbo');

// Currency settings
define('APP_CURRENCY', 'BWP');   // ISO currency code shown in inputs
define('APP_SYMBOL',   'P');     // Symbol used in formatted amounts
```

> **No OpenAI key?** The AI Advisor will automatically fall back to curated rule-based responses. All other features work fully without an API key.

---

## Database

The SQLite database is created automatically at `data/ecobudget.db` on the very first request. No manual migration or setup is required.

**Tables created automatically:**

| Table | Description |
|---|---|
| `users` | Accounts with hashed passwords |
| `categories` | User-defined expense categories with colours |
| `expenses` | All transactions linked to a user and optional category |
| `budgets` | Monthly per-category spending limits |
| `recurring_expenses` | Subscriptions and recurring bills |
| `ai_conversations` | Reserved for future conversation persistence |

**Default categories seeded per new user:** Food & Dining, Transportation, Entertainment, Shopping, Utilities, Health, Housing, Education.

---

## API Reference

All endpoints require an active session (HTTP 401 returned otherwise). Request and response bodies are JSON.

### Expenses

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/expenses.php` | List all expenses for the logged-in user |
| `POST` | `/api/expenses.php` | Create a new expense |
| `PUT` | `/api/expenses.php` | Update an expense by `id` |
| `DELETE` | `/api/expenses.php?id=N` | Delete an expense |

**POST body:**
```json
{
  "description": "Grocery shopping",
  "amount": 250.00,
  "category_id": 1,
  "date": "2026-05-03",
  "payment_method": "cash",
  "notes": "Weekly shop"
}
```

### Budgets

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/budgets.php?month=5&year=2026` | List budgets with spending for the month |
| `POST` | `/api/budgets.php` | Set or update a budget (upserts on conflict) |
| `DELETE` | `/api/budgets.php?id=N` | Remove a budget |

### Recurring Expenses

| Method | Endpoint | Body / Params | Description |
|--------|----------|---------------|-------------|
| `GET` | `/api/recurring.php` | — | List all recurring items |
| `POST` | `/api/recurring.php` | `{ description, amount, day_of_month, ... }` | Create recurring item |
| `POST` | `/api/recurring.php` | `{ "action": "generate" }` | Generate this month's expenses from active items |
| `PUT` | `/api/recurring.php` | `{ id, is_active: 0|1 }` | Toggle active state |
| `DELETE` | `/api/recurring.php?id=N` | — | Delete recurring item |

### AI

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/ai.php` | Chat or generate a monthly summary |

**Chat body:**
```json
{
  "type": "chat",
  "messages": [
    { "role": "user", "content": "Give me 3 saving tips" }
  ]
}
```

**Monthly summary body:**
```json
{
  "type": "monthly_summary",
  "month": 5,
  "year": 2026
}
```

---

## Security Notes

- Passwords are hashed with PHP's `password_hash()` using `PASSWORD_DEFAULT` (bcrypt)
- All database queries use PDO prepared statements — no SQL injection risk
- Session IDs are regenerated on login and registration
- API endpoints verify session ownership before any read or write
- All user-supplied output is escaped with `htmlspecialchars()` before rendering
- The `data/` directory (containing the SQLite file) should be placed outside the web root in production, or protected with a `.htaccess` deny rule

**Recommended `.htaccess` for the `data/` directory:**
```apache
Deny from all
```

---

## Customisation

### Change the currency

Update `config/config.php`:
```php
define('APP_CURRENCY', 'ZAR');  // e.g. South African Rand
define('APP_SYMBOL',   'R');
```

Then update the placeholder text in `add-expense.php` and `budgets.php` input fields.

### Change the colour scheme

The primary green can be changed in `assets/css/style.css`:
```css
:root {
  --primary:       #166534;   /* dark green — sidebar active, buttons */
  --primary-hover: #15803d;
  --primary-light: #16a34a;   /* lighter green — links, badges */
  --primary-faint: #dcfce7;   /* very light — icon backgrounds */
}
```

### Add more suggested AI prompts

In `ai-advisor.php`, duplicate a `.prompt-card` block and set the `data-prompt` attribute to your desired question.

---

## Troubleshooting

**Blank page / PHP errors**
- Confirm PHP 8.0+ is installed: `php -v`
- Ensure the `php-sqlite3` extension is active: `php -m | grep sqlite`
- Check that the `data/` directory is writable: `chmod 755 data/`

**AI Advisor returns fallback responses**
- Verify your OpenAI API key in `config/config.php`
- Confirm `php-curl` is enabled: `php -m | grep curl`
- Check your OpenAI account has available credits

**Charts not rendering**
- Chart.js is loaded from jsDelivr CDN — ensure the browser has internet access
- Check the browser console for JavaScript errors

**Session issues / keeps logging out**
- Ensure `session.save_path` is writable in your `php.ini`
- On shared hosting, set a custom session path: `session_save_path('/tmp')` at the top of each page

---

## Roadmap / Future Ideas

- [ ] Income tracking and net savings calculation
- [ ] CSV / PDF export of expenses
- [ ] Multi-currency support
- [ ] Email budget alerts
- [ ] Mobile PWA support
- [ ] Goal savings tracker
- [ ] Bank statement import (OFX/CSV)

---

## License

MIT License — free to use, modify, and distribute.

---

## Credits

- Icons by [Lucide](https://lucide.dev)
- Charts by [Chart.js](https://chartjs.org)
- Font by [Google Fonts — DM Sans](https://fonts.google.com/specimen/DM+Sans)
- AI powered by [OpenAI](https://openai.com)
