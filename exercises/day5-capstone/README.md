# Day 5: Capstone Project - Build a Task Management System

## ⏱️ Duration: 6-8 hours

## 🎯 Project Overview

Build a complete **Task Management System** using everything you've learned. This system will allow teams to create projects, assign tasks, track progress, and collaborate.

## 🎯 Learning Objectives

By the end of Day 5, you will:
- Apply all concepts learned during the week
- Design a complete data model from scratch
- Implement user roles and permissions
- Create a functional API
- Build custom views and dashboards
- Present your work professionally

## 📋 Project Requirements

### Core Features (Must Have)

1. **Project Management**
   - Create, edit, delete projects
   - Each project has: name, description, start date, end date, status
   - Projects can have multiple tasks

2. **Task Management**
   - Create, edit, delete tasks
   - Each task has: title, description, priority, status, due date, assignee
   - Tasks belong to a project
   - Tasks can have comments

3. **User Management**
   - Different roles: Admin, Project Manager, Team Member, Guest
   - Appropriate permissions for each role

4. **Comments & Collaboration**
   - Users can comment on tasks
   - Track who made what comment and when

5. **API Access**
   - Full CRUD via REST API
   - Proper authentication

### Bonus Features (Nice to Have)

- Task dependencies (task B can't start until task A is done)
- File attachments for tasks
- Task tags/labels
- Time tracking
- Activity log/audit trail
- Email notifications (design only)
- Task templates

## 📅 Project Schedule

### Phase 1: Planning (1 hour) - 9:00 AM

#### Activity 1.1: Requirements Analysis (20 min)

Document in `project-plan.md`:

1. **User Stories** - Write at least 10 user stories:
   ```
   As a [role], I want to [action], so that [benefit]
   
   Example:
   As a Project Manager, I want to create new projects, 
   so that I can organize work into logical groups.
   ```

2. **Success Criteria** - What does "done" look like?

#### Activity 1.2: Data Model Design (40 min)

Design your database schema:

**Tables to create:**
1. `projects`
2. `tasks`
3. `users` (may already exist)
4. `task_comments`
5. `task_status` (lookup table)
6. `task_priority` (lookup table)
7. `project_members` (junction table)

**Create a diagram** showing all tables and relationships:
- Draw on paper and photograph, or
- Use a digital tool
- Save as: `diagrams/data-model.png`

**Document in `data-model.md`:**
- Each table's purpose
- All fields for each table
- All relationships
- Justification for your design decisions

### Phase 2: Database Implementation (1.5 hours) - 10:00 AM

#### Activity 2.1: Create Lookup Tables (15 min)

**Table: task_status**
```
Fields:
- id (auto-increment)
- name (string): "To Do", "In Progress", "In Review", "Done", "Blocked"
- slug (string): "todo", "in_progress", "in_review", "done", "blocked"
- color (string): hex color codes for UI
```

**Table: task_priority**
```
Fields:
- id (auto-increment)
- name (string): "Low", "Medium", "High", "Urgent"
- slug (string): "low", "medium", "high", "urgent"
- order (integer): 1, 2, 3, 4
```

Create both tables and populate with data.

#### Activity 2.2: Create Core Tables (45 min)

**Table: projects**
```
Fields:
- id
- name (string, required)
- description (text)
- start_date (date)
- end_date (date)
- status (many-to-one → task_status)
- created_by (many-to-one → users)
- created_at (datetime)
- updated_at (datetime)
```

**Table: tasks**
```
Fields:
- id
- title (string, required)
- description (text)
- project_id (many-to-one → projects, required)
- assigned_to (many-to-one → users)
- created_by (many-to-one → users)
- status_id (many-to-one → task_status)
- priority_id (many-to-one → task_priority)
- due_date (date)
- estimated_hours (decimal)
- completed_at (datetime)
- created_at (datetime)
- updated_at (datetime)
```

**Table: task_comments**
```
Fields:
- id
- task_id (many-to-one → tasks, required)
- user_id (many-to-one → users, required)
- comment (text, required)
- created_at (datetime)
- updated_at (datetime)
```

Create all tables with proper field types and relationships.

#### Activity 2.3: Create Junction Table (15 min)

**Table: project_members**
```
Fields:
- id
- project_id (many-to-one → projects)
- user_id (many-to-one → users)
- role (string): "Owner", "Member", "Viewer"
- added_at (datetime)
```

This allows multiple users per project.

#### Activity 2.4: Verify Data Model (15 min)

1. Review all tables created
2. Verify all relationships work
3. Take screenshots of each table configuration
4. Document any issues encountered

### Phase 3: Sample Data Creation (30 min) - 11:45 AM

#### Activity 3.1: Create Sample Data (30 min)

**Create at least:**
- 3 projects
- 15 tasks (distributed across projects)
- 5 users (if not already existing)
- 10 comments
- Assign tasks to different users
- Use various statuses and priorities

**Example Project 1: Website Redesign**
- Tasks: Design mockups, Develop homepage, Create blog section, etc.

**Example Project 2: Mobile App Development**
- Tasks: User research, Wireframes, Backend API, Frontend development, etc.

**Example Project 3: Marketing Campaign**
- Tasks: Content creation, Social media posts, Email campaign, etc.

Document your sample data in `sample-data.md`

## 🍽️ Lunch Break (30-60 min) - 12:15 PM

### Phase 4: Roles & Permissions (1.5 hours) - 1:00 PM

#### Activity 4.1: Define Roles (30 min)

Create these roles with appropriate permissions:

**Role 1: Task Admin**
- Full access to everything
- Can manage all projects and tasks
- Can manage users

**Role 2: Project Manager**
- Can create/edit/delete projects
- Can create/edit/delete tasks in their projects
- Can assign tasks
- Can view all comments
- Cannot manage users

**Role 3: Team Member**
- Can view all projects
- Can edit tasks assigned to them
- Can create comments
- Can change status of their tasks
- Cannot delete projects or tasks

**Role 4: Guest**
- Read-only access to projects and tasks
- Can view comments
- Cannot edit anything

Document each role in `roles-documentation.md`

#### Activity 4.2: Configure Permissions (45 min)

For each role, configure table permissions:

| Table | Task Admin | Project Manager | Team Member | Guest |
|-------|-----------|-----------------|-------------|-------|
| projects | Full | CRUD | Read | Read |
| tasks | Full | CRUD | Read + Update (own) | Read |
| task_comments | Full | CRUD | CRUD | Read |
| task_status | Full | Read | Read | Read |
| task_priority | Full | Read | Read | Read |

Implement field-level permissions where appropriate.

#### Activity 4.3: Test Permissions (15 min)

Create test users for each role and verify permissions work correctly.

### Phase 5: Views & Dashboard (1 hour) - 2:30 PM

#### Activity 5.1: Create Task Views (30 min)

**View 1: My Tasks**
- Filter: Tasks assigned to current user
- Sort: Due date ascending
- Group by: Status

**View 2: Project Tasks** (for each project)
- Filter: By project
- Sort: Priority, then due date
- Layout: Kanban board by status

**View 3: Overdue Tasks**
- Filter: Due date < today AND status != done
- Sort: Due date
- Highlight in red

**View 4: All Tasks Table**
- Show all tasks
- Columns: Title, Project, Assignee, Status, Priority, Due Date
- Enable sorting and filtering

#### Activity 5.2: Create Dashboard (30 min)

Build a project dashboard showing:

**Widgets:**
1. **Tasks Summary**
   - Total tasks
   - By status (To Do, In Progress, Done)
   - Overdue count

2. **My Tasks Today**
   - Tasks due today assigned to me

3. **Project Progress**
   - For each project: X of Y tasks completed

4. **Recent Activity**
   - Latest comments
   - Recently updated tasks

5. **Team Workload**
   - Number of tasks per team member

Screenshot: `screenshots/dashboard.png`

### Phase 6: API Testing & Documentation (1 hour) - 3:30 PM

#### Activity 6.1: Create API Collection (30 min)

Create a complete API collection with these requests:

**Projects:**
```
GET /items/projects
GET /items/projects/{id}
POST /items/projects
PATCH /items/projects/{id}
DELETE /items/projects/{id}
```

**Tasks:**
```
GET /items/tasks
GET /items/tasks?filter[project_id][_eq]=1
GET /items/tasks?filter[assigned_to][_eq]=2
POST /items/tasks
PATCH /items/tasks/{id}
DELETE /items/tasks/{id}
```

**Comments:**
```
GET /items/task_comments?filter[task_id][_eq]=1
POST /items/task_comments
```

**Custom Queries:**
```
GET /items/tasks?filter[status_id][_neq]=4&filter[due_date][_lt]=$NOW
GET /items/projects/{id}?fields=*,tasks.*,tasks.assigned_to.*
```

#### Activity 6.2: API Documentation (30 min)

Document in `api-documentation.md`:

For each endpoint:
- HTTP Method and URL
- Purpose
- Request body (if applicable)
- Response format
- Example request
- Example response
- Possible errors

Export your API collection and save as `api-collection.json`

### Phase 7: Final Testing (30 min) - 4:30 PM

#### Activity 7.1: End-to-End Testing (30 min)

Test complete workflows:

**Workflow 1: Create a Project**
1. Create project as Project Manager
2. Add tasks to the project
3. Assign tasks to team members
4. Team members update status
5. Add comments
6. Mark tasks as complete

**Workflow 2: Track Progress**
1. View dashboard
2. Check overdue tasks
3. Filter tasks by priority
4. View project progress

**Workflow 3: Collaboration**
1. Add comment to task
2. Reassign task
3. Change priority
4. Update status

Document any issues in `testing-notes.md`

### Phase 8: Presentation Preparation (45 min) - 5:00 PM

#### Activity 8.1: Create Presentation (45 min)

Prepare a 10-minute presentation covering:

1. **Introduction** (1 min)
   - Project overview
   - Goals

2. **Data Model** (2 min)
   - Show your diagram
   - Explain key relationships
   - Justify decisions

3. **Features Demo** (4 min)
   - Live demo of creating project and tasks
   - Show different views
   - Demonstrate dashboard
   - Show API working

4. **Roles & Permissions** (2 min)
   - Explain your role structure
   - Demo permission differences

5. **Challenges & Learning** (1 min)
   - What was difficult?
   - What did you learn?

Create presentation slides or demo script in `presentation-notes.md`

## ✅ Final Checklist

### Data Model
- [ ] All required tables created
- [ ] All fields configured correctly
- [ ] All relationships working
- [ ] Sample data populated

### Roles & Permissions
- [ ] 4 roles created
- [ ] Permissions configured for all roles
- [ ] Tested with different users

### Views & Dashboard
- [ ] At least 4 views created
- [ ] Dashboard with 5+ widgets
- [ ] Views are useful and functional

### API
- [ ] All CRUD operations tested
- [ ] API collection created
- [ ] API documentation written
- [ ] Authentication working

### Documentation
- [ ] Project plan completed
- [ ] Data model documented
- [ ] API documented
- [ ] Presentation prepared

### Quality
- [ ] No test/junk data remaining
- [ ] Everything works as expected
- [ ] Screenshots captured
- [ ] Code/configurations are clean

## 📝 Final Deliverables

Submit a complete package including:

1. **Documentation**
   - `project-plan.md`
   - `data-model.md`
   - `roles-documentation.md`
   - `api-documentation.md`
   - `testing-notes.md`
   - `presentation-notes.md`
   - `reflection.md` (see below)

2. **Diagrams**
   - `diagrams/data-model.png`
   - `diagrams/workflow.png` (optional)

3. **Screenshots**
   - All table structures
   - Dashboard
   - Each view
   - API tests
   - Permission configurations

4. **API Collection**
   - `api-collection.json`

5. **Reflection Document** (`reflection.md`)
   - What went well?
   - What was challenging?
   - What would you do differently?
   - What did you learn this week?
   - How will you use Ucode in the future?

## 🎤 Presentation (30-60 min) - 5:45 PM

Each intern presents their project (10 minutes + 5 minutes Q&A)

**Presentation Tips:**
- Start strong with project overview
- Show, don't just tell (live demo!)
- Explain your design decisions
- Be honest about challenges
- Highlight what you learned
- End with next steps or improvements

## 🚀 Bonus Enhancements

If you finish early or want to go further:

1. **Build a Simple Frontend**
   - Create an HTML page that uses your API
   - Display projects and tasks
   - Allow task updates

2. **Custom Endpoint**
   - Create `/api/projects/{id}/progress` endpoint
   - Returns completion percentage and statistics

3. **Webhooks**
   - Set up a webhook that triggers when task status changes
   - Log to a webhook testing service

4. **Advanced Filters**
   - Create a "My Week" view showing all tasks due this week
   - Create "Team Performance" view with statistics

5. **Export Feature**
   - Enable exporting projects to CSV/PDF

## 💡 Tips for Success

1. **Time Management**: Stick to the schedule
2. **Start Simple**: Get basics working first
3. **Test Early**: Don't wait until the end to test
4. **Document As You Go**: Don't leave it all for the end
5. **Ask for Help**: If stuck for >15 minutes, ask mentor
6. **Be Creative**: Add your own ideas!
7. **Focus on Learning**: Perfection isn't required

## 📊 Evaluation Criteria

Your project will be evaluated on:

1. **Completeness** (30%)
   - All required features implemented
   - All deliverables submitted

2. **Quality** (25%)
   - Data model is well-designed
   - Permissions are appropriate
   - API works correctly
   - Clean, organized work

3. **Understanding** (25%)
   - Can explain your decisions
   - Understands concepts applied
   - Can answer questions

4. **Presentation** (20%)
   - Clear and organized
   - Good demonstration
   - Professional delivery

## ❓ Common Challenges

**Running out of time?**
- Focus on core features first
- Skip bonus features
- Simplify if needed

**Data model not working?**
- Review relationship types
- Check foreign key configurations
- Ask for help

**API not working?**
- Verify authentication
- Check permissions
- Review endpoint syntax

**Can't finish everything?**
- Submit what you have
- Document what's incomplete
- Explain what you would do differently

## 📖 Resources

- All previous day's exercises
- [Ucode Documentation](https://ucode.gitbook.io/ucode-docs)
- Your notes from Days 1-4

---

## 🎉 Congratulations!

By completing this capstone project, you've demonstrated mastery of:
- Database design and relationships
- User management and security
- REST API integration
- UI/UX with views and dashboards
- Professional documentation
- Project presentation

**You're now ready to build real applications with Ucode!**

---

**Previous:** [Day 4 - API Integration](../day4-api/)  
**End of Week 1 Training**

