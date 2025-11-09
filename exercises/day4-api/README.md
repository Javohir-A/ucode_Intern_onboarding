# Day 4: API Integration - REST API, Views & Custom Functions

## ⏱️ Duration: 4-5 hours

## 🎯 Learning Objectives

By the end of Day 4, you will:
- Understand Ucode's REST API structure
- Authenticate API requests
- Perform CRUD operations via API
- Create custom views and layouts
- Build a dashboard
- Understand API best practices
- (Optional) Create a custom endpoint

## 📋 Prerequisites

- API testing tool (Postman, Insomnia, or curl)
- Basic understanding of HTTP methods
- Your Ucode instance URL and credentials

## 📚 Morning Session (2-2.5 hours)

### Part 1: Understanding Ucode API (1 hour)

#### Activity 1.1: API Architecture (20 min)

Read and document in `api-concepts.md`:

**Key Concepts:**
- **REST**: Representational State Transfer
- **Endpoints**: URLs that expose data/functionality
- **HTTP Methods**: GET, POST, PUT, PATCH, DELETE
- **Authentication**: How to prove you're authorized
- **Response Formats**: JSON structure

**Answer these questions:**
1. What is a REST API?
2. What are the main HTTP methods and what do they do?
3. Why do APIs use JSON?
4. What's the difference between PUT and PATCH?

#### Activity 1.2: Explore API Documentation (20 min)

1. Find Ucode's API documentation
2. Identify the base URL for your instance
3. Document in `api-endpoints.md`:
   - Base URL format
   - Authentication method
   - Common endpoints
   - Response format examples

Example endpoints:
```
GET    /items/{table}           - Get all items
GET    /items/{table}/{id}      - Get single item
POST   /items/{table}           - Create item
PATCH  /items/{table}/{id}      - Update item
DELETE /items/{table}/{id}      - Delete item
```

#### Activity 1.3: Set Up API Testing Tool (20 min)

**Install and configure** (choose one):
- Postman (recommended for beginners)
- Insomnia
- curl (command line)
- Thunder Client (VS Code extension)

**Set up your environment:**
1. Create a new collection/workspace: "Ucode Blog API"
2. Set base URL variable: `{{base_url}}`
3. Screenshot: `screenshots/api-tool-setup.png`

### Part 2: API Authentication (1-1.5 hours)

#### Activity 2.1: Get Access Token (30 min)

**Method 1: Static Token**
1. Navigate to your Ucode user settings
2. Generate a new API token
3. Copy the token
4. Save it securely (never commit to git!)

**Method 2: Login Endpoint**
1. Create a POST request to `/auth/login`
2. Body:
```json
{
  "email": "your-email@example.com",
  "password": "your-password"
}
```
3. Extract the access token from response
4. Document the response structure in `authentication.md`

Screenshot: `screenshots/login-request.png`

#### Activity 2.2: Configure Authentication (15 min)

**Set up authentication in your API tool:**

1. For Bearer Token authentication:
   - Header: `Authorization: Bearer YOUR_TOKEN`
   
2. Create an environment variable: `{{access_token}}`

3. Configure it for all requests

4. Test with a simple GET request:
```
GET {{base_url}}/items/blog_posts
```

Document the response in `authentication.md`

#### Activity 2.3: Test Authentication (15 min)

Make these test requests and document results:

1. **Without token**: Should get 401 Unauthorized
2. **With invalid token**: Should get 401 or 403
3. **With valid token**: Should get data

Document all responses in `authentication.md`

### Part 3: CRUD via API (1 hour)

#### Activity 3.1: READ Operations (GET) (20 min)

**Test 1: Get all blog posts**
```
GET {{base_url}}/items/blog_posts
```

**Test 2: Get single blog post**
```
GET {{base_url}}/items/blog_posts/1
```

**Test 3: Filter blog posts**
```
GET {{base_url}}/items/blog_posts?filter[published][_eq]=true
```

**Test 4: Get with relationships**
```
GET {{base_url}}/items/blog_posts?fields=*,author.*,tags.*
```

Document all responses in `api-crud-tests.md`

#### Activity 3.2: CREATE Operations (POST) (15 min)

