# Mentor Guide - Ucode Intern Onboarding Program

## 📋 Program Overview

This is a **1-week intensive onboarding program** designed to train new interns on Ucode, the Backend as a Service platform. The program takes interns from zero knowledge to being able to build complete applications.

**Duration:** 5 days (40-50 hours total)  
**Format:** Hands-on exercises with mentor support  
**Outcome:** Interns can independently build Ucode applications

---

## 🎯 Your Role as Mentor

### Key Responsibilities

1. **Prepare Environment** (Before Day 1)
   - Set up Ucode instance for interns
   - Create database with sample tables
   - Prepare user accounts
   - Test all access

2. **Daily Check-ins**
   - Morning: Explain the day's objectives (15 min)
   - Mid-day: Answer questions, unblock issues
   - End of day: Review deliverables, provide feedback (30 min)

3. **Provide Support**
   - Answer questions promptly
   - Help with technical issues
   - Guide without giving complete solutions
   - Encourage problem-solving

4. **Evaluate Progress**
   - Review daily deliverables
   - Assess understanding through Q&A
   - Provide constructive feedback
   - Document progress

---

## 🛠️ Pre-Program Setup (1-2 days before)

### 1. Ucode Instance Setup

**Create a dedicated training instance:**
- [ ] Provision a Ucode instance
- [ ] Choose a database (PostgreSQL recommended)
- [ ] Configure basic settings
- [ ] Test admin access

**URL:** _______________________________  
**Admin Credentials:** _______________________________

### 2. Database Preparation

**Create a starter database with:**

**Table: products** (for Day 1 CRUD practice)
```sql
- id (auto-increment)
- name (string)
- price (decimal)
- category (string)
- in_stock (boolean)
- description (text)
- created_at (datetime)
```

**Populate with 10-15 sample products**

### 3. User Account Creation

Create accounts for each intern:

**Template:**
```
Email: intern_[name]@yourdomain.com
Password: [secure temporary password]
Role: Start with limited role, they'll create more
```

**Send credentials securely** (not via email!)

### 4. Documentation Access

Ensure interns have access to:
- [ ] Ucode official documentation
- [ ] This GitHub repository
- [ ] API testing tool (Postman/Insomnia)
- [ ] Communication channel (Slack/Teams)

### 5. Mentor Preparation

- [ ] Review all 5 days of exercises
- [ ] Test each exercise yourself
- [ ] Prepare answers to common questions
- [ ] Set up regular check-in times
- [ ] Create feedback template

---

## 📅 Daily Breakdown & Mentor Notes

### Day 1: Foundations - Getting Started & Basic CRUD

**⏰ Time Commitment:** 6-7 hours  
**Difficulty:** Easy  
**Key Concepts:** Navigation, CRUD, Data validation

#### Morning Kickoff (15 min)
- Introduce the program and week schedule
- Explain what Ucode is and why it's useful
- Demo database introspection live
- Show them their login credentials
- Answer setup questions

#### What to Watch For
- Interns struggling to log in (credential issues)
- Confusion about database introspection concept
- Hesitation to experiment (encourage exploration!)
- Not taking good notes

#### Common Questions
**Q: "Can I break anything?"**  
A: No, this is a sandbox environment. Experiment freely!

**Q: "What if I delete something important?"**  
A: We can restore it. This is a learning environment.

**Q: "How is Ucode different from [other platform]?"**  
A: Focus on database introspection and no vendor lock-in.

#### End of Day Review (30 min)
Check their deliverables:
- [ ] Can they navigate the interface confidently?
- [ ] Do they understand CRUD operations?
- [ ] Have they created quality documentation?
- [ ] Screenshots are clear and relevant?

**Feedback Focus:** Encourage thoroughness in documentation.

---

### Day 2: Data Modeling - Tables, Fields & Relationships

**⏰ Time Commitment:** 6-7 hours  
**Difficulty:** Medium  
**Key Concepts:** Data modeling, Relationships, Schema design

#### Morning Kickoff (15 min)
- Review Day 1 learnings
- Explain entity-relationship concepts
- Draw a simple ER diagram on whiteboard
- Discuss real-world relationship examples

#### What to Watch For
- Confusion about One-to-Many vs Many-to-Many
- Incorrect foreign key configurations
- Not testing relationships with actual data
- Poor field type choices

#### Common Questions
**Q: "When do I use One-to-Many vs Many-to-Many?"**  
A: Ask: Can A have multiple B's AND can B have multiple A's? If both yes → Many-to-Many

**Q: "What's a junction table?"**  
A: A table that connects two tables in a Many-to-Many relationship

**Q: "My relationship isn't showing data. Why?"**  
A: Check: 1) Both tables have data, 2) Foreign key is set, 3) Records are actually related

#### Mid-Day Check-in (15 min)
- Review their data model diagram
- Validate their relationship choices
- Suggest improvements if needed

