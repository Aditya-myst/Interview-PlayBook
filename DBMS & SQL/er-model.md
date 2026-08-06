# 02 — ER Model

## Designing Database Schemas

---

### What is the ER Model?

The **Entity-Relationship (ER) Model** is a conceptual data model used to design databases. It describes data in terms of **entities**, **attributes**, and **relationships**.

**Why it matters:** Before writing SQL, you need to design the schema. ER diagrams are the standard way to visualize and communicate database designs.

---

### ER Diagram Components

```
┌─────────────────────────────────────────────────────────┐
│                    ER Diagram                            │
│                                                         │
│  ┌───────┐         ┌───────────┐         ┌───────┐    │
│  │Entity │────────>│Relationship│<────────│Entity │    │
│  │(Rect) │  (Line) │ (Diamond) │ (Line)  │(Rect) │    │
│  └───────┘         └───────────┘         └───────┘    │
│       │                                     │          │
│       │                                     │          │
│  ┌────┴────┐                           ┌────┴────┐    │
│  │Attribute│                           │Attribute│    │
│  │(Oval)   │                           │(Oval)   │    │
│  └─────────┘                           └─────────┘    │
└─────────────────────────────────────────────────────────┘
```

---

### Entities

An **entity** is a real-world object or concept.

| Type | Description | Example |
|------|-------------|---------|
| **Strong Entity** | Has its own primary key | Student, Course |
| **Weak Entity** | Depends on strong entity for identification | Dependent (of Employee) |

```
Strong Entity:          Weak Entity:
┌──────────────┐       ┌──────────────┐
│   Student    │       │   Dependent  │
├──────────────┤       ├──────────────┤
│ PK: ID       │       │ PK: (EmpID,  │
│    Name      │       │     Name)    │
│    Email     │       │    Relation  │
└──────────────┘       └──────────────┘
```

---

### Attributes

| Type | Description | Symbol |
|------|-------------|--------|
| **Simple** | Cannot be divided | (Oval) |
| **Composite** | Can be divided | (Oval with sub-ovals) |
| **Derived** | Calculated from other attributes | (Dashed oval) |
| **Multi-valued** | Can have multiple values | (Double oval) |
| **Key Attribute** | Uniquely identifies entity | (Underlined oval) |

```
Student Entity:
                    ┌─────────┐
                    │StudentID│ (key, underlined)
                    └────┬────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   ┌────┴────┐     ┌────┴────┐     ┌────┴────┐
   │  Name   │     │ Address │     │  Phone  │
   │(composite)    │(composite)    │(multi-valued)
   └────┬────┘     └────┬────┘     └─────────┘
        │                │
   ┌────┴────┐     ┌────┴────┐
   │First    │     │City     │
   │Last     │     │State    │
   └─────────┘     │Zip      │
                   └─────────┘
```

---

### Relationships

| Type | Description | Example |
|------|-------------|---------|
| **Unary** | Entity relates to itself | Employee manages Employee |
| **Binary** | Two entities | Student enrolls in Course |
| **Ternary** | Three entities | Doctor prescribes Medicine to Patient |

---

### Cardinality

How many instances of one entity relate to another.

```
One-to-One (1:1):
┌─────────┐         ┌─────────┐
│ Person  │────1:1──│Passport │
└─────────┘         └─────────┘

One-to-Many (1:N):
┌─────────┐         ┌─────────┐
│Department│───1:N──│Employee │
└─────────┘         └─────────┘

Many-to-Many (M:N):
┌─────────┐         ┌─────────┐
│ Student │───M:N───│ Course  │
└─────────┘         └─────────┘
```

**Cardinality Notations:**

| Notation | 1:1 | 1:N | M:N |
|----------|-----|-----|-----|
| **Chen** | 1—1 | 1—N | M—N |
| **Crow's Foot** | ──┤├── | ──┤< | ──>< |

---

### Participation

| Type | Description | Example |
|------|-------------|---------|
| **Total** | Every entity must participate | Every employee MUST belong to a department |
| **Partial** | Some entities may not participate | A department MAY have no employees |

```
Total Participation:    Partial Participation:
(Employee MUST         (Department MAY
 have Department)       have no Employees)
      │                      │
      │                      │
  ┌───┴───┐              ┌───┴───┐
  │  ──   │ (double      │  ──   │ (single
  │  line)│  line)       │  line)│  line)
  └───────┘              └───────┘
```

