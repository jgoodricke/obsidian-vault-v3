# Release Checklist
- [x] PRs merged and approved
- [x] Latest main pulled locally
- [x] Exact deploy commit confirmed
- [x] Annotated tag created
- [x] Tag pushed to origin
- [x] PR Created with release notes
- [x] PR reviewed
- [x] Deployment run
- [x] Migrations checked
- [x] Smoke tests completed
# Key Commands
```bash
# Before Deployment
git checkout main && git pull
git tag -a prod-2026-03-23 -m "Production release 2026-03-23"
git push origin prod-2026-03-23

# After Deployment
git tag --sort=-creatordate
git log --oneline <previous-tag>..prod-2026-03-23
```

# Example Release Notes

**Release:** prod-2026-03-23
**Environment:** Production
**Cycle:** Cycle 1

**Cards Included:**
- CBR-118 Re-presentation updates

**Other changes :** 
- IDOR fixes (partial)
- Minor UI fixes

**Database changes:**
- None

**Notes:**
- Some IDOR tasks moved to next cycle
