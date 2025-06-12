# Iterative Development Example

This example demonstrates how to use the iteration features to continuously improve your project through multiple development cycles.

## Scenario

You've completed the initial version of a REST API and want to improve it based on user feedback and identified gaps.

## Initial Setup

Assume you have a completed project with:
- Basic REST API implementation
- Initial test coverage
- Simple documentation

```
api-project/
├── src/
│   ├── app.js
│   ├── routes/
│   └── models/
├── tests/
├── tasks/
│   ├── project-prd.md
│   ├── tasks-list.md (all marked [x])
│   └── agent-*-tasks.md
└── README.md
```

## Phase 1: Analysis and Feedback Collection

### Step 1: Collect User Feedback

Create feedback files for issues and requests:

```bash
mkdir -p feedback

# Bug report
cat > feedback/2024-01-15-bug-timeout.md << 'EOF'
# User Feedback

## Date: 2024-01-15

## Feedback Type: bug

## Priority: high

## Summary
API endpoints timeout under moderate load

## Details
When more than 50 concurrent requests hit the /api/products endpoint, 
the server starts timing out and returns 504 errors.

## Expected Behavior
API should handle at least 200 concurrent requests

## Actual Behavior
Timeouts occur at 50+ concurrent requests

## Suggested Solution
Implement caching and optimize database queries

## Impact
Cannot use in production due to performance issues
EOF

# Feature request
cat > feedback/2024-01-16-feature-auth.md << 'EOF'
# User Feedback

## Date: 2024-01-16

## Feedback Type: feature

## Priority: medium

## Summary
Add OAuth2 authentication support

## Details
Current API only supports basic API key authentication. 
Need OAuth2 support for third-party integrations.

## Impact
Cannot integrate with enterprise clients
EOF
```

### Step 2: Run Initial Analysis

```bash
# Analyze current state against PRD
/user:iterate-project
```

This generates:
- `tasks/iteration-analysis-[timestamp].md`
- Shows completion status
- Identifies gaps

## Phase 2: First Iteration - Bug Fixes

### Step 1: Create Improvement PRD

```bash
# Generate improvement-focused PRD
/user:iterate-project mode=improve feedback-file=feedback/2024-01-15-bug-timeout.md
```

Output: `tasks/project-prd-v2.md`

### Step 2: Generate New Tasks

```bash
# Generate tasks from updated PRD
/user:generate-tasks prd=tasks/project-prd-v2.md output=tasks/tasks-list-v2.md
```

### Step 3: Execute Improvements

```bash
# Execute the improvement tasks
/user:execute-tasks tasks/tasks-list-v2.md
```

Tasks might include:
- [ ] Implement Redis caching for product endpoints
- [ ] Optimize database queries with indexes
- [ ] Add connection pooling
- [ ] Implement request rate limiting
- [ ] Add load testing suite

### Step 4: Verify Improvements

```bash
# Run performance tests
npm run test:performance

# Check the results
cat test-results/performance-report.md
```

## Phase 3: Second Iteration - New Features

### Step 1: Analyze All Feedback

```bash
# Create feedback summary
source utils/feedback-handler.md
create_feedback_summary feedback feedback/summary.md

# Review summary
cat feedback/summary.md
```

### Step 2: Plan Next Phase

```bash
# Generate next phase PRD incorporating all feedback
/user:iterate-project mode=next-phase feedback-file=feedback/summary.md
```

Output: `tasks/project-prd-v3.md`

### Step 3: Multi-Agent Implementation

For larger feature additions, use multi-agent approach:

```bash
# Generate tasks
/user:generate-tasks prd=tasks/project-prd-v3.md

# Distribute to agents
/user:generate-multi-agent-tasks

# Execute with multiple agents
/user:execute-multi-agent-tasks max-agents=3
```

## Phase 4: Continuous Improvement Loop

### Step 1: Establish Metrics

Create a metrics tracking file:

```bash
cat > metrics/baseline.md << 'EOF'
# Performance Metrics Baseline

## Response Times
- GET /api/products: 45ms average
- POST /api/orders: 120ms average
- Peak concurrent users: 50

## Code Quality
- Test coverage: 65%
- Cyclomatic complexity: 12 average
- Technical debt: 3.2 days

## User Satisfaction
- Bug reports/week: 5
- Feature requests/week: 8
- User rating: 3.5/5
EOF
```

### Step 2: Regular Iteration Cycles

Set up a regular iteration schedule:

