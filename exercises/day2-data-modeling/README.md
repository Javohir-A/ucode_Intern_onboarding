# Day 2: Data Modeling - Tables, Fields & Relationships

## ⏱️ Duration: 4-5 hours

## 🎯 Learning Objectives

By the end of Day 2, you will:
- Create new tables from scratch
- Configure different field types
- Understand and implement One-to-Many relationships
- Understand and implement Many-to-Many relationships
- Design a normalized database schema
- Apply data modeling best practices

## 📚 Morning Session (2-2.5 hours)

### Part 1: Creating Tables (1 hour)

#### Activity 1.1: Create Your First Table (20 min)

**Scenario:** You're building a blog system. Start with a `blog_posts` table.

1. Navigate to **Tables** section
2. Click "Create New Table"
3. Configure the table:
   ```
   Table Name: blog_posts
   Icon: (choose an appropriate icon)
   ```
4. Add these fields:

| Field Name | Type | Required | Notes |
|------------|------|----------|-------|
| id | Auto-increment | Yes | Primary key |
| title | String/Text | Yes | Max 255 chars |
| content | Long Text | Yes | Full blog content |
| published | Boolean | No | Default: false |
| publish_date | Date | No | When published |
| views | Integer | No | Default: 0 |

5. Save the table
6. Screenshot: `screenshots/blog_posts_table.png`

#### Activity 1.2: Understand Field Types (20 min)

Create a practice table called `field_types_demo` with one of each type:

| Field Name | Field Type | Purpose |
|------------|------------|---------|
| text_field | String | Short text |
| long_text_field | Text | Long content |
| number_field | Integer | Whole numbers |
| decimal_field | Decimal/Float | Prices, measurements |
| boolean_field | Boolean | Yes/No, True/False |
| date_field | Date | Calendar date |
| datetime_field | DateTime | Date + Time |
| email_field | Email | Email validation |
| url_field | URL | Website links |
| json_field | JSON | Structured data |

Document in `field-types.md`:
- What validation does each type provide?
- When would you use each type?

#### Activity 1.3: Field Configuration Options (20 min)

For the `blog_posts` table, explore these options:
1. **Required vs Optional**: Make title required
2. **Default Values**: Set views default to 0
3. **Unique Constraint**: Should titles be unique?
4. **Hidden Fields**: Can fields be hidden?
5. **Read-only**: Can fields be read-only?

Document all options in `field-config.md`

### Part 2: One-to-Many Relationships (1-1.5 hours)

#### Activity 2.1: Understanding One-to-Many (15 min)

**Concept:** One author can write many blog posts, but each blog post has only one author.

Read about relationships in Ucode docs, then document in `relationships-notes.md`:
- What is a One-to-Many relationship?
- Real-world examples (3-5)
- How is this stored in the database?

#### Activity 2.2: Create Authors Table (20 min)

1. Create new table: `authors`
2. Add fields:

| Field Name | Type | Required |
|------------|------|----------|
| id | Auto-increment | Yes |
| first_name | String | Yes |
| last_name | String | Yes |
| email | Email | Yes (Unique) |
| bio | Text | No |
| active | Boolean | No |

3. Save the table
4. Add 3-5 author records

#### Activity 2.3: Create the Relationship (25 min)

1. Go back to `blog_posts` table
2. Add a new field: `author_id`
   - Type: **Many-to-One** (or Foreign Key)
   - Related Table: `authors`
   - Display Field: Choose which author field to show
3. Save the configuration
4. Screenshot: `screenshots/author_relationship.png`

#### Activity 2.4: Test the Relationship (15 min)

1. Create 3 new blog posts
2. Assign each to a different author
3. View an author record - can you see their posts?
4. Edit a blog post - how do you select the author?
5. Document behavior in `relationships-notes.md`

## 🍽️ Lunch Break (30-60 min)

## 📚 Afternoon Session (2-2.5 hours)

### Part 3: Many-to-Many Relationships (1.5 hours)

#### Activity 3.1: Understanding Many-to-Many (15 min)

**Concept:** A blog post can have many tags, and a tag can belong to many blog posts.

Document in `relationships-notes.md`:
- What is a Many-to-Many relationship?
- Real-world examples (3-5)
- Why do we need a junction table?

#### Activity 3.2: Create Tags Table (15 min)

1. Create new table: `tags`
2. Add fields:

| Field Name | Type | Required |
|------------|------|----------|
| id | Auto-increment | Yes |
| name | String | Yes (Unique) |
| slug | String | Yes (Unique) |
| color | String | No |

3. Add 8-10 tag records:
   - Technology
   - Tutorial
   - News
   - Review
   - Opinion
   - Guide
   - Tips
   - Best Practices

#### Activity 3.3: Create Junction Table (30 min)

