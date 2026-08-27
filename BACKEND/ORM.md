# 07 — ORMs & Data Access 

## Prisma, SQLAlchemy, Hibernate, Entity Framework

---

### What is an ORM?

An **Object-Relational Mapper (ORM)** maps database tables to programming language objects. You write code instead of raw SQL.

```javascript
// Without ORM (raw SQL)
const users = await db.query('SELECT * FROM users WHERE age > ?', [18]);
# 07 — ORMs & Data Access

## Prisma, SQLAlchemy, Hibernate — 20+ Interview Questions

---

### Prisma (Node.js/TypeScript)

```prisma
model User {
    id        Int      @id @default(autoincrement())
    email     String   @unique
    name      String
    posts     Post[]
    createdAt DateTime @default(now())
}

model Post {
    id        Int      @id @default(autoincrement())
    title     String
    content   String?
    published Boolean  @default(false)
    author    User     @relation(fields: [authorId], references: [id])
    authorId  Int
}
```

```javascript
// CRUD
const user = await prisma.user.create({ data: { email: 'alice@example.com', name: 'Alice' } });
const user = await prisma.user.findUnique({ where: { id: 1 }, include: { posts: true } });
const users = await prisma.user.findMany({ where: { age: { gte: 18 } }, take: 10 });
await prisma.user.update({ where: { id: 1 }, data: { name: 'Alice Smith' } });
await prisma.user.delete({ where: { id: 1 } });

// Transactions
await prisma.$transaction([
    prisma.user.create({ data: { name: 'Bob' } }),
    prisma.post.create({ data: { title: 'Hello', authorId: 1 } })
]);
```

---

### SQLAlchemy (Python)

```python
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    name = Column(String(100))
    email = Column(String(100), unique=True)

session = Session()
user = User(name='Alice', email='alice@example.com')
session.add(user)
session.commit()
```

---

### Interview Questions (20+)

**Q1: What is an ORM?**
A: "Object-Relational Mapper maps database tables to language objects. Benefits: type safety, rapid development, database abstraction. Trade-offs: performance overhead, N+1 problem, learning curve."

**Q2: What's the N+1 problem?**
A: "Fetching list, then querying related data for each item. 1 query for users + N queries for posts = N+1. Solution: eager loading (JOIN) or DataLoader."

**Q3: ORM vs raw SQL—which would you choose?**
A: "ORM for CRUD, type safety, rapid development. Raw SQL for complex queries, analytics, bulk operations. Most projects use both: ORM for CRUD, raw SQL for complex queries."

**Q4: What is eager vs lazy loading?**
A: "Eager: load related data immediately (JOIN). Lazy: load related data on access (separate query). Eager prevents N+1 but may over-fetch. Lazy is simpler but causes N+1."

**Q5: How do you handle database migrations?**
A: "Version control for schema changes. Tools: Prisma Migrate, Alembic, Flyway. Track changes, rollback safely. Run in CI/CD pipeline."

**Q6: What is database seeding?**
A: "Populate database with initial data. Use for development, testing, default data. Create seed scripts. Run after migrations."

**Q7: How do you optimize ORM queries?**
A: "Use select to limit fields. Use lean() for read-only. Avoid N+1 with eager loading. Use database indexes. Profile queries."

**Q8: What is query builder vs ORM?**
A: "Query builder: construct SQL programmatically (Knex.js). ORM: map tables to objects (Sequelize). Query builder more flexible; ORM more abstracted."

**Q9: How do you handle transactions in ORMs?**
A: "Wrap multiple operations in transaction. Rollback on error. Commit on success. Most ORMs support transactions natively."

**Q10: What is ActiveRecord vs Data Mapper pattern?**
A: "ActiveRecord: model knows how to persist itself (Sequelize). Data Mapper: separate mapper handles persistence (Prisma, SQLAlchemy). Data Mapper more flexible."

**Q11: How do you test ORM code?**
A: "Use in-memory database (SQLite). Mock database layer. Integration tests with test database. Transaction rollback after each test."

**Q12: What is connection pooling in ORMs?**
A: "Reuse database connections. Configure pool size. Handle connection errors. Most ORMs include connection pooling."

**Q13: How do you handle soft deletes?**
A: "Mark records as deleted instead of removing. Add deletedAt column. Filter out in queries. Use ORM hooks or scopes."

**Q14: What is database schema introspection?**
A: "Read database structure programmatically. Used for code generation, documentation. Prisma uses introspection for schema generation."

**Q15: How do you handle complex queries with ORMs?**
A: "Use raw SQL for complex queries. Use query builder for dynamic queries. Use ORM for simple CRUD. Combine approaches as needed."

**Q16: What is ORM performance optimization?**
A: "Avoid N+1 queries. Use select to limit fields. Use indexes. Profile queries. Use raw SQL for complex operations."

**Q17: How do you handle database constraints in ORMs?**
A: "Define in schema (unique, not null). Validate in application layer. Handle constraint violations gracefully."

**Q18: What is database relationship mapping?**
A: "One-to-one, one-to-many, many-to-many. Define in ORM schema. Use foreign keys. Handle cascading deletes."

**Q19: How do you handle database views in ORMs?**
A: "Map views as read-only models. Use for complex queries. Refresh materialized views on schedule."

**Q20: What is ORM hook/middleware?**
A: "Execute code before/after operations. Use for validation, logging, auditing. Examples: beforeCreate, afterUpdate hooks."

---

### Complete Prisma Implementation

```javascript
const { PrismaClient } = require('@prisma/client');

