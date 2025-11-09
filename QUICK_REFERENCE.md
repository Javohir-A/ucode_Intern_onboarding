# Ucode Quick Reference Guide

A handy cheat sheet for quick lookups during the onboarding program.

---

## 🗂️ Table of Contents

- [CRUD Operations](#crud-operations)
- [Field Types](#field-types)
- [Relationships](#relationships)
- [Permissions](#permissions)
- [API Reference](#api-reference)
- [Filters & Operators](#filters--operators)
- [Common Commands](#common-commands)
- [Troubleshooting](#troubleshooting)

---

## CRUD Operations

### In Admin Panel

| Action | Steps |
|--------|-------|
| **Create** | Navigate to table → Click "+" or "Create New Item" → Fill fields → Save |
| **Read** | Navigate to table → Click on item to view details |
| **Update** | Open item → Click "Edit" → Modify fields → Save |
| **Delete** | Open item → Click "Delete" → Confirm |
| **Search** | Use search box at top of item list |
| **Filter** | Click filter icon → Add conditions → Apply |
| **Sort** | Click column header to sort |

---

## Field Types

| Field Type | Use Case | Example |
|------------|----------|---------|
| **String** | Short text | Names, titles, usernames |
| **Text** | Long text | Descriptions, blog content |
| **Integer** | Whole numbers | Counts, IDs, ages |
| **Decimal/Float** | Numbers with decimals | Prices, measurements |
| **Boolean** | Yes/No, True/False | Is published, is active |
| **Date** | Calendar date | Birth date, publish date |
| **DateTime** | Date + Time | Created at, updated at |
| **Email** | Email with validation | user@example.com |
| **URL** | Website links | https://example.com |
| **JSON** | Structured data | Settings, metadata |
| **UUID** | Unique identifier | Auto-generated IDs |
| **File** | File uploads | Images, PDFs, documents |

---

## Relationships

### One-to-Many (1:M)

**Concept**: One parent has many children, each child has one parent

**Example**: One author has many blog posts
```
authors (1) ----< (many) blog_posts
```

**Implementation**:
1. Add field to "many" side: `author_id`
2. Type: Many-to-One (Foreign Key)
3. Related table: `authors`

### Many-to-Many (M:N)

**Concept**: Multiple on both sides can be related

**Example**: Blog posts have many tags, tags belong to many posts
```
blog_posts (M) >----< (M) tags
```

**Implementation** (Option 1 - Junction Table):
1. Create junction table: `blog_posts_tags`
2. Add two foreign keys:
   - `blog_post_id` → blog_posts
   - `tag_id` → tags

**Implementation** (Option 2 - Many-to-Many Field):
1. Add field to one table: `tags`
2. Type: Many-to-Many
3. Related table: `tags`

### Self-Referencing

**Example**: Comments reply to other comments
```
comments
  ├─ id
  ├─ content
  └─ parent_comment_id → comments.id
```

---

## Permissions

### Permission Levels

| Level | Description |
|-------|-------------|
| **None** | No access to table/field |
| **Read** | Can view data |
| **Create** | Can add new records |
| **Update** | Can modify existing records |
| **Delete** | Can remove records |
| **Full** | All permissions |

### Common Role Patterns

**Admin**: Full access to everything
**Manager**: CRUD on content, Read on users
**Editor**: CRUD on content (own items), Read on others
**Member**: Read all, Update own, Create limited
**Viewer**: Read-only access

### Permission Matrix Template

| Table | Admin | Manager | Editor | Member | Viewer |
|-------|-------|---------|--------|--------|--------|
| content | Full | CRUD | CRUD* | RU* | R |
| users | Full | R | R | R | - |
| settings | Full | R | - | - | - |

\* = own items only

---

## API Reference

### Base Structure

```
Base URL: https://your-instance.ucode.com
Authorization: Bearer YOUR_TOKEN
Content-Type: application/json
```

### CRUD Endpoints

```http
# Get all items
GET /items/{table}

# Get single item
GET /items/{table}/{id}

# Create item
POST /items/{table}
Body: { "field": "value", ... }

# Update item
PATCH /items/{table}/{id}
Body: { "field": "new_value", ... }

# Delete item
DELETE /items/{table}/{id}
```

### Query Parameters

```http
# Select specific fields
?fields=id,title,author.*

# Filter
?filter[field][_operator]=value

# Sort (ascending)
?sort=field

# Sort (descending)
?sort=-field

# Pagination
?limit=20&page=1

# Search
?search=keyword

# Combine multiple
?fields=*,author.*&filter[published][_eq]=true&sort=-created_at&limit=10
```

### Status Codes

| Code | Meaning |
|------|---------|
| **200** | Success |
| **201** | Created |
| **204** | Deleted (no content) |
| **400** | Bad request (invalid data) |
| **401** | Unauthorized (not logged in) |
| **403** | Forbidden (no permission) |
| **404** | Not found |
| **500** | Server error |

---

## Filters & Operators

### Filter Syntax

```
?filter[field][_operator]=value
```

### Common Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `_eq` | Equals | `?filter[status][_eq]=published` |
| `_neq` | Not equals | `?filter[status][_neq]=draft` |
| `_gt` | Greater than | `?filter[price][_gt]=100` |
| `_gte` | Greater or equal | `?filter[price][_gte]=100` |
| `_lt` | Less than | `?filter[views][_lt]=50` |
| `_lte` | Less or equal | `?filter[views][_lte]=50` |
| `_contains` | Contains text | `?filter[title][_contains]=tutorial` |
| `_ncontains` | Doesn't contain | `?filter[title][_ncontains]=draft` |
| `_in` | In array | `?filter[id][_in]=1,2,3` |
| `_nin` | Not in array | `?filter[id][_nin]=4,5` |
| `_null` | Is null | `?filter[deleted_at][_null]=true` |
| `_nnull` | Not null | `?filter[published_at][_nnull]=true` |
| `_between` | Between values | `?filter[price][_between]=10,100` |

### Multiple Filters (AND)

```
?filter[published][_eq]=true&filter[views][_gt]=100
```

### Complex Filters (OR)

```json
{
  "filter": {
    "_or": [
      { "status": { "_eq": "published" } },
      { "featured": { "_eq": true } }
    ]
  }
}
```

---

## Common Commands

### API Testing (curl)

```bash
# Login
curl -X POST https://your-instance.ucode.com/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"password"}'

# Get data with token
curl -X GET https://your-instance.ucode.com/items/blog_posts \
  -H "Authorization: Bearer YOUR_TOKEN"

# Create item
curl -X POST https://your-instance.ucode.com/items/blog_posts \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Test Post","content":"Hello"}'

# Update item
curl -X PATCH https://your-instance.ucode.com/items/blog_posts/1 \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Updated Title"}'

# Delete item
curl -X DELETE https://your-instance.ucode.com/items/blog_posts/1 \
  -H "Authorization: Bearer YOUR_TOKEN"
```

---

## Troubleshooting

### Common Issues & Solutions

| Problem | Possible Cause | Solution |
|---------|---------------|----------|
| Can't log in | Wrong credentials | Check email/password, contact mentor |
| Save fails | Validation error | Check required fields, data types |
| 401 API error | Not authenticated | Check token, refresh login |
| 403 API error | No permission | Check role permissions |
| Relationship shows no data | Not configured | Verify foreign keys, check data exists |
| Filter returns nothing | Too restrictive | Simplify filters, check data |
| Slow performance | Large dataset | Use pagination, limit fields |
| Can't see table | No permission | Check role permissions |
| Can't delete item | Referenced elsewhere | Check relationships/dependencies |

### Debug Checklist

When something doesn't work:

1. ✅ Check error message (read it carefully!)
2. ✅ Verify permissions (do you have access?)
3. ✅ Check data exists (is there data to display?)
4. ✅ Confirm relationships (are foreign keys correct?)
5. ✅ Review configuration (is everything set up right?)
6. ✅ Refresh page (sometimes it's just that!)
7. ✅ Check browser console (F12) for errors
8. ✅ Ask mentor if stuck >15 minutes

---

## Keyboard Shortcuts

### General

| Shortcut | Action |
|----------|--------|
| `Ctrl/Cmd + S` | Save |
| `Ctrl/Cmd + K` | Search |
| `Esc` | Close dialog/cancel |
| `Ctrl/Cmd + Enter` | Submit form |

*Note: Shortcuts may vary by Ucode version*

---

## Useful Formulas & Patterns

### Naming Conventions

**Tables**: lowercase, underscores
- ✅ `blog_posts`, `user_profiles`
- ❌ `BlogPosts`, `User-Profiles`

**Fields**: lowercase, underscores
- ✅ `first_name`, `created_at`
- ❌ `firstName`, `Created_At`

**Roles**: Title Case, spaces OK
- ✅ `Blog Editor`, `Content Manager`
- ❌ `blog_editor`, `CONTENT-MANAGER`

### Data Model Best Practices

1. **Always have a primary key** (usually `id`)
2. **Use timestamps** (`created_at`, `updated_at`)
3. **Foreign keys end in _id** (`author_id`, `category_id`)
4. **Junction tables**: `table1_table2` (`posts_tags`)
5. **Boolean fields**: prefix with `is_` or `has_` (`is_published`)
6. **Dates/times**: suffix with `_at` or `_date` (`published_at`)

### Security Principles

1. **Least Privilege**: Minimum permissions needed
2. **Test Thoroughly**: Try to break your permissions
3. **Document Decisions**: Why each role needs what access
4. **Separate Concerns**: Each role has clear purpose
5. **Never trust client input**: Always validate

---

## Quick Tips

### Day 1 Tips
- Explore freely, you won't break anything
- Take notes as you learn
- Test validation thoroughly
- Document everything you discover

### Day 2 Tips
- Draw your data model before creating tables
- Test relationships with actual data
- Think about normalization
- Consider future needs

### Day 3 Tips
- Test permissions from user's perspective
- Use private browsing for testing different users
- Document your permission reasoning
- Start with least privilege

### Day 4 Tips
- Save your API collection
- Test error cases, not just success
- Use meaningful view names
- Design views for actual users

### Day 5 Tips
- Plan before you build
- Start with MVP, add features later
- Test as you go, not at the end
- Done is better than perfect

---

## Resource Links

- [Ucode Documentation](https://ucode.gitbook.io/ucode-docs)
- [Program README](./README.md)
- [FAQ](./FAQ.md)
- [Mentor Guide](./MENTOR_GUIDE.md)
- [Progress Tracker](./PROGRESS.md)

---

## Print This Page!

Consider printing this reference guide to have it handy during exercises.

---

**Last Updated:** 2025  
**Version:** 1.0

