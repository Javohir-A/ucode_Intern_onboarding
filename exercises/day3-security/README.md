# Day 3: Security & Access Control - Users, Roles & Permissions

## ⏱️ Duration: 4-5 hours

## 🎯 Learning Objectives

By the end of Day 3, you will:
- Create and manage users in Ucode
- Define roles with specific permissions
- Configure table-level permissions
- Configure field-level permissions
- Implement granular access control
- Test permission scenarios
- Understand security best practices

## 📚 Morning Session (2-2.5 hours)

### Part 1: Understanding Access Control (45 min)

#### Activity 1.1: Access Control Concepts (20 min)

Read and document in `security-concepts.md`:

**Key Concepts:**
- **Authentication**: Verifying who someone is (login)
- **Authorization**: What they're allowed to do (permissions)
- **Role-Based Access Control (RBAC)**: Permissions assigned to roles, users assigned to roles
- **Principle of Least Privilege**: Give minimum permissions needed

**Answer these questions:**
1. Why do we need roles instead of assigning permissions to individual users?
2. What's the difference between authentication and authorization?
3. Give 3 real-world examples of role-based access control

#### Activity 1.2: Explore Current Setup (15 min)

1. Navigate to **Users** section
2. Document in `current-setup.md`:
   - How many users exist?
   - What roles are available by default?
   - What role do you have?
   - What permissions does your role have?

3. Navigate to **Roles** section
4. Examine each default role
5. Screenshot: `screenshots/default-roles.png`

#### Activity 1.3: Permission Types (10 min)

Understand these permission levels:
- **None**: No access
- **Read**: Can view data
- **Create**: Can add new records
- **Update**: Can modify existing records
- **Delete**: Can remove records
- **Full Access**: All of the above

Document examples of when to use each in `security-concepts.md`

### Part 2: Creating Roles (1-1.5 hours)

#### Activity 2.1: Create a "Blog Editor" Role (20 min)

**Scenario:** Blog editors can create and edit posts, but can't delete them or manage users.

1. Go to **Roles** section
2. Click "Create New Role"
3. Configure:
   ```
   Role Name: Blog Editor
   Description: Can create and edit blog posts and comments
   ```
