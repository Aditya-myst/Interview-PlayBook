# 07 — ORMs & Data Access

## Prisma, SQLAlchemy, Hibernate, Entity Framework

---

### What is an ORM?

An **Object-Relational Mapper (ORM)** maps database tables to programming language objects. You write code instead of raw SQL.

```javascript
// Without ORM (raw SQL)
const users = await db.query('SELECT * FROM users WHERE age > ?', [18]);

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

*Next: [08 — Backend Interview Q&A](08-Interview-QA.md)*
