# Day 1: Foundations - Getting Started & Basic CRUD

## ⏱️ Duration: 4-5 hours

## 🎯 Learning Objectives

By the end of Day 1, you will:
- Understand what Ucode is and how it works
- Navigate the Ucode Admin Panel confidently
- Perform all CRUD operations (Create, Read, Update, Delete)
- Understand database introspection
- Work with data validation

## 📚 Morning Session (2-2.5 hours)

### Part 1: Introduction & Setup (45 min)

#### Activity 1.1: Understanding Ucode (15 min)
**Read and understand:**
- What is Ucode? (Backend as a Service)
- Database introspection concept
- How Ucode differs from traditional CMS

**Documentation:** Write 3-5 bullet points in `notes.md` about what makes Ucode unique.

#### Activity 1.2: First Login (15 min)
1. Access your Ucode instance (URL provided by mentor)
2. Log in with your credentials
3. Take a screenshot of the Admin Panel: `screenshots/dashboard.png`
4. Identify these elements:
   - Main navigation menu
   - Current database name
   - Your user role

#### Activity 1.3: Explore the Interface (15 min)
Navigate through each section and document in `notes.md`:
- **Menu**: What can you do here?
- **Tables**: How many tables exist?
- **Items**: What are items?
- **Settings**: What settings are available?

### Part 2: Understanding Your Database (1-1.5 hours)

#### Activity 2.1: Database Introspection Demo (30 min)
Your mentor will demonstrate:
1. How Ucode connects to a database
2. How tables are automatically detected
3. How field types are mapped

**Your Task:** In `notes.md`, draw a simple diagram showing:
```
[Your Database] ←→ [Ucode Layer] ←→ [Admin Panel / API]
```
Explain each component in 1-2 sentences.

#### Activity 2.2: Explore Existing Tables (30 min)
1. Navigate to the **Tables** section
2. Pick the `products` table (or assigned table)
3. Document in `table-analysis.md`:
   - Table name
   - Number of fields
   - Field types (text, number, boolean, date, etc.)
   - Required vs optional fields
   - Any relationships to other tables

#### Activity 2.3: View Sample Data (30 min)
1. Click into the `products` table
2. View existing items (records)
3. Click on one item to see its detail view
4. Screenshot: `screenshots/product-detail.png`
5. Answer in `notes.md`: How is data organized? What information can you see?

## 🍽️ Lunch Break (30-60 min)

## 📚 Afternoon Session (2-2.5 hours)

### Part 3: CREATE Operations (45 min)

#### Activity 3.1: Create Your First Record (15 min)
1. Go to the `products` table items view
2. Click "Create New Item" or the "+" button
3. Create a product with these details:
   ```
   Name: Wireless Mouse Pro
   Price: 49.99
   Category: Electronics
   In Stock: Yes
   Description: Ergonomic wireless mouse with 6 buttons
   ```
4. Click Save
5. Screenshot: `screenshots/first-create.png`

#### Activity 3.2: Create 5 More Records (20 min)
Create diverse products - be creative! Examples:
- Office chair
- Coffee maker
- Desk lamp
- USB cable
- Monitor stand

Document all created records in `created-items.md`

#### Activity 3.3: Test Validation (10 min)
Experiment with errors:
1. Try creating a record WITHOUT a required field → What happens?
2. Try entering text in a number field → What happens?
3. Try entering a negative price → What happens?

Document findings in `validation-notes.md`

### Part 4: READ Operations (30 min)

#### Activity 4.1: View All Records (10 min)
1. Look at all products in list view
2. Notice the layout and columns
3. Screenshot: `screenshots/list-view.png`

#### Activity 4.2: Search and Filter (20 min)
**Search Tasks:**
1. Search for "Mouse" → How many results?
2. Search for "Pro" → How many results?
3. Search by category → Document results

**Filter Tasks:**
1. Filter: `In Stock = Yes` → Screenshot results
2. Filter: `Price > 100` → How many?
3. Filter: `Category = Electronics` → Screenshot results

Document all results in `search-results.md`

### Part 5: UPDATE Operations (30 min)

#### Activity 5.1: Edit Single Record (15 min)
1. Select your "Wireless Mouse Pro" product
2. Click Edit
3. Update:
   - Price: 39.99 (discount!)
   - Description: Add "Now on sale!"
   - In Stock: Update if needed
4. Save changes
5. Verify changes were saved

#### Activity 5.2: Bulk Edit (if available) (15 min)
1. Select 3 products using checkboxes
2. Look for bulk edit option
3. Try updating a common field
4. Document the process in `update-notes.md`
5. If bulk edit isn't available, edit 3 items individually

### Part 6: DELETE Operations (30 min)

#### Activity 6.1: Safe Delete Practice (15 min)
1. Create a test record: "DELETE ME - Test Product"
2. Find the delete button/option
3. Attempt to delete
4. Questions to answer in `delete-notes.md`:
   - Did you get a confirmation?
   - Was the item immediately removed?
   - Can you find it in a trash/archive?

#### Activity 6.2: Understand Delete Behavior (15 min)
Research and document:
1. Does Ucode use "soft delete" (archive) or "hard delete" (permanent)?
2. Is there a way to recover deleted items?
3. What happens if you delete an item that's referenced elsewhere?

## ✅ End of Day Checklist

- [ ] Successfully logged into Ucode
- [ ] Explored all main navigation sections
- [ ] Understood database introspection concept
- [ ] Created at least 6 new records
- [ ] Tested data validation
- [ ] Performed search and filter operations
- [ ] Updated existing records
- [ ] Deleted records safely
- [ ] Completed all documentation files

## 📝 Deliverables

Submit these files to your mentor:

1. **notes.md** - Your learning notes and observations
2. **table-analysis.md** - Analysis of the products table
3. **created-items.md** - List of all items you created
4. **validation-notes.md** - Validation test results
5. **search-results.md** - Search and filter experiments
6. **update-notes.md** - Update operation notes
7. **delete-notes.md** - Delete behavior findings
8. **screenshots/** folder with all screenshots

## 🚀 Bonus Challenges (If Time Permits)

1. **Speed Challenge**: Time how fast you can create 10 records
2. **Keyboard Shortcuts**: Discover any keyboard shortcuts
3. **Data Export**: Try exporting data to CSV/JSON
4. **Advanced Search**: Look for advanced search options

## 💡 Key Takeaways

Before ending the day, write in `notes.md`:
1. What was the most useful thing you learned today?
2. What was most challenging?
3. What do you want to learn more about?

## ❓ Daily Q&A

**Q: What if I make a mistake?**  
A: Most actions in Ucode can be undone or corrected. This is a learning environment!

**Q: How do I know if my CRUD operations worked?**  
A: Always verify by viewing the record after creating/updating it.

**Q: Can I delete the wrong thing?**  
A: Be careful with delete operations. Always read confirmation dialogs. Ask mentor if unsure.

## 📖 Resources

- [Ucode Overview](https://ucode.gitbook.io/ucode-docs/user-guide/overview)
- [Ucode Items Guide](https://ucode.gitbook.io/ucode-docs/user-guide/items)
- [Ucode Tables](https://ucode.gitbook.io/ucode-docs/user-guide/table)

---

**End of Day 1**  
**Next:** [Day 2 - Data Modeling](../day2-data-modeling/)
