# Project Context Template

Copy this template for new automation projects. Fill in each section during the discovery phase.

---

## Project Overview

| Field | Value |
|-------|-------|
| Project Name | |
| Version | 0.1 |
| Status | Discovery / Design / Build / Testing / Complete |
| Platform | n8n / Zapier / Make / Custom |
| Created | YYYY-MM-DD |
| Last Updated | YYYY-MM-DD |

## Problem Statement

_One paragraph describing the core problem this automation solves._

## Stakeholders

### 1. [Role Name]

**Problem Statement**: 

**User Story**: As a [role], I want [goal] so that [benefit].

**Acceptance Criteria**:
- [ ] Criteria 1
- [ ] Criteria 2
- [ ] Criteria 3

### 2. [Role Name]

**Problem Statement**: 

**User Story**: As a [role], I want [goal] so that [benefit].

**Acceptance Criteria**:
- [ ] Criteria 1
- [ ] Criteria 2

### 3. [Role Name] (if applicable)

**User Story**: As a [role], I want [goal] so that [benefit].

**Acceptance Criteria**:
- [ ] Criteria 1

## Terminology

Define key terms to avoid confusion:

| Term | Definition |
|------|------------|
| Customer | |
| Owner | |
| [Term] | |

## Decision Points

Decisions that must be made before building:

### Decision 1: [Topic]

**Options**:
- Option A: [description]
- Option B: [description]

**Selected**: 
**Rationale**: 

### Decision 2: [Topic]

**Options**:
- Option A: [description]
- Option B: [description]

**Selected**: 
**Rationale**: 

## Technical Architecture

### Trigger Mechanism

| Option | Pros | Cons |
|--------|------|------|
| Schedule | | |
| Webhook | | |
| Event-driven | | |

**Selected**: 
**Rationale**: 

### Data Storage

| Option | Pros | Cons |
|--------|------|------|
| Google Sheets | | |
| Airtable | | |
| Database | | |

**Selected**: 
**Rationale**: 

### Communication Channels

| Channel | Provider | Status |
|---------|----------|--------|
| Email | | Required / Optional / Future |
| SMS | | Required / Optional / Future |
| Voice | | Required / Optional / Future |

### Architecture Diagram

```
[Trigger]
    ↓
[Process 1] → [Process 2]
    ↓
[Output]
```

## Constraints

### Technical
- Platform: 
- Required integrations:
- Authentication method:

### Business
- Budget:
- Timeline:
- Compliance requirements:

### Scope Exclusions
_What is explicitly NOT included in this project:_
- 
- 

## Data Model

### Configuration Data

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| | | | |

### Primary Data Entity

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| | | | |

## Workflow Logic

### Main Workflow

```
[Trigger]
    ↓
[Step 1]
    ↓
[Decision Point]
    ├── [Branch A]
    └── [Branch B]
    ↓
[Final Step]
```

### Secondary Workflows (if applicable)

```
[Trigger]
    ↓
[Logic]
```

## Message Templates

### Template 1

**Channel**: 
**Trigger**: 
**Subject/Preview**: 
**Content**: 

### Template 2

**Channel**: 
**Trigger**: 
**Content**: 

## Timing / Scheduling

| Event | Demo Mode | Production Mode |
|-------|-----------|-----------------|
| | | |

## Error Handling

| Error Type | Detection | Response |
|------------|-----------|----------|
| | | |

## Testing Requirements

### Functional Tests
- [ ] Test 1:
- [ ] Test 2:
- [ ] Test 3:

### Edge Cases
- [ ] Empty data
- [ ] Invalid input
- [ ] Duplicate records

## Success Metrics

How will we know the project is successful?

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| | | |

## Known Issues / Lessons Learned

_Add as project progresses:_

### Issue 1: [Title]
**Cause**: 
**Solution**: 

## Future Enhancements

_Ideas for future versions:_
- [ ] 
- [ ] 

## References

- Documentation:
- Related projects:
- External resources:

---

## Template Usage Notes

1. **Discovery Phase**: Fill in Overview, Problem Statement, Stakeholders, Terminology
2. **Design Phase**: Complete Decision Points, Architecture, Data Model, Workflow Logic
3. **Build Phase**: Update as implementation reveals new details
4. **Testing Phase**: Add Known Issues, complete Testing Requirements
5. **Completion**: Update Status, add Lessons Learned, identify Future Enhancements

Keep this document updated throughout the project. It serves as the single source of truth for project scope and decisions.
