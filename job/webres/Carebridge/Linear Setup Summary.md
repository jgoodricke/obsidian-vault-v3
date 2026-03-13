This setup is designed for a **small development team (1 developer + product owner)** working primarily on **bug fixes and incremental improvements**. The goal is to keep the workflow simple while still allowing **prioritisation, backlog management, and delivery forecasting for clients**.

The system uses:

- **Kanban-style workflow** for issue status
    
- **Linear Cycles** for short-term planning and forecasting
    
- **Simple effort estimates** for capacity planning
    

---

# Workflow (Kanban)

Use a simple issue lifecycle with the following states:

```
Inbox
Backlog
Ready
In Progress
Review
Done
```

Meaning of each stage:

**Inbox**  
New issuese that are yet to be estimated.

**Backlog**  
Prioritised issues that are not yet scheduled.

**Ready**  
The issue is understood, estimated, and testable. It is ready for development in the current or upcoming cycle.

**In Progress**  
Currently being worked on.

**Review**  
Work completed and awaiting verification, testing, or deployment.

**Done**  
Completed and released work.

This workflow keeps the process simple while still showing progress clearly.

---

# Cycles (Planning Horizon)

Enable **Cycles** in Linear.

Recommended configuration:

```
Cycle length: 2 weeks
```

Maintain roughly **three cycles planned ahead**.

Example:

```
Cycle 1 – current work (weeks 1–2)
Cycle 2 – next planned work (weeks 3–4)
Cycle 3 – planned work after that (weeks 5–6)
Cycle 4 – planned work after that (weeks 7–8)
```

This provides approximately **8 weeks of forecast visibility**, which helps when customers ask when specific items may be completed.

Issues are assigned to cycles once they move into **Ready**.

---

# Issue Estimates

Enable **issue estimates** and use a simple size scale:

| Estimate | Typical effort |  Cycle Value |
| -------- | -------------- | ------------ |
| XS       | < 1 hour       | 1            |
| S        | ~½ day         | 2            |
| M        | ~1 day         | 3            |
| L        | 2–3 days       | 5            |
| XL       | 4–5 days       | 8            |

Rules:
- Anything larger than **XL should be split**
- Anything smaller than XS should e combined into one ticket.
- Estimates are used for **forecasting and planning**, not exact commitments

---

# Prioritisation

Use Linear’s built-in priority levels:

```
Urgent
High
Medium
Low
```

Guideline:

**Urgent** – production issues affecting customers  
**High** – important bug fixes or client requests  
**Medium** – normal backlog work
**Low** – improvements or technical debt

Backlog should always be sorted by **priority and importance**.

---

# Labels

Keep labels minimal.

Recommended labels:

**Type**

```
bug
feature
improvement
investigate
Maintanance
```

Also add a label for blocked tasks, indicating that it cannot be worked on until clarification is given from someone else. When an issue is blocked, a comment should be added explaining what the blocker is.

Optional **area labels** if the backlog grows:

```
reporting
authentication
patient-records
admin
```

Labels help organise and filter issues.

---
# Projects
**Projects** are used to group related issues that contribute to a larger initiative. A project represents a broader goal made up of multiple tickets.

Example:

```
Project: Hospital Reporting Improvements
  - Fix export bug
  - Add CSV fields
  - Improve reporting filters
```

Linear tracks project progress automatically based on completed issues, making it easier to see how far an initiative has progressed.

Note that not all tasks need to be part of a project. Projects should be used for **larger pieces of work spanning multiple tickets and cycles**. 

---

# GitHub Integration

Connect Linear with GitHub so development activity links directly to issues.

Typical workflow:

```
Create Linear issue
Create branch referencing issue
Open pull request
Merge PR → issue closes automatically
```

Example branch naming:

```
feature/QI-142-export-fix
bug/QI-151-login-error
```

This keeps commits and development work tied to the corresponding ticket.

---

# Backlog Management

Perform **weekly backlog grooming**.

Steps:

1. Review new issues in the backlog
2. Clarify descriptions
3. Add estimates
4. Assign priority
5. Decide whether the item should remain in the backlog

Keep backlog size manageable:

```
Target backlog size: 30–50 tickets
```

Archive outdated or irrelevant items.


---

# 11. Weekly Backlog Refinement

Once per week:

Review new issues  
Clarify tickets  
Split large work  
Reorder backlog

This should take **15–30 minutes** for a small team.

---

# 12. End-of-Cycle Review

At the end of each cycle:

Demonstrate completed work  
Discuss feedback  
Adjust priorities

This keeps stakeholders aligned and avoids surprises.


---

# 8. Forecasting Delivery

Cycles and estimates together allow rough forecasting.

Example cycle plan:

| Cycle   | Work                                            |
| ------- | ----------------------------------------------- |
| Cycle 1 | Work that will be completed in the next 2 weeks |
| Cycle 2 | Work that will be completed next                |
| Cycle 3 | Forecast                                        |
| Cycle 4 |                                                 |

When customers ask about delivery timelines, responses can be based on cycle assignment:

Example:

> This issue is currently scheduled for Cycle 2, which is planned for early April.

This gives **predictable delivery windows without overcommitting to specific dates**.

---

# Summary

The Linear setup should include:

**Workflow**

```
Backlog → Ready → In Progress → Review → Done
```

**Planning**

```
2-week cycles
3 cycles planned ahead (~6 weeks)
```

**Estimation**

```
XS / S / M / L / XL
```

**Prioritisation**

```
Urgent / High / Medium / Low
```

This structure keeps the system **simple, maintainable, and predictable**, while still allowing the team to manage a growing backlog and provide customers with **reasonable delivery forecasts**.