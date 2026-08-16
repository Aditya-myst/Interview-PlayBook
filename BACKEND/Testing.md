# 12 — Testing Strategies

## Unit, Integration, E2E, TDD — 25+ Interview Questions

---

### Testing Pyramid

```
        /\
       /  \  E2E Tests (few)
      /----\
     /      \  Integration Tests (some)
    /--------\
   /          \  Unit Tests (many)
  /------------\
```

---

### Unit Testing (Jest)

```javascript
describe('User utilities', () => {
    test('validates correct email', () => {
        expect(validateEmail('alice@example.com')).toBe(true);
    });
    
    test('rejects invalid email', () => {
        expect(validateEmail('invalid')).toBe(false);
    });
    
    test('calculates age correctly', () => {
        expect(calculateAge('1990-01-01')).toBe(35);
    });
});
```

---

### Integration Testing

```javascript
describe('User API', () => {
    it('creates a new user', async () => {
        const res = await request(app)
            .post('/api/users')
            .send({ name: 'Alice', email: 'alice@example.com' });
        
        expect(res.status).toBe(201);
        expect(res.body.data.name).toBe('Alice');
    });
    
    it('returns 400 for invalid email', async () => {
        const res = await request(app)
            .post('/api/users')
            .send({ name: 'Alice', email: 'invalid' });
        
        expect(res.status).toBe(400);
    });
});
```

---

### Interview Questions (25+)

**Q1: What's the testing pyramid?**
A: "Many unit tests (fast, isolated), some integration tests (test interactions), few E2E tests (slow, comprehensive). Unit tests catch bugs early; integration tests verify components work together."

**Q2: What is TDD?**
A: "Test-Driven Development: write failing test, write minimal code to pass, refactor. Benefits: better design, documentation, confidence in changes."

**Q3: Unit vs integration vs E2E tests?**
A: "Unit: test individual functions in isolation. Integration: test component interactions. E2E: test complete user flows. Unit fast and cheap; E2E slow and expensive."

**Q4: How do you test async code?**
A: "Use async/await in tests. Return promises. Jest handles async automatically. Use done callback for callbacks."

**Q5: What is mocking?**
A: "Replace dependencies with fake implementations. Mock external services, databases. Control behavior for testing. Jest.mock(), sinon."

**Q6: How do you test API endpoints?**
A: "Integration tests with supertest. Test different scenarios (success, error, edge cases). Mock external dependencies. Test authentication."

**Q7: What is code coverage?**
A: "Percentage of code executed by tests. Tools: Istanbul, Jest coverage. Don't chase 100%; focus on critical paths."

**Q8: How do you test database operations?**
A: "Use test database. Transaction rollback after each test. Or use in-memory database. Mock for unit tests."

**Q9: What is snapshot testing?**
A: "Capture output and compare with future runs. Good for UI components, API responses. Update snapshots when intentional changes."

**Q10: How do you test error handling?**
A: "Test error scenarios explicitly. Mock errors from dependencies. Verify error messages and status codes. Test recovery logic."

**Q11: What is contract testing?**
A: "Verify API matches documentation. Consumer-driven contracts (Pact). Ensures backward compatibility between services."

**Q12: How do you test authentication?**
A: "Test login/logout flows. Test token validation. Test authorization (different roles). Mock external auth providers."

**Q13: What is load testing?**
A: "Test system under expected load. Tools: k6, Artillery, JMeter. Measure response times, throughput, error rates."

**Q14: How do you test third-party integrations?**
A: "Mock external services. Use sandbox environments. Test error scenarios. Contract tests for API compatibility."

**Q15: What is test-driven development benefits?**
A: "Better design (testable code). Documentation (tests as specs). Confidence in refactoring. Fewer bugs in production."

**Q16: How do you test file uploads?**
A: "Mock file system. Test with different file types and sizes. Validate file processing. Test error scenarios."

**Q17: What is mutation testing?**
A: "Introduce bugs (mutations) and check if tests catch them. Tools: Stryker. Measures test quality, not just coverage."

**Q18: How do you test WebSocket connections?**
A: "Mock WebSocket server. Test message sending/receiving. Test connection/disconnection. Test error handling."

**Q19: What is test fixture?**
A: "Fixed state for tests. Setup and teardown. Database seeds. Mock data. Reusable test data."

**Q20: How do you test caching logic?**
A: "Mock Redis/cache. Test cache hit/miss. Test invalidation. Test cache stampede prevention."

**Q21: What is smoke testing?**
A: "Basic tests to verify system works. Run after deployment. Critical path only. Fast feedback."

**Q22: How do you test error boundaries?**
A: "Render component that throws. Verify fallback UI. Test error reporting. Test recovery."

**Q23: What is property-based testing?**
A: "Generate random inputs. Test properties that should hold. Libraries: fast-check. Finds edge cases."

**Q24: How do you test performance?**
A: "Load testing (k6). Benchmark critical paths. Profile memory/CPU. Set performance budgets."

**Q25: What is CI/CD testing strategy?**
A: "Run unit tests on every commit. Integration tests on PR. E2E tests before deployment. Performance tests periodically."

---

### Complete Testing Implementation

