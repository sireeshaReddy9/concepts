# Django — Technical Paper

---

## 1. Settings File

### Secret Key

The `SECRET_KEY` in `settings.py` is a long random string Django uses internally for cryptographic signing. It is used to sign session cookies, CSRF tokens, password reset links, and any other data that Django needs to verify hasn't been tampered with.

**If someone gets your secret key, they can:**
- Forge session cookies and log in as any user
- Generate valid CSRF tokens and bypass protection
- Forge password reset links

**Rule:** Never commit the secret key to version control. In production, load it from an environment variable.

```python
import os
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')
```

---

### Default Installed Apps

When you create a new Django project, `INSTALLED_APPS` comes with these by default:

| App | What it does |
|---|---|
| `django.contrib.admin` | The admin panel at `/admin/` |
| `django.contrib.auth` | User authentication — login, logout, permissions |
| `django.contrib.contenttypes` | Tracks all models across all apps, used by auth |
| `django.contrib.sessions` | Manages user sessions |
| `django.contrib.messages` | One-time flash messages (e.g. "Login successful") |
| `django.contrib.staticfiles` | Serves CSS, JS, images during development |

**We can also add:**
- `django.contrib.postgres` — PostgreSQL-specific fields and lookups
- `rest_framework` — Django REST Framework for building APIs
- `corsheaders` — Handle Cross-Origin requests
- Any app you create yourself, like `ipl`

---

### Middleware

Middleware is code that runs on **every request and every response**, before it reaches your view and after your view returns a response. Think of it as a pipeline — the request passes through each middleware layer going in, and back through them going out.

```
Request → Middleware 1 → Middleware 2 → View → Middleware 2 → Middleware 1 → Response
```

**Default middleware in Django:**

#### `SecurityMiddleware`
Handles several HTTP-level security headers and redirects. Key things it does:
- Redirects HTTP to HTTPS when `SECURE_SSL_REDIRECT = True`
- Sets the `Strict-Transport-Security` header (tells browsers to only use HTTPS)
- Sets `X-Content-Type-Options: nosniff` (prevents browsers from guessing content types)

#### `SessionMiddleware`
Enables the session framework. It reads the session cookie from the request and makes `request.session` available in views.

#### `CommonMiddleware`
Handles small conveniences like:
- Appending a trailing slash to URLs if `APPEND_SLASH = True`
- Returning a 404 for disallowed hosts (checks `ALLOWED_HOSTS`)

#### `CsrfViewMiddleware`
Protects against CSRF attacks (see CSRF section below). It checks that POST requests include a valid CSRF token.

#### `AuthenticationMiddleware`
Attaches the logged-in user to every request. Makes `request.user` available in every view.

#### `MessageMiddleware`
Enables the flash messages framework. Works with `django.contrib.messages`.

#### `XFrameOptionsMiddleware`
Protects against Clickjacking by setting the `X-Frame-Options` header (see Clickjacking section below).

---

### Django Security

#### CSRF — Cross-Site Request Forgery

**What is it?**
Imagine you are logged into your bank. In another tab, you visit a malicious website. That site secretly submits a form to your bank using your active session cookie. Your bank sees a valid session and processes the request — even though you never intended it.

**How Django protects against it:**
Django generates a unique CSRF token for every session. When a form is submitted, Django checks that the token in the form matches the token in the cookie. A malicious third-party site cannot read your cookie, so it cannot forge the token.

In templates, you add `{% csrf_token %}` inside every form:
```html
<form method="POST">
    {% csrf_token %}
    ...
</form>
```

The `CsrfViewMiddleware` validates this token on every POST, PUT, PATCH, and DELETE request.

---

#### XSS — Cross-Site Scripting

**What is it?**
An attacker injects malicious JavaScript into a page that other users load. For example, if a site displays user-submitted comments without sanitising them, an attacker could post a comment containing `<script>steal_cookies()</script>`. Every user who loads the page runs that script.

**Django's role:**
Django's template engine **auto-escapes** all variables by default. So if a user submits `<script>alert('xss')</script>`, Django renders it as the plain text `&lt;script&gt;alert('xss')&lt;/script&gt;` — harmless on screen, not executed.