4. Save the role (don't configure permissions yet)
5. Screenshot: `screenshots/blog-editor-role.png`

#### Activity 2.2: Create a "Blog Viewer" Role (15 min)

**Scenario:** Viewers can only read blog posts and comments.

1. Create new role:
   ```
   Role Name: Blog Viewer
   Description: Read-only access to blog content
   ```
2. Save the role

#### Activity 2.3: Create a "Blog Admin" Role (15 min)

**Scenario:** Admins have full control over all blog-related data.

1. Create new role:
   ```
   Role Name: Blog Admin
   Description: Full access to all blog features
   ```
2. Save the role

#### Activity 2.4: Create a "Comment Moderator" Role (15 min)

**Scenario:** Moderators can approve/delete comments but can't edit posts.

1. Create new role:
   ```
   Role Name: Comment Moderator
   Description: Can manage comments and moderation
   ```
2. Save the role

Document all roles in `roles-created.md`

### Part 3: Configuring Table Permissions (1 hour)

#### Activity 3.1: Configure Blog Editor Permissions (20 min)

1. Select the "Blog Editor" role
2. Go to **Permissions** or **Table Permissions** section

**Configure permissions for each table:**

| Table | Create | Read | Update | Delete |
|-------|--------|------|--------|--------|
| blog_posts | ✓ | ✓ | ✓ | ✗ |
| authors | ✗ | ✓ | ✓ (own only) | ✗ |
| categories | ✗ | ✓ | ✗ | ✗ |
| tags | ✓ | ✓ | ✓ | ✗ |
| comments | ✓ | ✓ | ✓ | ✓ |

3. Save configuration
4. Screenshot: `screenshots/blog-editor-permissions.png`

#### Activity 3.2: Configure Blog Viewer Permissions (15 min)

| Table | Create | Read | Update | Delete |
|-------|--------|------|--------|--------|
| blog_posts | ✗ | ✓ | ✗ | ✗ |
| authors | ✗ | ✓ | ✗ | ✗ |
| categories | ✗ | ✓ | ✗ | ✗ |
| tags | ✗ | ✓ | ✗ | ✗ |
| comments | ✗ | ✓ | ✗ | ✗ |

Apply and save.

#### Activity 3.3: Configure Blog Admin Permissions (10 min)

Grant **Full Access** to all blog-related tables.

#### Activity 3.4: Configure Comment Moderator Permissions (15 min)

| Table | Create | Read | Update | Delete |
|-------|--------|------|--------|--------|
| blog_posts | ✗ | ✓ | ✗ | ✗ |
| comments | ✗ | ✓ | ✓ | ✓ |
| authors | ✗ | ✓ | ✗ | ✗ |

Apply and save.

## 🍽️ Lunch Break (30-60 min)

## 📚 Afternoon Session (2-2.5 hours)

### Part 4: Field-Level Permissions (1 hour)

#### Activity 4.1: Understanding Field Permissions (15 min)

**Concept:** Sometimes users can see a record but not all fields.

Examples:
- Editors see draft posts but not author email addresses
- Viewers see published posts but not view counts
- Moderators see comment content but not IP addresses

Document in `field-permissions.md`: 3 scenarios where field-level permissions are useful

#### Activity 4.2: Configure Field Permissions for Blog Editor (20 min)

For the **blog_posts** table, Blog Editor role:
- Can see: all fields
- Can edit: title, content, tags, category
- Cannot edit: id, views, published_date
- Cannot see: (nothing hidden)

Configure these field permissions and document the process.

#### Activity 4.3: Configure Field Permissions for Blog Viewer (15 min)

For the **blog_posts** table, Blog Viewer role:
- Can see: id, title, content, published_date, category, tags, author
- Cannot see: views, internal notes (if you added this field)
- Cannot edit: any fields (read-only)

Screenshot: `screenshots/viewer-field-permissions.png`

#### Activity 4.4: Test Field Permissions (10 min)

Document in `field-permissions.md`:
1. What happens when a user tries to edit a forbidden field?
2. Can hidden fields be accessed via the API?
3. How do you hide sensitive data?

### Part 5: Creating and Testing Users (1-1.5 hours)

#### Activity 5.1: Create Test Users (30 min)

Create 4 test users:

**User 1: Editor**
```
Email: editor@blogtest.com
Password: TestPassword123!
First Name: Emma
Last Name: Editor
Role: Blog Editor
Status: Active
```

**User 2: Viewer**
```
Email: viewer@blogtest.com
Password: TestPassword123!
First Name: Victor
Last Name: Viewer
Role: Blog Viewer
Status: Active
```

**User 3: Admin**
```
Email: admin@blogtest.com
Password: TestPassword123!
First Name: Alice
Last Name: Admin
Role: Blog Admin
Status: Active
```

**User 4: Moderator**
```
Email: moderator@blogtest.com
Password: TestPassword123!
First Name: Mike
Last Name: Moderator
Role: Comment Moderator
Status: Active
```

Screenshot: `screenshots/test-users.png`

#### Activity 5.2: Test Permission Scenarios (45 min)

For each user, test and document in `permission-tests.md`:

**Test with Blog Editor (editor@blogtest.com):**
1. ✓ Can they create a new blog post?
2. ✓ Can they edit an existing blog post?
3. ✗ Can they delete a blog post?
4. ✗ Can they create a new author?
5. ✓ Can they add tags to a post?

**Test with Blog Viewer (viewer@blogtest.com):**
1. ✗ Can they create a blog post?
2. ✓ Can they view blog posts?
3. ✗ Can they edit comments?
4. ✗ Can they see the users section?

**Test with Comment Moderator (moderator@blogtest.com):**
1. ✗ Can they edit blog posts?
2. ✓ Can they view comments?
3. ✓ Can they delete comments?
4. ✓ Can they approve/reject comments?

**Test with Blog Admin (admin@blogtest.com):**
1. ✓ Can they do everything?

**Document each test:**
- What you tested
- Expected result
- Actual result
- Screenshot if relevant

#### Activity 5.3: Document Permission Matrix (15 min)

Create a complete permissions matrix in `permission-matrix.md`:

```
| Action | Blog Admin | Blog Editor | Comment Moderator | Blog Viewer |
|--------|-----------|-------------|-------------------|-------------|
| Create Post | ✓ | ✓ | ✗ | ✗ |
| Edit Post | ✓ | ✓ | ✗ | ✗ |
| Delete Post | ✓ | ✗ | ✗ | ✗ |
| View Posts | ✓ | ✓ | ✓ | ✓ |
| ... (complete this) |
```

### Part 6: Advanced Permission Scenarios (30 min)

#### Activity 6.1: Custom Permission Rules (15 min)

**Scenario:** Authors should only edit their own blog posts.

Research and document in `advanced-permissions.md`:
1. How can you implement "edit own items only"?
2. Does Ucode support custom permission logic?
3. If yes, configure it for the Blog Editor role
4. If no, document how you would work around this

#### Activity 6.2: Public vs Private Access (15 min)

**Scenario:** Some blog posts should be visible to everyone, others only to logged-in users.

Questions to answer in `advanced-permissions.md`:
1. How would you implement this in Ucode?
2. Can you have a "published" flag that affects permissions?
3. Can anonymous users access Ucode data?
4. Document your approach

## ✅ End of Day Checklist

- [ ] Created 4 custom roles
- [ ] Configured table-level permissions for all roles
- [ ] Configured field-level permissions
- [ ] Created 4 test users
- [ ] Tested all permission scenarios
- [ ] Documented permission matrix
- [ ] Explored advanced permission features
- [ ] Completed all documentation

## 📝 Deliverables

1. **security-concepts.md** - Security concepts and explanations
2. **current-setup.md** - Analysis of default setup
3. **roles-created.md** - Documentation of all roles
4. **field-permissions.md** - Field-level permission notes
5. **permission-tests.md** - Test results for each user
6. **permission-matrix.md** - Complete permission matrix
7. **advanced-permissions.md** - Advanced scenarios
8. **screenshots/** - All screenshots

## 🚀 Bonus Challenges

1. **Row-Level Security**: Research if Ucode supports filtering data based on user (e.g., users only see their own data)
2. **API Permissions**: Do API permissions differ from UI permissions?
3. **Token-Based Access**: Create an API token with limited permissions
4. **Audit Logging**: Check if Ucode logs permission changes and access attempts
5. **Password Policies**: Configure password requirements (length, complexity, expiration)

## 💡 Security Best Practices

Document in `best-practices.md`:

1. **Principle of Least Privilege**: Always give minimum necessary permissions
2. **Role Hierarchy**: Organize roles from least to most privileged
3. **Regular Audits**: Review permissions periodically
4. **Test Thoroughly**: Always test permissions before going to production
5. **Document Everything**: Keep clear documentation of who can do what
6. **Separate Concerns**: Don't mix responsibilities in a single role
7. **Disable Unused Accounts**: Deactivate accounts when not needed
8. **Strong Passwords**: Enforce password policies

## 📊 Self-Assessment Questions

Answer in `self-assessment.md`:

1. What's the difference between a role and a permission?
2. Why use RBAC instead of permission per user?
3. When would you need field-level permissions?
4. How do you test if permissions are working correctly?
5. What security risks exist if permissions are misconfigured?
6. How would you handle a user who needs permissions from two roles?

## ❓ Common Issues

**User can't log in?**
- Check if account is active
- Verify credentials
- Check if account is locked

**Permissions not applying?**
- Refresh the page
- Log out and log back in
- Check role assignment
- Verify permission configuration was saved

**User sees more than they should?**
- Review role permissions
- Check for inherited permissions
- Verify no conflicting rules

## 📖 Resources

- [Ucode Table Permissions](https://ucode.gitbook.io/ucode-docs/user-guide/table-permission)
- [Ucode User Guide](https://ucode.gitbook.io/ucode-docs/user-guide)

---

**Previous:** [Day 2 - Data Modeling](../day2-data-modeling/)  
**Next:** [Day 4 - API Integration](../day4-api/)