const prisma = new PrismaClient({
    log: ['query', 'info', 'warn', 'error'],
    errorFormat: 'pretty'
});

// User Repository
class UserRepository {
    async findById(id) {
        return prisma.user.findUnique({
            where: { id },
            include: { posts: true, profile: true }
        });
    }
    
    async findByEmail(email) {
        return prisma.user.findUnique({ where: { email } });
    }
    
    async create(data) {
        return prisma.user.create({ data });
    }
    
    async update(id, data) {
        return prisma.user.update({ where: { id }, data });
    }
    
    async delete(id) {
        return prisma.user.delete({ where: { id } });
    }
    
    async findMany({ page = 1, limit = 10, filters = {}, sort = 'createdAt' }) {
        const skip = (page - 1) * limit;
        const [users, total] = await Promise.all([
            prisma.user.findMany({
                where: filters,
                skip,
                take: limit,
                orderBy: { [sort]: 'desc' },
                include: { posts: { where: { published: true } } }
            }),
            prisma.user.count({ where: filters })
        ]);
        
        return { users, total, page, limit, pages: Math.ceil(total / limit) };
    }
}

// Transaction example
async function transferFunds(fromId, toId, amount) {
    return prisma.$transaction(async (tx) => {
        const from = await tx.account.update({
            where: { id: fromId },
            data: { balance: { decrement: amount } }
        });
        
        if (from.balance < 0) {
            throw new Error('Insufficient funds');
        }
        
        const to = await tx.account.update({
            where: { id: toId },
            data: { balance: { increment: amount } }
        });
        
        await tx.transaction.create({
            data: { fromId, toId, amount }
        });
        
        return { from, to };
    });
}
```

---

### Additional Interview Questions (15+)

**Q21: How do you implement pagination with Prisma?**
A: "Use skip and take for offset pagination. Use cursor-based pagination with cursor option for better performance. Include total count for metadata."

**Q22: What is Prisma middleware?**
A: "Execute code before/after queries. Use for logging, soft deletes, auditing. Implement with $use method. Can modify queries and results."

**Q23: How do you handle database errors with ORMs?**
A: "Catch specific error types (PrismaClientKnownRequestError). Handle constraint violations (unique, foreign key). Return appropriate error messages."

**Q24: What is ORM query optimization?**
A: "Use select to limit fields. Use include for eager loading. Use where to filter early. Avoid N+1 with DataLoader. Profile with logging."

**Q25: How do you implement soft deletes with Prisma?**
A: "Add deletedAt field. Use middleware to filter deleted records. Implement restore functionality. Override delete to set deletedAt."

**Q26: What is Prisma accelerate?**
A: "Connection pooling and caching for serverless. Reduces cold start latency. Global database access. Built-in caching."

**Q27: How do you handle database connections in serverless?**
A: "Connection pooling (Prisma Accelerate, PgBouncer). Limit connections per function. Use connection string pooling. Handle connection exhaustion."

**Q28: What is ORM vs query builder?**
A: "ORM: map tables to objects (Prisma, Sequelize). Query builder: construct SQL (Knex.js). ORM for CRUD; query builder for complex queries."

**Q29: How do you test Prisma code?**
A: "Use test database. Mock Prisma client. Integration tests with real database. Transaction rollback after tests. Use factory patterns."

**Q30: What is Prisma schema best practices?**
A: "Use meaningful names. Add indexes for queries. Use enums for fixed values. Document relations. Version control schema."

**Q31: How do you handle database seeding?**
A: "Create seed scripts. Use Prisma seed command. Generate realistic data. Run after migrations. Use for development and testing."

**Q32: What is Prisma migrate?**
A: "Database migration tool. Version control for schema. Generate SQL migrations. Apply migrations. Rollback support."

**Q33: How do you handle complex queries with Prisma?**
A: "Use raw SQL with $queryRaw. Use query builder for dynamic queries. Combine ORM for CRUD with raw for analytics."

**Q34: What is Prisma client generation?**
A: "Auto-generate type-safe client from schema. Supports TypeScript. Updates on schema changes. Reduces boilerplate."

**Q35: How do you handle database transactions?**
A: "Use $transaction for multiple operations. Handle rollback on error. Use interactive transactions for complex logic. Ensure atomicity."

---

*Next: [08 — Microservices Architecture](08-Microservices.md)*

// With ORM (Prisma)
const users = await prisma.user.findMany({
    where: { age: { gt: 18 } }
});
```