You opt out of escaping only when you deliberately trust the content, using `{{ content|safe }}` or `{% autoescape off %}`. Be careful with these.

---

#### Clickjacking

**What is it?**
An attacker embeds your site inside a hidden `<iframe>` on their page. They overlay their own UI on top. The user thinks they are clicking on the attacker's button but they are actually clicking on your site underneath — transferring money, changing settings, etc.

**How Django prevents it:**
The `XFrameOptionsMiddleware` sets the `X-Frame-Options: DENY` header on every response. This tells browsers to refuse to load the page inside any `<iframe>`. You can also set it to `SAMEORIGIN` to allow iframes from the same domain only.

```python
X_FRAME_OPTIONS = 'DENY'  # default
```

---

### WSGI

**WSGI** stands for Web Server Gateway Interface. It is a standard protocol that defines how a Python web application talks to a web server.

When you run `python manage.py runserver`, Django uses its own built-in WSGI server. In production, you use a real web server like **Gunicorn** or **uWSGI** as the WSGI server, and put **Nginx** in front of it.

```
Browser → Nginx → Gunicorn (WSGI) → Django app
```

Your project has a `wsgi.py` file that exposes the WSGI callable Django needs:

```python
# wsgi.py
from django.core.wsgi import get_wsgi_application
application = get_wsgi_application()
```

The modern alternative is **ASGI** (Asynchronous Server Gateway Interface), used when you need WebSockets or async views.

---

## 2. Models File

### `on_delete` Options

When you define a `ForeignKey`, Django requires you to specify what happens to the child row when the parent row is deleted.

```python
class Delivery(models.Model):
    match = models.ForeignKey(Match, on_delete=models.CASCADE)
```

| Option | What happens |
|---|---|
| `CASCADE` | Delete the child rows too. Delete a match → all its deliveries are deleted. |
| `PROTECT` | Block the deletion. You cannot delete a match if deliveries still reference it. Raises an error. |
| `SET_NULL` | Set the foreign key column to `NULL`. Requires `null=True` on the field. |
| `SET_DEFAULT` | Set the foreign key to its default value. Requires a `default` on the field. |
| `DO_NOTHING` | Do nothing at the Django level. Can cause database integrity errors if the DB enforces constraints. |
| `RESTRICT` | Similar to PROTECT but slightly stricter — blocks deletion even if a cascade could technically clear the way. |

---

### Fields

Django model fields map to database column types. Here are the most common:

| Field | Use for |
|---|---|
| `CharField(max_length=n)` | Short strings — names, titles |
| `TextField()` | Long strings — descriptions, content |
| `IntegerField()` | Whole numbers |
| `FloatField()` | Decimal numbers |
| `BooleanField()` | True / False |
| `DateField()` | Dates |
| `DateTimeField()` | Date + time |
| `ForeignKey()` | Many-to-one relationship |
| `ManyToManyField()` | Many-to-many relationship |
| `OneToOneField()` | One-to-one relationship |

Common field options:
- `null=True` — allow NULL in the database
- `blank=True` — allow empty value in forms
- `default=value` — set a default
- `unique=True` — enforce uniqueness at the DB level

---

### Validators

Validators are functions that raise a `ValidationError` if a value does not meet a condition. You attach them to fields.

```python
from django.core.validators import MinValueValidator, MaxValueValidator

class Player(models.Model):
    age = models.IntegerField(validators=[MinValueValidator(10), MaxValueValidator(50)])
```

Built-in validators include:
- `MinValueValidator` / `MaxValueValidator` — numeric range
- `MinLengthValidator` / `MaxLengthValidator` — string length
- `EmailValidator` — valid email format
- `URLValidator` — valid URL format
- `RegexValidator` — custom pattern match

You can also write your own:

```python
def no_spaces(value):
    if ' ' in value:
        raise ValidationError("No spaces allowed.")
```

---

### Python Module vs Python Class

**Module** — a `.py` file. It is a container that can hold classes, functions, and variables.

```
models.py      ← this is a module
views.py       ← this is a module
```

**Class** — a blueprint for creating objects. It lives inside a module.