**Create a new blog post via API:**

```
POST {{base_url}}/items/blog_posts
Content-Type: application/json

{
  "title": "My First API Post",
  "content": "This post was created via API!",
  "author_id": 1,
  "published": false
}
```

Document:
- Request body sent
- Response received
- Status code
- Created item ID

Screenshot: `screenshots/api-create.png`

#### Activity 3.3: UPDATE Operations (PATCH) (15 min)

**Update the post you just created:**

```
PATCH {{base_url}}/items/blog_posts/{id}
Content-Type: application/json

{
  "title": "My First API Post (Updated)",
  "published": true
}
```

Document the update response in `api-crud-tests.md`

#### Activity 3.4: DELETE Operations (DELETE) (10 min)

**Create a test post and delete it:**

```
DELETE {{base_url}}/items/blog_posts/{id}
```

Document:
- Which item was deleted
- Status code received
- Response body

## 🍽️ Lunch Break (30-60 min)

## 📚 Afternoon Session (2-2.5 hours)

### Part 4: Working with Views (1-1.5 hours)

#### Activity 4.1: Understanding Views (15 min)

**Concept:** Views are custom layouts for displaying and interacting with data.

Read Ucode Views documentation and answer in `views-concepts.md`:
1. What is a view?
2. What are the different view types? (Table, Card, Calendar, Map, etc.)
3. When would you use each type?

#### Activity 4.2: Create a Table View (20 min)

1. Navigate to **Views** section
2. Create a new view for `blog_posts`:
   ```
   Name: All Blog Posts
   Type: Table
   Table: blog_posts
   ```

3. Configure columns to display:
   - ID
   - Title
   - Author
   - Category
   - Published
   - Publish Date

4. Add sorting: Published Date (descending)
5. Screenshot: `screenshots/table-view.png`

#### Activity 4.3: Create a Card View (20 min)

Create a visual card layout for blog posts:

1. Create new view:
   ```
   Name: Blog Post Cards
   Type: Card/Grid
   Table: blog_posts
   ```

2. Configure card display:
   - Title as main heading
   - Content preview (first 150 chars)
   - Author and date as metadata
   - Tags as badges

3. Screenshot: `screenshots/card-view.png`

#### Activity 4.4: Create Filtered Views (20 min)

Create specialized views:

**View 1: Published Posts Only**
- Filter: `published = true`
- Sort: Published date (newest first)

**View 2: Draft Posts**
- Filter: `published = false`
- Sort: Updated at (newest first)

**View 3: My Posts** (if you implemented author-based filtering)
- Filter: Current user's posts only

Document each view's purpose in `views-documentation.md`

### Part 5: Building a Dashboard (45 min)

#### Activity 5.1: Create a Dashboard View (30 min)

Build an overview dashboard showing:

1. **Total Posts Count**
   - Published vs Draft
   
2. **Recent Posts** (last 5)
   - Title, author, date
   
3. **Comments Requiring Moderation**
   - Unapproved comments
   
4. **Popular Tags**
   - Most used tags

Document in `dashboard-design.md`:
- What widgets/sections you included
- What data each section shows
- Who would use this dashboard

Screenshot: `screenshots/dashboard.png`

#### Activity 5.2: Create Role-Specific Dashboards (15 min)