#### End of Day Review (30 min)
Check:
- [ ] Data model is properly normalized
- [ ] All relationships work correctly
- [ ] Sample data demonstrates relationships
- [ ] Diagram is clear and accurate

**Feedback Focus:** Data modeling thinking, not just technical execution.

---

### Day 3: Security - Users, Roles & Permissions

**⏰ Time Commitment:** 6-7 hours  
**Difficulty:** Medium-Hard  
**Key Concepts:** RBAC, Permissions, Security thinking

#### Morning Kickoff (15 min)
- Discuss principle of least privilege
- Explain role-based access control
- Give real-world security scenario examples
- Emphasize importance of testing

#### What to Watch For
- Giving too many permissions (insecure)
- Not testing thoroughly
- Confusion about role vs permission
- Not documenting permission decisions

#### Common Questions
**Q: "How do I test permissions without logging out?"**  
A: Use private/incognito browser windows for each test user

**Q: "Can a user have multiple roles?"**  
A: [Depends on Ucode implementation - clarify for your version]

**Q: "What if I lock myself out?"**  
A: You have admin access. We can reset if needed.

#### Critical Review Point (After lunch)
- Review their permission matrix
- Validate security logic
- Have them explain their choices

#### End of Day Review (30 min)
Check:
- [ ] Roles are well-defined with clear purposes
- [ ] Permissions follow least privilege principle
- [ ] All scenarios have been tested
- [ ] Documentation explains reasoning

**Feedback Focus:** Security mindset and thorough testing.

---

### Day 4: API Integration - REST API & Views

**⏰ Time Commitment:** 6-7 hours  
**Difficulty:** Medium  
**Key Concepts:** REST APIs, Authentication, Views, Dashboards

#### Morning Kickoff (15 min)
- Demo an API call live
- Explain REST principles
- Show authentication in action
- Discuss API best practices

#### What to Watch For
- Authentication troubles (expired tokens)
- Not understanding HTTP status codes
- Skipping error cases
- Views that aren't actually useful

#### Common Questions
**Q: "My API call returns 401. What's wrong?"**  
A: Check: 1) Token is valid, 2) Authorization header format, 3) User has permissions

**Q: "What's the difference between GET and POST?"**  
A: GET retrieves data, POST creates new data

**Q: "How do I include related data in API response?"**  
A: Use fields parameter: `?fields=*,author.*`

#### Mid-Day API Check (15 min)
- Review their API collection
- Test a few requests with them
- Validate authentication setup

#### End of Day Review (30 min)
Check:
- [ ] Complete API collection works
- [ ] Views are useful and well-designed
- [ ] Dashboard provides real value
- [ ] API documentation is clear

**Feedback Focus:** Practical usability of views and API integration.

---

### Day 5: Capstone Project - Task Management System

**⏰ Time Commitment:** 8 hours  
**Difficulty:** Hard (but should feel achievable)  
**Key Concepts:** Everything from Week 1

#### Morning Kickoff (20 min)
- Explain the project requirements
- Emphasize this is their chance to shine
- Remind them to manage time wisely
- Set expectation for presentation

#### Your Role Today
- **Be available but hands-off**
- Let them struggle a bit (builds confidence)
- Only intervene if stuck >20 minutes
- Encourage creativity and ownership

#### Check-in Schedule
- **10:30 AM:** Review data model design (15 min)
- **12:30 PM:** Quick progress check (10 min)
- **3:00 PM:** Review before final push (15 min)
- **5:00 PM:** Presentation prep review (20 min)

#### What to Watch For
- Poor time management
- Perfectionism (done is better than perfect)
- Not testing as they go
- Incomplete documentation

#### Common Questions
**Q: "I'm running out of time. What should I skip?"**  
A: Skip bonus features, focus on core requirements

