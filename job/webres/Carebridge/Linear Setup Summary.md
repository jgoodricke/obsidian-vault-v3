This setup is designed for a **small development team (1 developer + product owner)** working primarily on **bug fixes and incremental improvements**. The goal is to keep the workflow simple while still allowing **prioritisation, backlog management, and delivery forecasting for clients**.

The system uses:

- **Kanban-style workflow** for issue status
    
- **Linear Cycles** for short-term planning and forecasting
    
- **Simple effort estimates** for capacity planning
    

---

# 1. Workflow (Kanban)

Use a simple issue lifecycle with the following states:

```
Backlog
Ready
In Progress
Review
Done
```

Meaning of each stage:

**Backlog**  
Prioritised issues that are not yet scheduled.

**Ready**  
Issues prepared for development in the current or upcoming cycle.

**In Progress**  
Currently being worked on.

**Review**  
Work completed and awaiting verification, testing, or deployment.

**Done**  
Completed and released work.

This workflow keeps the process simple while still showing progress clearly.

---

# 2. Cycles (Planning Horizon)

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
```

This provides approximately **six weeks of forecast visibility**, which helps when customers ask when specific items may be completed.

Issues are assigned to cycles once they move into **Ready**.

---

# 3. Issue Estimates

Enable **issue estimates** and use a simple size scale:

| Estimate | Typical effort | Story Points |
| -------- | -------------- | ------------ |
| XS       | < 1 hour       | 1            |
| S        | ~½ day         | 2            |
| M        | ~1 day         | 3            |
| L        | 2–3 days       | 5            |
| XL       | 4–5 days       | 8            |

Rules:

- Anything larger than **XL should be split**
    
- Estimates are used for **forecasting and planning**, not exact commitments
    

---

# 4. Prioritisation

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

# 5. Labels

Keep labels minimal.

Recommended labels:

**Type**

```
bug
enhancement
investigation
technical-debt
```

Optional **area labels** if the backlog grows:

```
reporting
authentication
patient-records
admin
```

Labels help organise and filter issues.

---

# 6. GitHub Integration

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

# 7. Backlog Management

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

# 8. Forecasting Delivery

Cycles and estimates together allow rough forecasting.

Example cycle plan:

|Cycle|Work|
|---|---|
|Cycle 1|urgent fixes and high-priority issues|
|Cycle 2|next most important backlog items|
|Cycle 3|lower priority improvements|

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