```javascript
const request = require('supertest');
const app = require('../app');
const User = require('../models/User');

// Test setup
beforeAll(async () => {
    await connectTestDatabase();
});

afterAll(async () => {
    await disconnectTestDatabase();
});

beforeEach(async () => {
    await User.deleteMany({});
});

// Unit tests
describe('User Model', () => {
    test('validates required fields', async () => {
        const user = new User({});
        const error = user.validateSync();
        expect(error.errors.email).toBeDefined();
        expect(error.errors.name).toBeDefined();
    });
    
    test('hashes password before save', async () => {
        const user = await User.create({
            name: 'Alice',
            email: 'alice@example.com',
            password: 'password123'
        });
        expect(user.password).not.toBe('password123');
        expect(user.password).toMatch(/^\$2[aby]\$\d{12}\$/);
    });
    
    test('compares password correctly', async () => {
        const user = await User.create({
            name: 'Alice',
            email: 'alice@example.com',
            password: 'password123'
        });
        expect(await user.comparePassword('password123')).toBe(true);
        expect(await user.comparePassword('wrong')).toBe(false);
    });
});

// Integration tests
describe('User API', () => {
    describe('POST /api/users', () => {
        test('creates a new user', async () => {
            const res = await request(app)
                .post('/api/users')
                .send({ name: 'Alice', email: 'alice@example.com', password: 'Pass123!' });
            
            expect(res.status).toBe(201);
            expect(res.body.data.name).toBe('Alice');
            expect(res.body.data.email).toBe('alice@example.com');
            expect(res.body.data.password).toBeUndefined();
        });
        
        test('returns 400 for invalid email', async () => {
            const res = await request(app)
                .post('/api/users')
                .send({ name: 'Alice', email: 'invalid', password: 'Pass123!' });
            
            expect(res.status).toBe(400);
            expect(res.body.error.code).toBe('VALIDATION_ERROR');
        });
        
        test('returns 409 for duplicate email', async () => {
            await User.create({ name: 'Alice', email: 'alice@example.com', password: 'Pass123!' });
            
            const res = await request(app)
                .post('/api/users')
                .send({ name: 'Alice', email: 'alice@example.com', password: 'Pass123!' });
            
            expect(res.status).toBe(409);
        });
    });
    
    describe('GET /api/users', () => {
        test('returns paginated users', async () => {
            await User.create([
                { name: 'Alice', email: 'alice@example.com', password: 'Pass123!' },
                { name: 'Bob', email: 'bob@example.com', password: 'Pass123!' }
            ]);
            
            const res = await request(app)
                .get('/api/users')
                .query({ page: 1, limit: 10 });
            
            expect(res.status).toBe(200);
            expect(res.body.data).toHaveLength(2);
            expect(res.body.pagination.total).toBe(2);
        });
    });
});

// Mocking
describe('Email Service', () => {
    test('sends welcome email', async () => {
        const sendEmail = jest.spyOn(emailService, 'sendEmail').mockResolvedValue(true);
        
        await userService.createUser({ name: 'Alice', email: 'alice@example.com' });
        
        expect(sendEmail).toHaveBeenCalledWith('alice@example.com', 'welcome');
        sendEmail.mockRestore();
    });
});
```

---

### Additional Interview Questions (20+)

**Q26: How do you test middleware?**
A: "Test with mock req/res objects. Verify next() is called. Test error scenarios. Use supertest for HTTP middleware."

**Q27: What is test isolation?**
A: "Each test runs independently. No shared state between tests. Use beforeEach/afterEach for setup/teardown. Use test database."

**Q28: How do you test async operations?**
A: "Use async/await in tests. Return promises. Jest handles async automatically. Use done callback for callbacks."

**Q29: What is test coverage tools?**
A: "Istanbul/nyc for JavaScript. JaCoCo for Java. Coverage.py for Python. Don't chase 100%; focus on critical paths."

**Q30: How do you test error handling?**
A: "Test error scenarios explicitly. Mock errors from dependencies. Verify error messages and status codes. Test recovery logic."

**Q31: What is contract testing?**
A: "Verify API matches documentation. Consumer-driven contracts (Pact). Ensures backward compatibility between services."

**Q32: How do you test database migrations?**
A: "Run migrations on test database. Verify schema changes. Test rollback. Test data integrity."

**Q33: What is load testing tools?**
A: "k6, Artillery, JMeter, Gatling. Measure response times, throughput, error rates. Set performance budgets."

**Q34: How do you test authentication?**
A: "Test login/logout flows. Test token validation. Test authorization (different roles). Mock external auth providers."

**Q35: What is snapshot testing?**
A: "Capture output and compare with future runs. Good for UI components, API responses. Update snapshots when intentional changes."

**Q36: How do you test WebSocket?**
A: "Mock WebSocket server. Test message sending/receiving. Test connection/disconnection. Test error handling."

**Q37: What is mutation testing?**
A: "Introduce bugs (mutations) and check if tests catch them. Tools: Stryker. Measures test quality, not just coverage."

**Q38: How do you test file uploads?**
A: "Mock file system. Test with different file types and sizes. Validate file processing. Test error scenarios."

**Q39: What is test fixture?**
A: "Fixed state for tests. Setup and teardown. Database seeds. Mock data. Reusable test data."

**Q40: How do you test caching logic?**
A: "Mock Redis/cache. Test cache hit/miss. Test invalidation. Test cache stampede prevention."

**Q41: What is smoke testing?**
A: "Basic tests to verify system works. Run after deployment. Critical path only. Fast feedback."

**Q42: How do you test error boundaries?**
A: "Render component that throws. Verify fallback UI. Test error reporting. Test recovery."

**Q43: What is property-based testing?**
A: "Generate random inputs. Test properties that should hold. Libraries: fast-check. Finds edge cases."

**Q44: How do you test performance?**
A: "Load testing (k6). Benchmark critical paths. Profile memory/CPU. Set performance budgets."

**Q45: What is CI/CD testing strategy?**
A: "Run unit tests on every commit. Integration tests on PR. E2E tests before deployment. Performance tests periodically."

---

*Next: [13 — Node.js Deep Dive](13-NodeJS.md)*