Design (document, don't necessarily implement) dashboards for:

1. **Blog Editor Dashboard**:
   - Their draft posts
   - Their published posts
   - Recent comments on their posts

2. **Comment Moderator Dashboard**:
   - Pending comments
   - Flagged comments
   - Comment statistics

Document these designs in `dashboard-design.md`

### Part 6: Advanced API Features (45 min)

#### Activity 6.1: Pagination (15 min)

Test pagination with large datasets:

```
GET {{base_url}}/items/blog_posts?limit=10&page=1
GET {{base_url}}/items/blog_posts?limit=10&page=2
```

Document in `advanced-api.md`:
- How to specify page size
- How to navigate pages
- How to get total count

#### Activity 6.2: Sorting and Filtering (15 min)

**Test various filters:**

```
# Posts with more than 100 views
GET /items/blog_posts?filter[views][_gt]=100

# Posts containing "tutorial" in title
GET /items/blog_posts?filter[title][_contains]=tutorial

# Posts by specific author
GET /items/blog_posts?filter[author_id][_eq]=1

# Multiple conditions
GET /items/blog_posts?filter[published][_eq]=true&filter[views][_gt]=50
```

**Test sorting:**
```
GET /items/blog_posts?sort=-views,-published_date
```

Document all filter operators in `advanced-api.md`

#### Activity 6.3: Batch Operations (15 min)

**Test batch create:**
```
POST {{base_url}}/items/tags
Content-Type: application/json

[
  {"name": "API", "slug": "api"},
  {"name": "Testing", "slug": "testing"},
  {"name": "Automation", "slug": "automation"}
]
```

**Test batch update:**
```
PATCH {{base_url}}/items/blog_posts
Content-Type: application/json

{
  "keys": [1, 2, 3],
  "data": {
    "published": true
  }
}
```

Document batch capabilities in `advanced-api.md`

## ✅ End of Day Checklist

- [ ] Set up API testing tool
- [ ] Successfully authenticated API requests
- [ ] Performed all CRUD operations via API
- [ ] Created at least 3 different views
- [ ] Built a dashboard
- [ ] Tested pagination, filtering, and sorting
- [ ] Tested batch operations
- [ ] Documented all API endpoints used
- [ ] Saved API collection/workspace

## 📝 Deliverables

1. **api-concepts.md** - API concepts and architecture
2. **api-endpoints.md** - Documentation of all endpoints
3. **authentication.md** - Authentication setup and testing
4. **api-crud-tests.md** - All CRUD operation tests
5. **views-concepts.md** - Views explanation
6. **views-documentation.md** - All views created
7. **dashboard-design.md** - Dashboard specifications
8. **advanced-api.md** - Advanced API features
9. **screenshots/** - All screenshots
10. **api-collection.json** - Exported API collection from your tool

## 🚀 Bonus Challenges

1. **Create a Simple Web Page**: Build an HTML page that fetches and displays blog posts using the API

2. **Error Handling**: Test and document all error responses (400, 401, 403, 404, 500)

3. **Rate Limiting**: Research if Ucode has rate limits and how to handle them

4. **Webhooks**: Investigate if Ucode supports webhooks (notifications when data changes)

5. **GraphQL**: Check if Ucode supports GraphQL in addition to REST

6. **Custom Endpoint**: If time permits, create a custom endpoint (e.g., `/api/posts/popular` that returns top viewed posts)

## 💡 API Best Practices

Document in `api-best-practices.md`:

1. **Always use HTTPS** in production
2. **Never expose tokens** in client-side code
3. **Handle errors gracefully** - check status codes
4. **Cache responses** when appropriate
5. **Use pagination** for large datasets
6. **Validate data** before sending to API
7. **Rate limit your requests** to avoid overwhelming server
8. **Use specific fields** - don't request more data than needed
9. **Keep tokens secure** - rotate regularly
10. **Log API errors** for debugging

## 📊 Self-Assessment Questions

Answer in `self-assessment.md`:

1. What's the difference between PUT and PATCH?
2. What HTTP status code means "success" for a POST request?
3. How do you include related data in an API response?
4. Why is authentication important for APIs?
5. What's the advantage of views over direct table access?
6. How would you handle API errors in your application?
7. What's pagination and why is it necessary?

## ❓ Common Issues

**401 Unauthorized?**
- Check your token is valid
- Verify Authorization header format
- Token might have expired

**Can't create/update items?**
- Check permission for your API user
- Verify JSON syntax
- Check required fields

**Empty response?**
- Data might not exist
- Check filters are correct
- Verify table name spelling

**CORS errors?**
- This happens when testing from browser
- Use API tool or configure CORS properly

## 📖 Resources

- [Ucode API Reference](https://ucode.gitbook.io/ucode-docs/developer-guide/api-reference)
- [Ucode Views](https://ucode.gitbook.io/ucode-docs/user-guide/views)
- [REST API Best Practices](https://restfulapi.net/)

---

**Previous:** [Day 3 - Security & Access Control](../day3-security/)  
**Next:** [Day 5 - Capstone Project](../day5-capstone/)