**Q: "My [feature] isn't working. Help?"**  
A: What have you tried? What error do you see? (Guide, don't solve)

**Q: "Is this good enough?"**  
A: Does it meet requirements? Can you demo it? If yes, it's good!

#### Presentation Time (5:30 PM - as needed)

**As Evaluator:**
- [ ] Listen actively and take notes
- [ ] Ask probing questions about decisions
- [ ] Encourage them to show their work
- [ ] Provide positive reinforcement
- [ ] Note areas for improvement

**Evaluation Criteria:**
1. Completeness (30%) - All core features done?
2. Quality (25%) - Well-designed and functional?
3. Understanding (25%) - Can they explain their work?
4. Presentation (20%) - Clear communication?

#### End of Day Wrap-up (30 min)
- Congratulate them on completion!
- Provide comprehensive feedback
- Discuss next steps / future learning
- Get their feedback on the program
- Complete PROGRESS.md together

---

## 💡 General Mentoring Tips

### Do's ✅

1. **Be Encouraging**
   - Celebrate small wins
   - Acknowledge effort
   - Frame mistakes as learning opportunities

2. **Ask Questions**
   - "What do you think will happen?"
   - "Why did you choose this approach?"
   - "How would you test this?"

3. **Guide Discovery**
   - Point to documentation
   - Suggest where to look
   - Help them find answers

4. **Be Patient**
   - Everyone learns at different speeds
   - Some will need more support
   - Confusion is part of learning

5. **Share Experience**
   - Tell relevant stories
   - Share common pitfalls
   - Explain real-world applications

### Don'ts ❌

1. **Don't Give Direct Answers**
   - Avoid: "Here's the solution..."
   - Instead: "Have you tried...?" or "What if you...?"

2. **Don't Compare Interns**
   - Each person's journey is unique
   - Some have different backgrounds

3. **Don't Overwhelm**
   - Stick to the curriculum
   - Save advanced topics for later

4. **Don't Skip Fundamentals**
   - Even if they seem obvious
   - Build strong foundation

5. **Don't Neglect Soft Skills**
   - Documentation matters
   - Communication is crucial
   - Presentation skills are important

---

## 🚨 Troubleshooting Common Issues

### Technical Issues

**Issue: Can't log into Ucode**
- Reset password
- Check account is active
- Verify URL is correct

**Issue: Relationship not working**
- Check field types match
- Verify foreign key is set
- Ensure data exists in both tables

**Issue: API returns 403 Forbidden**
- Check user role permissions
- Verify token hasn't expired
- Check API endpoint exists

**Issue: View not showing data**
- Check filters aren't too restrictive
- Verify permissions for tables
- Refresh the page

### Learning Issues

**Issue: Intern is falling behind**
- Identify bottleneck (technical vs conceptual)
- Provide targeted support
- Adjust expectations if needed
- Consider pairing with stronger intern

**Issue: Intern is racing ahead**
- Give bonus challenges
- Have them help others (teaching reinforces learning)
- Ask them to document advanced features

**Issue: Intern is frustrated**
- Take a break
- Review what they've accomplished
- Break problem into smaller pieces
- Pair program with them

---

## 📊 Evaluation & Feedback

### Daily Feedback Template

```markdown
**Intern Name:** _____________
**Day:** _____
**Date:** _____________

### Deliverables Submitted
- [ ] All required files
- [ ] Screenshots
- [ ] Documentation
- [ ] Clean work (no test data)

### Understanding (1-5)
- Concepts: ___/5
- Technical execution: ___/5
- Problem-solving: ___/5

### Strengths Demonstrated
1. 
2. 
3. 

### Areas for Improvement
1. 
2. 
3. 

### Tomorrow's Focus
1. 
2. 

### Additional Notes


**Mentor Signature:** _____________
```

### Final Evaluation Template

```markdown
**Intern Name:** _____________
**Program Completion Date:** _____________

### Overall Performance

**Technical Skills** (1-10): ___
- Database design
- CRUD operations
- API integration
- Security configuration

**Soft Skills** (1-10): ___
- Documentation
- Problem-solving
- Communication
- Time management

**Capstone Project** (1-10): ___
- Completeness
- Quality
- Presentation

### Readiness Assessment

Can this intern:
- [ ] Build a basic Ucode application independently?
- [ ] Design appropriate data models?
- [ ] Configure security properly?
- [ ] Integrate with Ucode API?
- [ ] Troubleshoot common issues?

### Recommendation

- [ ] Ready for independent work
- [ ] Ready with occasional support
- [ ] Needs more training on: __________

### Additional Comments



**Mentor Signature:** _____________
**Date:** _____________
```

---

## 📚 Additional Resources

### For You (Mentor)
- [Ucode Documentation](https://ucode.gitbook.io/ucode-docs)
- [Teaching Best Practices]
- [Mentoring Guide]

### For Interns
- Ucode Docs (provided in exercises)
- Your experience and guidance!

---

## 🔄 Program Improvement

After each cohort:

1. **Gather Feedback**
   - What worked well?
   - What was confusing?
   - What should be added/removed?

2. **Update Materials**
   - Fix errors or unclear instructions
   - Add FAQs
   - Update screenshots if UI changed

3. **Share Learnings**
   - Document common issues
   - Create additional resources
   - Improve mentor guide

---

## ✅ Final Checklist

Before starting program:
- [ ] Ucode instance set up and tested
- [ ] Intern accounts created
- [ ] Sample database prepared
- [ ] All exercises reviewed
- [ ] Schedule planned
- [ ] Evaluation templates ready

During program:
- [ ] Daily check-ins completed
- [ ] Questions answered promptly
- [ ] Feedback provided daily
- [ ] Progress documented

After program:
- [ ] Final evaluation completed
- [ ] Feedback collected from interns
- [ ] Materials updated
- [ ] Next steps discussed with interns

---

## Questions?

If you have questions about the program or need support:
- Review the exercise README files
- Check the Ucode documentation
- Consult with other mentors
- Contact program coordinator

---

**Good luck! You're helping shape the next generation of Ucode developers!** 🚀