**Method 1: Manual Junction Table**
1. Create table: `blog_posts_tags`
2. Add fields:

| Field Name | Type | Required |
|------------|------|----------|
| id | Auto-increment | Yes |
| blog_post_id | Many-to-One | Yes |
| tag_id | Many-to-One | Yes |

**Method 2: Many-to-Many Field** (if Ucode supports)
1. Add field to `blog_posts`: `tags`
2. Type: Many-to-Many
3. Related Table: `tags`

Choose the method available in your Ucode version.  
Screenshot: `screenshots/many_to_many_setup.png`

#### Activity 3.4: Test Many-to-Many (30 min)

1. Go to a blog post
2. Add 2-3 tags to it
3. Go to another blog post
4. Add some of the same tags + different ones
5. View a tag - can you see all posts with that tag?

**Document in `relationships-notes.md`:**
- How do you add multiple tags to a post?
- How do you view all posts for a tag?
- What happens if you delete a tag?

### Part 4: Complete Data Model Design (1 hour)

#### Activity 4.1: Add Categories Table (15 min)

Extend your blog with categories (different from tags):
1. Create `categories` table
2. Add fields: id, name, description, slug
3. Add relationship to blog_posts (One-to-Many)
4. Add 4-5 categories:
   - Web Development
   - Mobile Development
   - Data Science
   - DevOps
   - Design

#### Activity 4.2: Add Comments Table (20 min)

Add ability for users to comment on posts:
1. Create `comments` table
2. Fields:

| Field Name | Type | Required |
|------------|------|----------|
| id | Auto-increment | Yes |
| blog_post_id | Many-to-One | Yes |
| author_name | String | Yes |
| author_email | Email | Yes |
| content | Text | Yes |
| created_at | DateTime | Yes |
| approved | Boolean | No |

3. Add relationship to `blog_posts`
4. Create 3-5 test comments

#### Activity 4.3: Data Model Diagram (25 min)

Create a complete diagram showing all tables and relationships:

```
authors (1) ----< (M) blog_posts (M) >----< (M) tags
                       |
                       | (1)
                       |
                       v (M)
                     comments
                       
                 blog_posts (M) >---- (1) categories
```

**Tasks:**
1. Draw this diagram (digital or hand-drawn, then photographed)
2. Label each relationship type (One-to-Many, Many-to-Many)
3. Save as: `diagrams/blog-data-model.png`
4. In `data-model.md`, explain each table's purpose

## ✅ End of Day Checklist

- [ ] Created at least 5 tables
- [ ] Used 8+ different field types
- [ ] Implemented One-to-Many relationship (authors-posts)
- [ ] Implemented Many-to-Many relationship (posts-tags)
- [ ] Created junction table
- [ ] Added sample data to all tables
- [ ] Created complete data model diagram
- [ ] Tested all relationships
- [ ] Completed all documentation

## 📝 Deliverables

1. **field-types.md** - Field types reference
2. **field-config.md** - Field configuration options
3. **relationships-notes.md** - Relationship explanations and findings
4. **data-model.md** - Complete data model documentation
5. **screenshots/** - All screenshots
6. **diagrams/blog-data-model.png** - Data model diagram

## 🚀 Bonus Challenges

1. **Self-Referencing**: Create a relationship where comments can reply to other comments (parent-child)
2. **User Table**: Create a `users` table and link it to authors
3. **Media**: Add an `images` table for blog post featured images
4. **Advanced**: Add a `post_revisions` table to track edit history

## 💡 Data Modeling Best Practices

Document these in `best-practices.md`:
1. When to use One-to-Many vs Many-to-Many?
2. How to name tables (singular vs plural)?
3. When to create a separate table vs add fields?
4. How to handle optional relationships?
5. What makes a good primary key?

## 📊 Self-Assessment Questions

Answer in `self-assessment.md`:

1. What's the difference between One-to-Many and Many-to-Many?
2. Why do we need a junction table for Many-to-Many?
3. When should a field be required vs optional?
4. What field type would you use for a blog post's featured image?
5. How would you model a relationship where a post has one primary author and multiple co-authors?

## ❓ Common Issues

**Can't create relationship?**
- Ensure both tables exist first
- Check that field types are compatible
- Verify you have permissions

**Relationship not showing data?**
- Make sure you've created records in both tables
- Check that the foreign key is properly set
- Refresh the page

**Junction table errors?**
- Verify both foreign keys are configured correctly
- Ensure no duplicate relationships exist

## 📖 Resources

- [Ucode Tables](https://ucode.gitbook.io/ucode-docs/user-guide/table)
- [Ucode Fields & Relations](https://ucode.gitbook.io/ucode-docs/user-guide/field-and-relation)

---

**Previous:** [Day 1 - Foundations](../day1-foundations/)  
**Next:** [Day 3 - Security & Access Control](../day3-security/)

