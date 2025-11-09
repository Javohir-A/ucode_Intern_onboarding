# Frequently Asked Questions (FAQ)

## General Questions

### What is Ucode?

Ucode is a Backend as a Service (BaaS) platform that sits on top of your database. It automatically generates REST APIs and provides a no-code admin interface for managing your data. Unlike traditional platforms, Ucode uses database introspection - it reads your existing database structure and mirrors it, giving you complete control over your data without vendor lock-in.

### How is Ucode different from other platforms?

**Key Differences:**
- **Database Introspection**: Works with YOUR database, not a proprietary one
- **No Vendor Lock-in**: Your data stays in your database
- **Auto-generated APIs**: REST APIs created automatically from your schema
- **Flexible**: Can be added or removed from any project anytime

### Do I need programming experience?

Basic understanding helps, but Ucode is designed to be accessible:
- **No coding needed** for basic CRUD operations
- **Basic SQL knowledge** helpful for data modeling
- **Programming required** only for custom endpoints and functions

### What databases does Ucode support?

Ucode supports major databases including:
- PostgreSQL
- MySQL
- SQLite
- Microsoft SQL Server
- And others (check your version documentation)

---

## Technical Questions

### Day 1: Getting Started & CRUD

**Q: I can't log in. What should I do?**

A: Check the following:
1. Are you using the correct URL?
2. Is your email/username correct?
3. Is your password correct? (it's case-sensitive)
4. Is your account active?
5. Contact your mentor if still stuck

**Q: What's database introspection?**

A: Database introspection is when Ucode "reads" your existing database structure (tables, fields, relationships) and automatically creates an interface and API for it. You don't need to configure anything manually - Ucode detects everything automatically.

**Q: I deleted something by mistake. Can I recover it?**

A: It depends on your Ucode configuration:
- Some instances use "soft delete" (items go to archive/trash)
- Some use "hard delete" (permanent removal)
- Ask your mentor how your instance is configured
- This is a learning environment, so don't worry too much!

**Q: Why did my save fail?**

A: Common reasons:
- Required field is empty
- Wrong data type (e.g., text in a number field)
- Unique constraint violation (duplicate value)
- Permission issue
- Check the error message for details

**Q: What's the difference between Update and Delete?**

A: 
- **Update**: Modify existing data, the record remains
- **Delete**: Remove the entire record

---

### Day 2: Data Modeling

**Q: What's the difference between One-to-Many and Many-to-Many?**

A: 
- **One-to-Many**: One parent can have many children, but each child has only one parent
  - Example: One author writes many blog posts, but each post has one author
  
- **Many-to-Many**: Multiple items on both sides can be related to multiple items on the other side
  - Example: Blog posts can have many tags, and tags can be on many blog posts

**Q: When do I need a junction table?**

A: Junction tables are needed for **Many-to-Many relationships**. They connect two tables by storing the IDs of both related items.

Example:
```
blog_posts ←→ blog_posts_tags ←→ tags
```

**Q: What field type should I use for...?**

Common field types:
- **Short text** (names, titles): String (255 chars)
- **Long text** (descriptions, content): Text
- **Money/prices**: Decimal/Float
- **Counts/quantities**: Integer
- **Yes/No values**: Boolean
- **Dates**: Date or DateTime
- **Email addresses**: Email (includes validation)
- **URLs**: URL (includes validation)
- **Complex data**: JSON

**Q: My relationship isn't showing any data. Why?**

A: Checklist:
1. Do both tables have data?
2. Is the foreign key field properly configured?
3. Are records actually related (foreign key values match)?
4. Do you have permission to view the related table?
5. Try refreshing the page

**Q: Should I use singular or plural for table names?**

A: This is a preference, but be consistent:
- **Plural**: `products`, `users`, `blog_posts` (common in Rails)
- **Singular**: `product`, `user`, `blog_post` (common in some frameworks)

Choose one convention and stick with it throughout your project.

---

### Day 3: Security & Permissions

**Q: What's the difference between a role and a permission?**

A:
- **Permission**: A specific action (e.g., "can create blog posts")
- **Role**: A collection of permissions (e.g., "Blog Editor" has multiple permissions)
- **User**: Has one or more roles, which give them their permissions

**Q: I can't see a table/field I should have access to. Why?**

A: Possible reasons:
1. Your role doesn't have permission for that table/field
2. Field-level permissions are hiding it
3. The table/field doesn't exist
4. You need to refresh the page
5. Ask your mentor to check your permissions

**Q: How do I test permissions without affecting my main account?**

A: 
1. Create test user accounts with different roles
2. Use incognito/private browser windows
3. Log in as different users to test their access
4. Document what each role can/cannot do

**Q: What's the "Principle of Least Privilege"?**

A: Give users the **minimum permissions** they need to do their job - nothing more. This reduces security risks. For example:
- Viewers should only read, not edit
- Editors shouldn't be able to delete
- Only admins should manage users

**Q: Can I make a field visible but not editable?**

A: Yes! This is called "read-only" access. Configure field-level permissions:
- Read: ✓ (can see the field)
- Update: ✗ (can't change it)

---

### Day 4: API Integration

**Q: What's a REST API?**

A: REST (Representational State Transfer) is a standard way for applications to communicate over HTTP. Key concepts:
- **GET**: Retrieve data
- **POST**: Create new data
- **PATCH/PUT**: Update data
- **DELETE**: Remove data

**Q: I'm getting a 401 error. What does this mean?**

A: **401 Unauthorized** means authentication failed:
1. Check your access token is valid
2. Verify the Authorization header format: `Bearer YOUR_TOKEN`
3. Token might have expired - get a new one
4. Make sure you're logged in

**Q: What's the difference between 401 and 403?**

A:
- **401 Unauthorized**: You're not authenticated (not logged in)
- **403 Forbidden**: You're authenticated, but don't have permission

**Q: How do I include related data in API responses?**

A: Use the `fields` parameter:
```
GET /items/blog_posts?fields=*,author.*,tags.*

* = all fields of blog_posts
author.* = all fields of related author
tags.* = all fields of related tags
```

**Q: What API testing tool should I use?**

A: Popular options:
- **Postman**: Feature-rich, beginner-friendly (recommended)
- **Insomnia**: Clean interface, good for REST
- **Thunder Client**: VS Code extension
- **curl**: Command-line, powerful but less visual

Choose whichever you're comfortable with!

**Q: How do I filter API results?**

A: Use filter parameters:
```
# Equals
?filter[published][_eq]=true

# Greater than
?filter[views][_gt]=100

# Contains
?filter[title][_contains]=tutorial

# Multiple conditions (AND)
?filter[published][_eq]=true&filter[views][_gt]=50
```

Common operators: `_eq`, `_neq`, `_gt`, `_gte`, `_lt`, `_lte`, `_contains`, `_in`

**Q: What's pagination and why do I need it?**

A: Pagination splits large datasets into smaller "pages". Benefits:
- Faster loading (don't fetch 10,000 records at once!)
- Better performance
- Manageable data chunks

Usage:
```
?limit=20&page=1    # First 20 items
?limit=20&page=2    # Next 20 items
```

---

### Day 5: Capstone Project

**Q: I'm running out of time. What should I prioritize?**

A: Priority order:
1. **Core data model** (tables and relationships)
2. **Sample data** (so you can demo)
3. **Basic permissions** (at least 2 roles)
4. **One working view**
5. **Basic API testing**
6. Bonus features (skip if time is tight)

**Q: My feature isn't working. How do I debug?**

A: Systematic debugging:
1. **What exactly isn't working?** (be specific)
2. **What error message do you see?**
3. **What did you expect to happen?**
4. **When did it last work?** (what changed?)
5. **Can you reproduce the issue?**
6. Check: permissions, data, relationships, configuration

**Q: Is my project "good enough"?**

A: Ask yourself:
- Does it meet the core requirements?
- Can you demonstrate it working?
- Can you explain your design decisions?
- Is it documented?

If yes to all → it's good enough! Remember: Done is better than perfect.

**Q: Can I add features beyond the requirements?**

A: Yes, if time permits! But:
1. Finish core requirements first
2. Keep it simple and functional
3. Don't sacrifice quality for extra features
4. Document what you added and why

**Q: I'm nervous about the presentation. Help?**

A: Presentation tips:
1. **Practice** your demo beforehand
2. **Start with overview**, then dive into details
3. **Show, don't just tell** (live demo is powerful)
4. **Explain your thinking** (why you made certain choices)
5. **Be honest** about challenges
6. **Remember**: Everyone wants you to succeed!

---

## Workflow Questions

**Q: How much time should I spend on documentation?**

A: Documentation is important but shouldn't dominate:
- Write notes as you go (don't save for the end)
- Keep it concise but clear
- Focus on "why" not just "what"
- 15-20% of your time is reasonable

**Q: When should I ask for help?**

A: Ask for help if:
- You're stuck for more than 15-20 minutes
- You don't understand a core concept
- You're getting repeated errors
- You're not sure if your approach is right

**Don't** wait until the end of the day!

**Q: Can I work ahead?**

A: Yes, but:
1. Make sure you fully understand current day's content
2. Complete all deliverables for current day
3. Some concepts build on previous days
4. Don't rush - depth > speed

**Q: Can I collaborate with other interns?**

A: Yes! Collaboration is encouraged:
- **Do**: Discuss concepts, share ideas, help each other debug
- **Don't**: Copy each other's work, share answers directly
- Everyone should submit their own unique work

---

## Ucode-Specific Questions

**Q: Does Ucode support [specific database feature]?**

A: Check:
1. Ucode official documentation
2. Your specific Ucode version (features vary)
3. Ask your mentor

**Q: Can I use Ucode for production applications?**

A: Yes! Ucode is designed for production use. Many companies use it for:
- Backend APIs
- Headless CMS
- Internal tools
- Admin panels

**Q: How do I learn more after this program?**

A: Continue learning:
- [Ucode Documentation](https://ucode.gitbook.io/ucode-docs)
- Build personal projects
- Contribute to open source
- Join Ucode community
- Read the Developer Guide section

**Q: Can I extend Ucode with custom code?**

A: Yes! Ucode is extensible:
- Custom endpoints
- Custom functions
- Webhooks
- Micro frontends

These are advanced topics for after the onboarding program.

---

## Program Questions

**Q: What if I don't finish everything?**

A: Don't panic!
1. Submit what you have
2. Document what's incomplete
3. Explain your progress and challenges
4. Focus on understanding > completion
5. You can continue learning after the program

**Q: Will there be a final exam/test?**

A: The capstone project (Day 5) is your final evaluation. It's a practical demonstration of everything you learned.

**Q: What happens after I complete the program?**

A: Depends on your situation:
- You may be assigned to projects using Ucode
- You may continue with advanced training
- You'll have foundational skills to build on
- Discuss next steps with your mentor

**Q: Can I access the training environment after the program?**

A: Ask your mentor about continued access to:
- Training Ucode instance
- Sample projects you built
- Documentation and resources

---

## Still Have Questions?

1. **Check the exercise README** for that specific day
2. **Review Ucode documentation**
3. **Ask your mentor** - that's what they're there for!
4. **Discuss with other interns** - they might have the same question

**Remember: There are no stupid questions. If you're confused, ask!** 🙋‍♂️

---

## Quick Links

- [Ucode Official Documentation](https://ucode.gitbook.io/ucode-docs)
- [Program README](./README.md)
- [Progress Tracker](./PROGRESS.md)
- [Mentor Guide](./MENTOR_GUIDE.md)

---

**Last Updated:** 2025  
**Program Version:** 1.0 - 1 Week Intensive

