# Solving the N+1 Query Problem in Django ORM

## 1) The N+1 Problem

Your application starts with a single query:

```sql
SELECT id, title, author_id
FROM posts;
```

Then loops through the result rows and fetches each author:

```sql
SELECT id, name FROM authors WHERE id = 5;
SELECT id, name FROM authors WHERE id = 12;
SELECT id, name FROM authors WHERE id = 5;
-- repeated once per post
```

If you loaded 100 posts, you can easily end up with **101 queries**.

---

## 2) Fix in SQL: JOIN (single query)

```sql
SELECT
  p.id,
  p.title,
  a.id   AS author_id,
  a.name AS author_name
FROM posts p
JOIN authors a ON a.id = p.author_id;
```

Now posts + authors come back in **one query**.

---

## 3) N+1 Example in Django ORM

### Example Models

```python
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)

class Post(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
```

### The N+1 Problem (Lazy Loading)

```python
posts = Post.objects.all()

for post in posts:
    print(post.author.name)  # triggers an extra query per post
```

What typically happens:
- **1 query**: fetch all posts
- **N queries**: each time you access `post.author`, Django queries the DB
- **Total: 1 + N**

---

## 4) Fix #1: `select_related()` (ForeignKey, OneToOne)

### What `select_related()` Does

`select_related()` tells Django to load the related object using a **SQL JOIN**.

Use it when the relation is **single-valued**:
- `ForeignKey`
- `OneToOneField`

### Example (Fixing Post → Author N+1)

```python
posts = Post.objects.select_related("author").all()

for post in posts:
    print(post.author.name)  # author is already fetched
```

### Why It Works

- Django performs a JOIN behind the scenes.
- Usually becomes **one query**, not 1 + N.

### Deep Joins (Follow Relationships)

```python
Order.objects.select_related("user__profile")
```

This joins `Order -> user -> profile` (as long as those are FK/OneToOne links).

---

## 5) Fix #2: `prefetch_related()` (ManyToMany, Reverse ForeignKey)

### What `prefetch_related()` Does

`prefetch_related()` runs **separate queries** and then matches results in Python.

Use it when the relation is **multi-valued**:
- `ManyToManyField`
- Reverse `ForeignKey` (one-to-many)

---

### 5.1) Many-to-Many Example: Post ↔ Tag

#### Models

```python
class Tag(models.Model):
    name = models.CharField(max_length=50)

class Post(models.Model):
    title = models.CharField(max_length=200)
    tags = models.ManyToManyField(Tag)
```

#### N+1 Pattern

```python
posts = Post.objects.all()

for post in posts:
    print([t.name for t in post.tags.all()])  # query per post
```

#### Fix with `prefetch_related("tags")`

```python
posts = Post.objects.prefetch_related("tags").all()

for post in posts:
    print([t.name for t in post.tags.all()])
```

What typically happens now:
- **Query 1**: fetch posts
- **Query 2**: fetch tags for all those posts (and the join table)
- **Total: ~2 queries**, not 1 + N

---

### 5.2) Reverse ForeignKey Example: Author → Posts

#### Models (same as earlier)

```python
class Author(models.Model):
    name = models.CharField(max_length=100)

class Post(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
```

#### N+1 Pattern (Reverse Relation)

```python
authors = Author.objects.all()

for a in authors:
    print(a.post_set.all())  # queries per author
```

#### Fix

```python
authors = Author.objects.prefetch_related("post_set").all()

for a in authors:
    print(a.post_set.all())
```

If you set `related_name="posts"` on `Post.author`, use:

```python
authors = Author.objects.prefetch_related("posts").all()
```

---

## 6) Using Both Together (Real-World Pattern)

Very common when you have both FK and M2M relations:

```python
posts = (
    Post.objects
    .select_related("author")   # FK: single author per post
    .prefetch_related("tags")   # M2M: many tags per post
)
```

---

## Quick Reference

| Relation Type | Method | Queries |
|---|---|---|
| `ForeignKey` / `OneToOneField` | `select_related()` | 1 (JOIN) |
| `ManyToManyField` | `prefetch_related()` | ~2 (separate + merge) |
| Reverse `ForeignKey` | `prefetch_related()` | ~2 (separate + merge) |
