# Example: Schema in 3NF but not in BCNF

## 1. Relation Schema

Consider the relation:

**R(Student, Course, Instructor)**

### Functional Dependencies (FDs)
1. **(Student, Course) → Instructor**  
2. **Instructor → Course**

### Candidate Key
- From FD1: **(Student, Course)** determines Instructor  
- Therefore **(Student, Course)** is a **candidate key**.

---

# 2. Check for Third Normal Form (3NF)

A relation is in **3NF** if for every functional dependency **X → Y**, at least one of the following holds:

1. **X is a superkey**, OR  
2. **Y is a prime attribute** (part of a candidate key)

### Analyze FDs

#### FD1: (Student, Course) → Instructor
- LHS is a **candidate key**
- ✅ Satisfies 3NF

#### FD2: Instructor → Course
- LHS is **not a superkey**
- RHS attribute **Course** is part of candidate key (Student, Course)
- Course is therefore a **prime attribute**

✅ Condition 2 satisfied → **3NF holds**

So **R is in 3NF**.

---

# 3. Check for BCNF

A relation is in **BCNF** if for every functional dependency **X → Y**, **X must be a superkey**.

### Check FD2

**Instructor → Course**

- Instructor is **not a superkey**

❌ **BCNF violation**

Therefore:

**R is NOT in BCNF**

---

# 4. BCNF Decomposition

We decompose using the violating dependency:

**Instructor → Course**

### Step 1: Create Relations

1. **R1(Instructor, Course)**
2. **R2(Student, Instructor)**

### Step 2: Verify BCNF

#### R1(Instructor, Course)
FD: Instructor → Course  
Instructor is the **key**  

✅ BCNF

#### R2(Student, Instructor)
No non-trivial dependencies except the key.

Key = **(Student, Instructor)**

✅ BCNF

So the final BCNF decomposition is:

- **R1(Instructor, Course)**
- **R2(Student, Instructor)**

---

# 5. When Is It Acceptable to Keep a Schema in 3NF Instead of BCNF?

Sometimes decomposing to BCNF introduces problems. In practice, we may keep **3NF** when:

### 1. Dependency Preservation
BCNF decomposition may **lose some functional dependencies**.

This means constraints cannot be enforced without performing **joins**, which can hurt performance and integrity checks.

### 2. Query Efficiency
BCNF may introduce **additional joins**, making queries slower.

### 3. Simpler Design
3NF often provides a **good balance between normalization and practicality**.

### 4. Widely Used in Practice
Most database systems and textbooks treat **3NF as sufficient normalization** for real-world systems.

---

# 6. Summary

| Property | Result |
|--------|--------|
| Relation | R(Student, Course, Instructor) |
| Candidate Key | (Student, Course) |
| 3NF | ✅ Satisfied |
| BCNF | ❌ Violated due to Instructor → Course |
| BCNF Decomposition | R1(Instructor, Course), R2(Student, Instructor) |
| Reason to Keep 3NF | Dependency preservation and fewer joins |