# Contributing to WeatherOps Frontend

Welcome! We're excited to have you contribute to WeatherOps. This document provides guidelines for contributing to the project.

## Table of Contents

1. [Code of Conduct](#code-of-conduct)
2. [Getting Started](#getting-started)
3. [Development Workflow](#development-workflow)
4. [Git Conventions](#git-conventions)
5. [Pull Request Process](#pull-request-process)
6. [Coding Standards](#coding-standards)
7. [Testing](#testing)
8. [Documentation](#documentation)
9. [Reporting Issues](#reporting-issues)
10. [Getting Help](#getting-help)

---

## Code of Conduct

### Our Pledge

We are committed to providing a welcoming and inspiring community for all. Please read and respect our Code of Conduct:

- **Be Respectful**: Treat everyone with respect and courtesy
- **Be Inclusive**: Welcome contributors from all backgrounds
- **Be Collaborative**: Work together to solve problems
- **Be Professional**: Keep discussions focused and constructive
- **Zero Tolerance**: Harassment or discrimination is never acceptable

### Enforcement

Violations of the Code of Conduct may result in:
- Private warning
- Public warning
- Temporary or permanent ban from contributing

Report violations to: conduct@weatherops.com

---

## Getting Started

### 1. Fork the Repository

```bash
# Navigate to repository
# Click "Fork" button on GitHub

# Clone your fork
git clone https://github.com/YOUR_USERNAME/weatherops-frontend.git
cd weatherops-frontend

# Add upstream remote
git remote add upstream https://github.com/Kizito24/weatherops-frontend.git
git remote -v
```

### 2. Set Up Development Environment

```bash
# Install dependencies
npm install

# Create .env file
cp .env.example .env

# Start development server
npm run dev

# In another terminal, run tests
npm run test:e2e
```

### 3. Create a Branch

```bash
# Pull latest from upstream
git fetch upstream
git rebase upstream/main

# Create feature branch
git checkout -b feature/short-description

# Example: feature/add-location-export
```

---

## Development Workflow

### 1. Write Code

```bash
# Start development server
npm run dev

# Open http://localhost:3000 in browser
# Editor should auto-reload on file changes
```

### 2. Type Checking

```bash
# Check for TypeScript errors
npm run lint

# Fix auto-fixable issues
npm run lint -- --fix
```

### 3. Manual Testing

```bash
# Test in browser
# - Use all browsers (Chrome, Firefox, Safari)
# - Test both dark and light modes
# - Test on mobile (use DevTools)
# - Test form validation
# - Test error states
```

### 4. Automated Testing

```bash
# Run E2E tests
npm run test:e2e

# Run specific test file
npm run test:e2e -- locations.spec.ts

# Run in headed mode to see browser
npm run test:e2e:headed

# Debug failing test
npm run test:e2e:debug -- locations.spec.ts
```

### 5. Code Review (Self-Review)

```bash
# Review your own changes
git diff main..HEAD

# Check for:
# ✅ Clear variable/function names
# ✅ No console.log statements
# ✅ Proper error handling
# ✅ No hardcoded values
# ✅ Comments explaining "why" not "what"
# ✅ No large commits (split into logical chunks)
```

---

## Git Conventions

### Branch Naming

```
feature/description     - New feature
fix/description         - Bug fix
refactor/description    - Code refactoring
docs/description        - Documentation
test/description        - Test addition
perf/description        - Performance improvement
style/description       - Styling changes
chore/description       - Maintenance tasks

Examples:
- feature/add-csv-export
- fix/login-validation-bug
- refactor/simplify-api-client
- docs/update-setup-guide
```

### Commit Messages

Follow the conventional commits format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types**:
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation
- `style` - Formatting (no code change)
- `refactor` - Code reorganization
- `perf` - Performance improvement
- `test` - Test changes
- `chore` - Maintenance

**Scope**: Component or module affected

**Subject**:
- Imperative mood: "add" not "added"
- Don't capitalize first letter
- No period at end
- Max 50 characters

**Body**:
- Explain what and why, not how
- Wrap at 72 characters
- Separate from subject with blank line

**Footer**:
- Reference issues: `Closes #123`
- Breaking changes: `BREAKING CHANGE: description`

### Commit Examples

```
✅ Good commit messages:
feat(locations): add batch location import

Allow users to import multiple locations via CSV file.
Includes validation and error reporting.

Closes #456

fix(auth): correct token expiration check

The token expiration was being calculated incorrectly,
causing premature logout. Fixed calculation logic.

docs(setup): update environment variables guide

refactor(api): extract error handling to separate module

Makes error handling more testable and reusable.

❌ Bad commit messages:
"fix stuff"
"Updated code"
"Random changes"
```

### Squash Related Commits

```bash
# Before submitting PR, squash related commits
git rebase -i upstream/main

# In interactive rebase, mark commits to squash:
# pick abc1234 Initial implementation
# squash def5678 Fix typo
# squash ghi9012 Add tests

# Result: One clean commit
```

---

## Pull Request Process

### 1. Push Your Branch

```bash
# Push to your fork
git push origin feature/your-feature

# Or track the branch
git push -u origin feature/your-feature
```

### 2. Create Pull Request

```bash
# Via CLI
gh pr create \
  --title "Add CSV export for alerts" \
  --body "$(cat <<EOF
## Description
Users can now export alerts to CSV format for analysis.

## Changes
- Added ExportModal component
- Implemented CSV formatting
- Added export button to alerts page

## Testing
- [x] Manual testing on Chrome
- [x] Manual testing on Firefox
- [x] Mobile responsive tested
- [x] E2E tests added

## Screenshots
![export modal](url)

## Related Issues
Closes #456
EOF
)"

# Or via GitHub web interface
# 1. Push branch
# 2. Go to repository
# 3. Click "Compare & pull request"
# 4. Fill in description
# 5. Submit
```

### 3. PR Description Template

```markdown
## Description
Brief description of what this PR does

## Motivation & Context
Why are these changes needed? What problem do they solve?

## Types of Changes
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Added unit tests
- [ ] Added E2E tests
- [ ] Manual testing completed
- [ ] No new warnings

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] No breaking changes
- [ ] Tests pass locally

## Screenshots (if applicable)
[Add screenshots or GIFs]

## Related Issues
Closes #123
```

### 4. Address Review Comments

```bash
# Make requested changes
# Don't force push unless asked

# Commit changes
git add .
git commit -m "Address review comments: improve error handling"

# Push to same branch
git push origin feature/your-feature

# GitHub automatically updates PR
```

### 5. Wait for Approval

- Maintainers will review
- At least one approval required
- All checks must pass
- CI/CD tests must pass

### 6. Squash and Merge (if requested)

```bash
# After approval, squash commits
git rebase -i upstream/main

# Reorder and squash commits as needed
# mark final commit as "pick"
# mark others as "squash"

# Force push (only after rebasing)
git push -f origin feature/your-feature

# Merge via GitHub interface or CLI
gh pr merge <number> --squash
```

---

## Coding Standards

### TypeScript

```typescript
// ✅ Always define types
interface LocationFormData {
  name: string;
  latitude: number;
  longitude: number;
}

// ✅ Use explicit return types
function calculateDistance(
  lat1: number,
  lng1: number,
  lat2: number,
  lng2: number
): number {
  // implementation
}

// ❌ Avoid any
const data: any = response.data;  // Bad

// ✅ Use union types
type AlertStatus = 'active' | 'resolved';

// ✅ Use strict null checking
const user: UserProfile | null = getUser();
if (user) {
  console.log(user.email);
}
```

### React Components

```typescript
// ✅ Functional components with hooks
interface LocationsPageProps {
  locations: Location[];
  onAddLocation: (name: string, lat: number, lng: number) => Promise<void>;
}

export default function LocationsPage({
  locations,
  onAddLocation,
}: LocationsPageProps) {
  const [isLoading, setIsLoading] = useState(false);

  useEffect(() => {
    // Load data
  }, []);

  return (
    <div>
      {/* JSX */}
    </div>
  );
}

// ❌ Avoid class components (unless ErrorBoundary)
class MyComponent extends React.Component {
  // Not recommended for new code
}
```

### Styling

```typescript
// ✅ Use Tailwind CSS
<div className="p-4 rounded-lg bg-white dark:bg-slate-900">
  Content
</div>

// ✅ Extract repeated styles to components
function Card({ children }: { children: React.ReactNode }) {
  return (
    <div className="p-4 rounded-lg bg-white dark:bg-slate-900">
      {children}
    </div>
  );
}

// ❌ Avoid inline styles
<div style={{ padding: '16px', backgroundColor: 'white' }}>  // Bad
```

### Error Handling

```typescript
// ✅ Specific error handling
try {
  await createLocation(name, lat, lng);
  showToast('Location created', 'LOW');
} catch (err) {
  if (err instanceof ApiError) {
    if (err.statusCode === 400) {
      showToast('Invalid coordinates', 'MEDIUM');
    } else {
      showToast(err.message, 'MEDIUM');
    }
  }
}

// ❌ Generic error handling
try {
  // something
} catch {
  console.log('Error');  // Vague and unhelpful
}
```

### Comments

```typescript
// ❌ Avoid obvious comments
const name = user.name; // Get user name

// ✅ Explain "why" not "what"
// User authentication token expires after 1 hour,
// so we check and refresh if needed
const shouldRefreshToken = Date.now() > tokenExpiresAt - 300000;

// ✅ Explain complex logic
// Calculate bounding box to minimize API calls
// by grouping nearby locations
const bbox = calculateBoundingBox(locations);

// ❌ Don't leave commented code
// const oldImplementation = () => { ... };  // Remove this
```

---

## Testing

### Write Tests for Your Changes

```typescript
// Example: Location CRUD test
test('should create new location', async ({ page }) => {
  // Arrange
  await loginAs(page, 'test@example.com', 'password');
  await page.goto('/#locations');

  // Act
  await page.locator('input[name="name"]').fill('New City');
  await page.locator('input[name="latitude"]').fill('40.7128');
  await page.locator('input[name="longitude"]').fill('-74.0060');
  await page.locator('button:has-text("Add Location")').click();

  // Assert
  await expect(page.locator('text=Location Provisioned')).toBeVisible();
  await expect(page.locator('text=New City')).toBeVisible();
});
```

### Test Coverage Requirements

- New features: Add E2E tests
- Bug fixes: Add regression test
- Refactoring: No new tests needed (existing should pass)

---

## Documentation

### Update Documentation When Needed

```markdown
When to update docs:
- New feature added → Update COMPONENTS.md or ARCHITECTURE.md
- API changes → Update API_INTEGRATION.md
- Setup changes → Update SETUP.md
- New development pattern → Update DEVELOPMENT.md

Documentation is part of the PR!
```

### Documentation Format

- Use clear, concise language
- Include code examples
- Add ASCII diagrams where helpful
- Keep files up-to-date
- Link between related sections

---

## Reporting Issues

### Bug Report Template

```markdown
## Description
Clear description of the bug

## Steps to Reproduce
1. Login as user@example.com
2. Navigate to Locations page
3. Click "Add Location"
4. Observe the bug

## Expected Behavior
What should happen?

## Actual Behavior
What actually happens?

## Environment
- Browser: Chrome 120
- OS: macOS 14.2
- Node version: 18.17.0

## Screenshots
[Attach relevant screenshots]

## Additional Context
Any other information that helps understand the issue
```

### Feature Request Template

```markdown
## Description
Description of the requested feature

## Motivation
Why is this feature needed?

## Proposed Solution
How should this feature work?

## Alternatives
Other ways to solve this problem?

## Example Use Case
How would users use this feature?
```

---

## Getting Help

### Resources

- **Documentation**: [DOCUMENTATION_INDEX.md](./DOCUMENTATION_INDEX.md)
- **Setup Guide**: [SETUP.md](./SETUP.md)
- **Development Guide**: [DEVELOPMENT.md](./DEVELOPMENT.md)
- **Architecture**: [ARCHITECTURE.md](./ARCHITECTURE.md)

### Communication Channels

- **GitHub Issues**: For bugs and feature requests
- **GitHub Discussions**: For questions and ideas
- **Slack #development**: For real-time help
- **Email**: development@weatherops.com

### Asking for Help

When asking questions:
1. Describe what you're trying to do
2. Show what you've tried
3. Share error messages or logs
4. Include environment details

---

## Review Process

### What Reviewers Look For

✅ **Code Quality**
- Follows coding standards
- Type-safe (no `any`)
- Clear variable names
- Proper error handling

✅ **Testing**
- Tests added/updated
- Tests pass
- Coverage adequate

✅ **Documentation**
- Comments explain "why"
- Docs updated if needed
- Commit messages clear

✅ **Performance**
- No performance regression
- No unnecessary re-renders
- Bundle size impact acceptable

### Common Review Feedback

```
❓ "Can you add types here?"
→ Add TypeScript interfaces/types

❓ "This needs error handling"
→ Wrap in try-catch, show user feedback

❓ "Does this have tests?"
→ Add E2E test for the feature

❓ "Can you add a comment explaining this?"
→ Add comment explaining "why" not "what"
```

---

## Contribution Recognition

We recognize and appreciate all contributions:

- First-time contributors get a badge
- Regular contributors listed in CONTRIBUTORS.md
- Major contributors acknowledged in release notes

---

## Questions?

Don't hesitate to ask! We're here to help.

- **GitHub Issues**: Ask on the issue
- **Slack**: Post in #development
- **Email**: development@weatherops.com

---

**Thank you for contributing to WeatherOps!** 🚀

---

**Last Updated**: June 2026  
**Version**: 1.0.0