```bash
# Weekly iteration script
cat > scripts/weekly-iteration.sh << 'EOF'
#!/bin/bash

echo "=== Weekly Iteration - $(date) ==="

# 1. Collect new feedback
echo "New feedback files:"
find feedback -name "*.md" -mtime -7 -type f

# 2. Run analysis
/user:iterate-project mode=improve

# 3. Update metrics
./scripts/update-metrics.sh

# 4. Generate report
echo "## Iteration Report" > reports/week-$(date +%U).md
echo "### Improvements Made" >> reports/week-$(date +%U).md
git log --oneline --since="1 week ago" >> reports/week-$(date +%U).md

echo "Iteration complete. Review: reports/week-$(date +%U).md"
EOF

chmod +x scripts/weekly-iteration.sh
```

### Step 3: Track Progress Over Time

```bash
# After each iteration, update documentation
source utils/doc-updater.md
update_all_docs

# Generate changelog
update_changelog CHANGELOG.md

# Create visual progress
cat > metrics/progress.md << 'EOF'
# Progress Tracking

## Version 1.0 (Baseline)
- Response time: 45ms
- Concurrent users: 50
- Test coverage: 65%

## Version 2.0 (After Iteration 1)
- Response time: 22ms ⬇️ 51%
- Concurrent users: 200 ⬆️ 300%
- Test coverage: 78% ⬆️ 20%

## Version 3.0 (After Iteration 2)
- Response time: 18ms ⬇️ 18%
- Concurrent users: 500 ⬆️ 150%
- Test coverage: 85% ⬆️ 9%
- Added: OAuth2 support ✅
- Added: GraphQL endpoint ✅
EOF
```

## Complete Iteration Example

```bash
# Starting point: completed v1.0
cd api-project

# 1. Collect feedback over time
mkdir -p feedback
# ... users report issues and requests ...

# 2. First iteration - performance fixes
/user:iterate-project mode=improve
/user:generate-tasks prd=tasks/project-prd-v2.md
/user:execute-tasks

# 3. Test improvements
npm run test:performance
npm run test:load

# 4. Second iteration - new features
/user:iterate-project mode=next-phase
/user:generate-tasks prd=tasks/project-prd-v3.md
/user:generate-multi-agent-tasks
/user:execute-multi-agent-tasks

# 5. Update documentation
source utils/doc-updater.md
update_all_docs

# 6. Tag release
git tag -a v2.0 -m "Performance improvements and OAuth2 support"

# 7. Plan next iteration
./scripts/weekly-iteration.sh
```

## Iteration Strategies

### 1. Feedback-Driven Iteration

```bash
# Prioritize by user impact
for feedback in feedback/*.md; do
    priority=$(grep "Priority:" "$feedback" | cut -d' ' -f3)
    if [[ "$priority" == "high" ]]; then
        /user:iterate-project mode=improve feedback-file="$feedback"
        break
    fi
done
```

### 2. Metric-Driven Iteration

```bash
# Focus on specific metrics
if [[ $(get_test_coverage) -lt 80 ]]; then
    echo "Focus: Improve test coverage" > feedback/metric-coverage.md
    /user:iterate-project mode=improve feedback-file=feedback/metric-coverage.md
fi
```

### 3. Feature-Complete Iteration

```bash
# After all planned features are done
if grep -c "\[ \]" tasks/project-prd.md | grep -q "0"; then
    /user:iterate-project mode=next-phase
fi
```

## Best Practices for Iteration

1. **Regular Cadence**: Iterate on a schedule (weekly/bi-weekly)
2. **Measure Everything**: Track metrics to show improvement
3. **Prioritize Feedback**: Address high-impact issues first
4. **Document Changes**: Maintain clear changelog
5. **Test Thoroughly**: Ensure improvements don't break existing features
6. **Communicate Progress**: Share iteration reports with stakeholders

## Common Iteration Patterns

| Pattern | When to Use | Command |
|---------|-------------|---------|
| Bug Fix Sprint | High-priority bugs | `mode=improve` |
| Feature Release | New capabilities needed | `mode=next-phase` |
| Tech Debt | Code quality issues | `mode=improve` |
| Performance | Speed/scale issues | `mode=improve` |
| User Experience | Usability feedback | `mode=next-phase` |

## Next Steps

- [Automated CI/CD Integration](ci-integration.md)
- [Advanced Testing Strategies](testing-strategies.md)
- [Release Management](release-management.md)