---

### ORMs by Language

| Language | ORM | Database Support |
|----------|-----|-----------------|
| JavaScript/TypeScript | Prisma, Sequelize, TypeORM | PostgreSQL, MySQL, SQLite |
| Python | SQLAlchemy, Django ORM | PostgreSQL, MySQL, SQLite |
| Java | Hibernate, JPA | PostgreSQL, MySQL, Oracle |
| C# | Entity Framework | SQL Server, PostgreSQL, MySQL |

---

### Prisma (Node.js/TypeScript)

#### Setup

```prisma
// schema.prisma
datasource db {
    provider = "postgresql"
    url      = env("DATABASE_URL")
}

generator client {
    provider = "prisma-client-js"
}

model User {
    id        Int      @id @default(autoincrement())
    email     String   @unique
    name      String
    age       Int?
    posts     Post[]
    createdAt DateTime @default(now())
    updatedAt DateTime @updatedAt
}

model Post {
    id        Int      @id @default(autoincrement())
    title     String
    content   String?
    published Boolean  @default(false)
    author    User     @relation(fields: [authorId], references: [id])
    authorId  Int
    createdAt DateTime @default(now())
}
```

#### CRUD Operations

```javascript
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();

// Create
const user = await prisma.user.create({
    data: {
        email: 'alice@example.com',
        name: 'Alice',
        age: 30
    }
});

// Read
const user = await prisma.user.findUnique({
    where: { id: 1 }
});

// Read with relations
const userWithPosts = await prisma.user.findUnique({
    where: { id: 1 },
    include: { posts: true }
});

// Update
const updated = await prisma.user.update({
    where: { id: 1 },
    data: { name: 'Alice Smith' }
});

// Delete
await prisma.user.delete({
    where: { id: 1 }
});

// Complex queries
const users = await prisma.user.findMany({
    where: {
        age: { gte: 18 },
        posts: { some: { published: true } }
    },
    include: {
        posts: {
            where: { published: true },
            orderBy: { createdAt: 'desc' },
            take: 5
        }
    },
    skip: 0,
    take: 10
});
```

#### Transactions

```javascript
// Interactive transactions
const [user, post] = await prisma.$transaction([
    prisma.user.create({ data: { email: 'bob@example.com', name: 'Bob' } }),
    prisma.post.create({ data: { title: 'Hello', authorId: 1 } })
]);

// Sequential transactions
await prisma.$transaction(async (tx) => {
    const user = await tx.user.create({ data: { email: 'carol@example.com', name: 'Carol' } });
    await tx.post.create({ data: { title: 'First Post', authorId: user.id } });
});
```

---

### SQLAlchemy (Python)

#### Setup

```python
from sqlalchemy import create_engine, Column, Integer, String, ForeignKey, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship, sessionmaker

engine = create_engine('postgresql://user:pass@localhost/mydb')
Session = sessionmaker(bind=engine)
Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    email = Column(String(100), unique=True, nullable=False)
    age = Column(Integer)
    
    posts = relationship('Post', back_populates='author')

class Post(Base):
    __tablename__ = 'posts'
    
    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    content = Column(String)
    author_id = Column(Integer, ForeignKey('users.id'))
    
    author = relationship('User', back_populates='posts')

Base.metadata.create_all(engine)
```