---

### ER to Relational Schema Conversion

#### Strong Entity → Table
```
ER: Student (ID, Name, Email)
SQL: CREATE TABLE Student (
    ID INT PRIMARY KEY,
    Name VARCHAR(100),
    Email VARCHAR(100)
);
```

#### 1:1 Relationship → Foreign Key in either table
```
ER: Person ──1:1── Passport
SQL: CREATE TABLE Person (
    ID INT PRIMARY KEY,
    Name VARCHAR(100),
    PassportID INT UNIQUE,
    FOREIGN KEY (PassportID) REFERENCES Passport(ID)
);
```

#### 1:N Relationship → Foreign Key in "Many" side
```
ER: Department ──1:N── Employee
SQL: CREATE TABLE Employee (
    ID INT PRIMARY KEY,
    Name VARCHAR(100),
    DeptID INT,
    FOREIGN KEY (DeptID) REFERENCES Department(ID)
);
```

#### M:N Relationship → New table with both foreign keys
```
ER: Student ──M:N── Course (via Enrollment)
SQL: CREATE TABLE Enrollment (
    StudentID INT,
    CourseID INT,
    Grade CHAR(1),
    PRIMARY KEY (StudentID, CourseID),
    FOREIGN KEY (StudentID) REFERENCES Student(ID),
    FOREIGN KEY (CourseID) REFERENCES Course(ID)
);
```

#### Weak Entity → Composite primary key including owner's key
```
ER: Dependent (weak) of Employee (strong)
SQL: CREATE TABLE Dependent (
    EmpID INT,
    Name VARCHAR(100),
    Relation VARCHAR(50),
    PRIMARY KEY (EmpID, Name),
    FOREIGN KEY (EmpID) REFERENCES Employee(ID)
);
```

---

### Complete Example: University Database

```
ER Diagram:
┌──────────┐     M:N      ┌──────────┐
│ Student  │──────────────│  Course  │
│          │  (Enrollment)│          │
│ PK: ID   │              │ PK: ID   │
│    Name  │              │    Title │
│    Email │              │    Credits│
└──────────┘              └────┬─────┘
                               │ 1:N
                               │
                          ┌────┴─────┐
                          │Professor │
                          │PK: ID    │
                          │   Name   │
                          │   Dept   │
                          └──────────┘

SQL Schema:
CREATE TABLE Student (
    ID INT PRIMARY KEY,
    Name VARCHAR(100),
    Email VARCHAR(100) UNIQUE
);

CREATE TABLE Professor (
    ID INT PRIMARY KEY,
    Name VARCHAR(100),
    Dept VARCHAR(50)
);

CREATE TABLE Course (
    ID INT PRIMARY KEY,
    Title VARCHAR(100),
    Credits INT,
    ProfID INT,
    FOREIGN KEY (ProfID) REFERENCES Professor(ID)
);

CREATE TABLE Enrollment (
    StudentID INT,
    CourseID INT,
    Grade CHAR(1),
    Semester VARCHAR(20),
    PRIMARY KEY (StudentID, CourseID, Semester),
    FOREIGN KEY (StudentID) REFERENCES Student(ID),
    FOREIGN KEY (CourseID) REFERENCES Course(ID)
);
```

---

### Interview Questions

**Q: What is an ER diagram?**

A: "An ER diagram is a visual representation of a database schema showing entities, attributes, and relationships. Entities are rectangles, attributes are ovals, relationships are diamonds. It's used during the design phase to communicate the database structure before implementation."

**Q: What's the difference between a strong and weak entity?**

A: "A strong entity has its own primary key and exists independently (Student, Course). A weak entity depends on a strong entity for identification—it has a partial key and total participation with the owner (Dependent of Employee, OrderItem of Order)."

**Q: How do you convert M:N relationships to tables?**

A: "Create a new junction (bridge) table with foreign keys to both entities' primary keys. For Student M:N Course, create Enrollment(StudentID, CourseID) with both as foreign keys and the composite as primary key. This table can also hold relationship attributes like Grade."

**Q: What's the difference between total and partial participation?**

A: "Total participation means every instance of the entity MUST participate in the relationship (every employee must belong to a department—double line in ER). Partial participation means some instances may not participate (a department may have no employees—single line)."

---

*Next: [03 — Normalization](03-Normalization.md)*