```python
# models.py (module)

class Match(models.Model):     # ← this is a class
    season = models.IntegerField()

class Delivery(models.Model):  # ← this is another class
    match = models.ForeignKey(Match, on_delete=models.CASCADE)
```

So `models.py` is a **module**. `Match` and `Delivery` are **classes** defined inside that module. When you import them, you import from the module: `from ipl.models import Match`.

---

## 3. Django ORM

### Using ORM Queries in the Django Shell

The Django shell gives you a Python prompt with your project loaded. You can query the database interactively.

```bash
python manage.py shell
```

```python
from ipl.models import Match, Delivery

# Get all matches
Match.objects.all()

# Filter
Match.objects.filter(season=2016)

# Get one record
Match.objects.get(id=1)

# Chaining filters
Match.objects.filter(season=2016, winner='Mumbai Indians')

# Exclude
Match.objects.exclude(result='no result')

# Count
Match.objects.filter(season=2016).count()

# Order by
Match.objects.order_by('-season')  # descending
```

---

### Turning ORM Queries into Raw SQL

To see the SQL Django generates for any queryset, use `.query`:

```python
qs = Match.objects.filter(season=2016)
print(qs.query)
```

Output:
```sql
SELECT * FROM ipl_match WHERE season = 2016
```

For more complex queries with aggregations, this is very useful for debugging and understanding what is actually hitting the database.

---

### Aggregations

Aggregations collapse many rows into a single value — a count, sum, average, min, or max. You import them from `django.db.models`.

```python
from django.db.models import Count, Sum, Avg, Max, Min
from ipl.models import Match

# Total number of matches
Match.objects.aggregate(total=Count('id'))
# {'total': 756}

# Matches per season
Match.objects.values('season').annotate(count=Count('id'))
```

`aggregate()` returns a dictionary. It applies to the whole queryset.

Common aggregation functions:
- `Count` — number of rows
- `Sum` — total of a numeric column
- `Avg` — average value
- `Max` / `Min` — highest / lowest value

---

### Annotations

Annotations add a **computed column** to each row in the queryset, rather than collapsing all rows into one value. Think of it as adding a new calculated field per group.

```python
from django.db.models import Count
from ipl.models import Match

# Add a match count to each season group
Match.objects.values('season').annotate(matches_played=Count('id'))

# Each row now has: {'season': 2008, 'matches_played': 58}
```

The difference:
- `aggregate()` → one result for the whole queryset
- `annotate()` → one result per row / group

---

### Migration Files

A migration file is auto-generated code that describes a change to your database schema — creating a table, adding a column, changing a field type, etc.

When you change a model, you run:
```bash
python manage.py makemigrations   # creates the migration file
python manage.py migrate          # applies it to the database
```

**Why is it needed?**
Your Python model and your actual database table can get out of sync. Migrations are the version-controlled record of every schema change. They let you:
- Apply the same changes to any environment (dev, staging, production)
- Roll back a change if something goes wrong
- Track the history of how your schema evolved

Without migrations, you would have to manually write and run `ALTER TABLE` SQL every time you changed a model.

---

### SQL Transactions

A transaction is a group of SQL operations that are treated as a single unit. Either **all of them succeed**, or **none of them are applied**.

Consider loading data from a CSV:
```
Insert row 1 ✓
Insert row 2 ✓
Insert row 3 ✗ (fails — bad data)
```

Without a transaction: rows 1 and 2 are in the database, row 3 is not. Your data is now in a broken, partial state.

With a transaction: when row 3 fails, the entire transaction rolls back. The database is left exactly as it was before you started — clean.

This is the core guarantee of transactions: **all or nothing**.

---

### Atomic Transactions

In Django, you mark a block of code as atomic using `transaction.atomic()`. If any exception is raised inside the block, Django rolls back every database operation that happened within it.

```python
from django.db import transaction

with transaction.atomic():
    for row in csv_data:
        Match.objects.create(**row)
```

You can also use it as a decorator on a function:

```python
@transaction.atomic
def import_data():
    for row in csv_data:
        Match.objects.create(**row)
```

In the IPL project, the `import_ipl_data` management command wraps the entire CSV import in `transaction.atomic()`. If any row fails to insert, the whole import rolls back — leaving the database empty and consistent, rather than half-loaded.