#### CRUD Operations

```python
session = Session()

# Create
user = User(name='Alice', email='alice@example.com', age=30)
session.add(user)
session.commit()

# Read
user = session.query(User).filter_by(id=1).first()
users = session.query(User).filter(User.age >= 18).all()

# Update
user = session.query(User).filter_by(id=1).first()
user.name = 'Alice Smith'
session.commit()

# Delete
user = session.query(User).filter_by(id=1).first()
session.delete(user)
session.commit()

# Complex queries
users = session.query(User)\
    .join(User.posts)\
    .filter(Post.published == True)\
    .order_by(User.name)\
    .limit(10)\
    .all()

# Transaction
try:
    user = User(name='Bob', email='bob@example.com')
    session.add(user)
    session.flush()  # Get user.id
    
    post = Post(title='Hello', author_id=user.id)
    session.add(post)
    session.commit()
except:
    session.rollback()
    raise
```

---

### Hibernate (Java)

#### Entity

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    private Integer age;
    
    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL)
    private List<Post> posts = new ArrayList<>();
    
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    private LocalDateTime updatedAt;
    
    // Getters and setters
}

@Entity
@Table(name = "posts")
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String title;
    
    private String content;
    
    @ManyToOne
    @JoinColumn(name = "author_id")
    private User author;
}
```

#### Repository (Spring Data JPA)

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByAgeGreaterThan(Integer age);
    Optional<User> findByEmail(String email);
    
    @Query("SELECT u FROM User u WHERE u.age >= :minAge")
    List<User> findUsersAboveAge(@Param("minAge") Integer minAge);
}
```

#### Service

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    @Transactional
    public User createUser(User user) {
        return userRepository.save(user);
    }
    
    public List<User> getActiveUsers() {
        return userRepository.findByAgeGreaterThan(18);
    }
}
```

---

### Entity Framework (C#)

```csharp
// DbContext
public class AppDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
    public DbSet<Post> Posts { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<User>()
            .HasMany(u => u.Posts)
            .WithOne(p => p.Author)
            .HasForeignKey(p => p.AuthorId);
    }
}

// Entity
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public int? Age { get; set; }
    public List<Post> Posts { get; set; }
}

// CRUD
using var context = new AppDbContext();

// Create
context.Users.Add(new User { Name = "Alice", Email = "alice@example.com" });
await context.SaveChangesAsync();

// Read
var user = await context.Users.FindAsync(1);
var users = await context.Users.Where(u => u.Age >= 18).ToListAsync();

// Update
user.Name = "Alice Smith";
await context.SaveChangesAsync();

// Delete
context.Users.Remove(user);
await context.SaveChangesAsync();
```

---

### N+1 Problem

```javascript
// BAD: N+1 queries
const users = await prisma.user.findMany();  // 1 query
for (const user of users) {
    const posts = await prisma.post.findMany({
        where: { authorId: user.id }
    });  // N queries!
}

// GOOD: Eager loading
const users = await prisma.user.findMany({
    include: { posts: true }  // 1 query with JOIN
});
```

---

### When NOT to Use ORMs

| Use ORM | Use Raw SQL |
|---------|-------------|
| CRUD operations | Complex analytics queries |
| Rapid development | Performance-critical queries |
| Type safety | Bulk operations |
| Simple to moderate queries | Database-specific features |

---

### Interview Questions

**Q: What is an ORM and when would you use one?**

A: "An ORM maps database tables to objects. Use for CRUD operations, rapid development, type safety. Don't use for complex analytics, bulk operations, or when performance is critical. Popular: Prisma (JS), SQLAlchemy (Python), Hibernate (Java)."

**Q: What's the N+1 problem?**

A: "Fetching a list, then querying related data for each item. 1 query for users + N queries for posts = N+1 queries. Solution: eager loading (JOIN) or batching (DataLoader). Always check your query count in development."

**Q: ORM vs raw SQL—which would you choose?**

A: "ORM for most application code—type safety, rapid development, maintainability. Raw SQL for complex queries, analytics, bulk operations, or when you need database-specific features. Most projects use both: ORM for CRUD, raw SQL for complex queries."

---

*Next: [08 — Backend Interview Q&A](https://github.com/Aditya-myst/Interview-PlayBook/blob/main/BACKEND/Interview-qa.md)*
