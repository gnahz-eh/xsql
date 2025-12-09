# XSQL

[![Build Status](https://github.com/gnahZ-eH/equal/workflows/Java%20CI/badge.svg)](https://github.com/gnahZ-eH/equal/actions)
[![codecov](https://codecov.io/gh/gnahZ-eH/equal/branch/master/graph/badge.svg)](https://codecov.io/gh/gnahZ-eH/equal)

**E<font color='green' size='5'>X</font>cel operations with <font color='green' size='5'>S</font>QL-style <font color='green' size='5'>Q</font>uery <font color='green' size='5'>L</font>anguage**

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Detailed Usage](#detailed-usage)
  - [Setting up Bean Classes](#setting-up-bean-classes)
  - [Reading Files (SELECT)](#reading-files-select)
  - [Writing Files (INSERT)](#writing-files-insert)
  - [Updating Files (UPDATE)](#updating-files-update)
  - [Deleting Records (DELETE)](#deleting-records-delete)
- [Supported File Types](#supported-file-types)
- [Advanced Features](#advanced-features)
- [API Reference](#api-reference)
- [Examples](#examples)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

XSQL is a powerful Java library that provides SQL-style operations for Excel and CSV files. It allows you to perform CRUD operations (Create, Read, Update, Delete) on structured data files using intuitive, fluent API similar to SQL syntax. The library eliminates the complexity of working directly with Excel/CSV APIs by providing a higher-level abstraction.

### Why XSQL?

- **SQL-like Syntax**: Familiar SELECT, INSERT, UPDATE, DELETE operations
- **Type Safety**: Strongly typed operations with Java POJOs
- **Multiple Formats**: Supports XLS, XLSX, and CSV files
- **Annotation-driven**: Simple configuration using annotations
- **Fluent API**: Chain operations for better readability
- **Stream Support**: Modern Java 8+ stream integration

## ✨ Features

- 📊 **Multiple File Format Support**: XLS, XLSX, CSV
- 🔍 **SQL-style Queries**: SELECT, INSERT, UPDATE, DELETE operations
- 🏷️ **Annotation-based Mapping**: Map Java fields to file columns
- 🎯 **Type Adapters**: Built-in converters for common data types
- 📅 **Date/Time Support**: LocalDate, LocalDateTime handling
- 🔢 **Numeric Types**: Support for all Java numeric types
- 📝 **Custom Adapters**: Extensible type conversion system
- 🗂️ **Multi-sheet Support**: Work with specific sheets by index or name
- 🎨 **Flexible Ranges**: Read/write specific row ranges
- 🔄 **Stream Processing**: Java 8+ streams for filtering and processing
- 🛡️ **Exception Handling**: Comprehensive error reporting
- 📊 **Character Encoding**: UTF-8 and custom charset support

## 🔧 Requirements

- **Java**: 8 or higher
- **Maven**: 3.6+ (for building from source)
- **Dependencies**: Apache POI 4.0.1, SLF4J 1.7.25

## 📦 Installation

### Maven

Add the following dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>com.github.zhanghe</groupId>
    <artifactId>XSQL</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

### Gradle

Add to your `build.gradle`:

```gradle
implementation 'com.github.zhanghe:XSQL:1.0-SNAPSHOT'
```

### Build from Source

```bash
git clone https://github.com/gnahZ-eH/equal.git
cd equal
mvn clean install
```

## 🚀 Quick Start

### 1. Define Your Bean Class

```java
import com.github.equal.annotation.Column;
import java.time.LocalDate;

public class Student {
    @Column(name = "name", index = 0)
    private String name;

    @Column(name = "age", index = 1)
    private Integer age;

    @Column(name = "grade", index = 2)
    private String grade;

    @Column(name = "birthDate", index = 3)
    private LocalDate birthDate;

    // Default constructor is required
    public Student() {}
    
    // Getters and setters...
}
```

### 2. Read Data (SELECT)

```java
import com.github.equal.processor.selector.Selector;

// Read all students
List<Student> students = Selector
    .select(Student.class)
    .from(new File("students.xlsx"))
    .executeQuery();

// Read with filtering
List<Student> filteredStudents = students.stream()
    .filter(s -> s.getAge() > 18)
    .collect(Collectors.toList());
```

### 3. Write Data (INSERT)

```java
import com.github.equal.processor.inserter.Inserter;

List<Student> newStudents = Arrays.asList(
    new Student("John", 20, "A", LocalDate.of(2003, 5, 15)),
    new Student("Jane", 19, "B", LocalDate.of(2004, 8, 22))
);

Inserter.insert(Student.class)
    .into(new File("new_students.xlsx"))
    .values(newStudents)
    .flush();
```

## 📚 Detailed Usage

### Setting up Bean Classes

Bean classes represent the structure of your Excel/CSV files. Each field should be annotated with `@Column`:

```java
public class Employee {
    @Column(name = "employee_id", index = 0)
    private Long id;

    @Column(name = "full_name", index = 1)
    private String name;

    @Column(name = "department", index = 2)
    private String department;

    @Column(name = "hire_date", index = 3, dateTimePattern = "yyyy-MM-dd")
    private LocalDate hireDate;

    @Column(name = "salary", index = 4, adapter = BigDecimalAdapter.class)
    private BigDecimal salary;

    @Column(name = "is_active", index = 5)
    private Boolean active;

    // Mandatory default constructor
    public Employee() {}
    
    // Constructor, getters, and setters...
}
```

#### Column Annotation Properties

- `name`: Column name (used for headers)
- `index`: Column position (0-based)
- `adapter`: Custom type adapter class
- `dateTimePattern`: Date/time format pattern

### Reading Files (SELECT)

#### Basic Reading

```java
// Read entire file
List<Student> allStudents = Selector
    .select(Student.class)
    .from(new File("students.xlsx"))
    .executeQuery();

// Read from InputStream
List<Student> students = Selector
    .select(Student.class)
    .from(inputStream)
    .executeQuery();
```

#### Reading Specific Ranges

```java
// Read from row 2, get 10 rows
List<Student> students = Selector
    .select(Student.class)
    .from(new File("students.xlsx"))
    .where(2, 10)
    .executeQuery();

// Read from row 5 to end
List<Student> students = Selector
    .select(Student.class)
    .from(new File("students.xlsx"))
    .where(5)
    .executeQuery();
```

#### Multi-sheet Support

```java
// Read specific sheet by index
List<Student> students = Selector
    .select(Student.class)
    .from(new File("workbook.xlsx"), 1)
    .executeQuery();

// Read specific sheet by name
List<Student> students = Selector
    .select(Student.class)
    .from(new File("workbook.xlsx"), "Student_Data")
    .executeQuery();
```

#### Stream Processing

```java
// Get stream for processing
Stream<Student> studentStream = Selector
    .select(Student.class)
    .from(new File("students.xlsx"))
    .executeStream();

// Process with filters
List<Student> adults = studentStream
    .filter(s -> s.getAge() >= 18)
    .filter(s -> "A".equals(s.getGrade()))
    .collect(Collectors.toList());
```

#### Custom Charset

```java
List<Student> students = Selector
    .select(Student.class)
    .from(new File("students_utf16.csv"))
    .charset(StandardCharsets.UTF_16)
    .executeQuery();
```

### Writing Files (INSERT)

#### Basic Writing

```java
List<Student> students = Arrays.asList(
    new Student("Alice", 22, "A"),
    new Student("Bob", 21, "B")
);

Inserter.insert(Student.class)
    .into(new File("output.xlsx"))
    .values(students)
    .flush();
```

#### Writing to Specific Ranges

```java
// Start writing from row 5
Inserter.insert(Student.class)
    .into(new File("output.xlsx"))
    .values(students)
    .range(5)
    .flush();

// Write 3 rows starting from row 2
Inserter.insert(Student.class)
    .into(new File("output.xlsx"))
    .values(students)
    .range(2, 3)
    .flush();
```

#### Multi-sheet Writing

```java
// Write to specific sheet
Inserter.insert(Student.class)
    .into(new File("workbook.xlsx"))
    .table(1)  // Sheet index
    .values(students)
    .flush();

// Write to named sheet
Inserter.insert(Student.class)
    .into(new File("workbook.xlsx"))
    .table("NewStudents")  // Sheet name
    .values(students)
    .flush();
```

#### Header Behavior

| Scenario | Header Behavior |
|----------|----------------|
| File exists with data | Headers are NOT added |
| New file | Headers are automatically added |
| Empty existing file | Headers are added |

### Updating Files (UPDATE)

```java
// Read, modify, and write back
List<Student> students = Selector
    .select(Student.class)
    .from(new File("students.xlsx"))
    .executeQuery();

// Update logic
students.forEach(s -> {
    if (s.getAge() < 18) {
        s.setGrade("Junior");
    }
});

// Write back
Inserter.insert(Student.class)
    .into(new File("students_updated.xlsx"))
    .values(students)
    .flush();
```

### Deleting Records (DELETE)

#### Delete by Index Range

```java
// Delete rows 3-5
Deleter.delete(Student.class)
    .from(new File("students.xlsx"))
    .where(3, 3)  // start index, count
    .flush();
```

#### Delete by Row Numbers

```java
// Delete specific rows
Set<Integer> rowsToDelete = Set.of(2, 5, 8);
Deleter.delete(Student.class)
    .from(new File("students.xlsx"))
    .where(rowsToDelete)
    .flush();
```

#### Delete from Specific Sheet

```java
// Delete from named sheet
Deleter.delete(Student.class)
    .from(new File("workbook.xlsx"))
    .table("OldStudents")
    .where(1, 5)
    .flush();
```

## 📄 Supported File Types

| Format | Extension | Read | Write | Multi-sheet |
|--------|-----------|------|-------|-------------|
| Excel 97-2003 | `.xls` | ✅ | ✅ | ✅ |
| Excel 2007+ | `.xlsx` | ✅ | ✅ | ✅ |
| CSV | `.csv` | ✅ | ✅ | ❌ |

## 🔧 Advanced Features

### Custom Type Adapters

Create custom adapters for complex data types:

```java
public class CustomDateAdapter implements Adapter<String, LocalDate> {
    @Override
    public LocalDate adapt(String value) {
        return LocalDate.parse(value, DateTimeFormatter.ofPattern("dd/MM/yyyy"));
    }
}

// Use in bean
public class Event {
    @Column(name = "event_date", index = 2, adapter = CustomDateAdapter.class)
    private LocalDate eventDate;
}
```

### Built-in Adapters

- `StringAdapter`: String conversion
- `IntegerAdapter`: Integer conversion
- `LongAdapter`: Long conversion
- `DoubleAdapter`: Double conversion
- `FloatAdapter`: Float conversion
- `BigDecimalAdapter`: BigDecimal conversion
- `BooleanAdapter`: Boolean conversion
- `DateAdapter`: Date conversion
- `DateTimeAdapter`: DateTime conversion
- `TimeAdapter`: Time conversion

### Exception Handling

```java
try {
    List<Student> students = Selector
        .select(Student.class)
        .from(new File("students.xlsx"))
        .executeQuery();
} catch (SelectorException e) {
    logger.error("Error reading file: {}", e.getMessage());
} catch (EqualException e) {
    logger.error("General error: {}", e.getMessage());
}
```

## 📖 API Reference

### Selector Methods

| Method | Description |
|--------|-------------|
| `select(Class<T>)` | Start query for class type |
| `from(File)` | Source file |
| `from(File, int)` | Source file with sheet index |
| `from(File, String)` | Source file with sheet name |
| `from(InputStream)` | Source from stream |
| `where()` | Read all rows |
| `where(int)` | Read from start index |
| `where(int, int)` | Read range |
| `charset(Charset)` | Set character encoding |
| `executeQuery()` | Execute and return List |
| `executeStream()` | Execute and return Stream |

### Inserter Methods

| Method | Description |
|--------|-------------|
| `insert(Class<T>)` | Start insert for class type |
| `into(File)` | Target file |
| `table(int)` | Target sheet index |
| `table(String)` | Target sheet name |
| `values(Collection)` | Data to insert |
| `range()` | Write to default position |
| `range(int)` | Write from start index |
| `range(int, int)` | Write range |
| `charset(Charset)` | Set character encoding |
| `flush()` | Execute write operation |

### Deleter Methods

| Method | Description |
|--------|-------------|
| `delete(Class<T>)` | Start delete for class type |
| `from(File)` | Source file |
| `table(int)` | Target sheet index |
| `table(String)` | Target sheet name |
| `where(int, int)` | Delete range |
| `where(Set<Integer>)` | Delete specific rows |
| `charset(Charset)` | Set character encoding |
| `flush()` | Execute delete operation |

## 💡 Examples

### Complete Student Management Example

```java
import com.github.equal.annotation.Column;
import com.github.equal.processor.selector.Selector;
import com.github.equal.processor.inserter.Inserter;
import com.github.equal.processor.deleter.Deleter;

import java.io.File;
import java.time.LocalDate;
import java.util.List;
import java.util.stream.Collectors;

public class StudentManager {
    
    public static class Student {
        @Column(name = "name", index = 0)
        private String name;
        
        @Column(name = "age", index = 1)
        private Integer age;
        
        @Column(name = "grade", index = 2)
        private String grade;
        
        @Column(name = "enrollment_date", index = 3)
        private LocalDate enrollmentDate;
        
        public Student() {}
        
        public Student(String name, Integer age, String grade, LocalDate enrollmentDate) {
            this.name = name;
            this.age = age;
            this.grade = grade;
            this.enrollmentDate = enrollmentDate;
        }
        
        // Getters and setters...
    }
    
    public static void main(String[] args) throws Exception {
        File studentFile = new File("students.xlsx");
        
        // Read all students
        List<Student> students = Selector
            .select(Student.class)
            .from(studentFile)
            .executeQuery();
            
        System.out.println("Total students: " + students.size());
        
        // Filter adult students
        List<Student> adults = students.stream()
            .filter(s -> s.getAge() >= 18)
            .collect(Collectors.toList());
            
        // Save adult students to new file
        Inserter.insert(Student.class)
            .into(new File("adult_students.xlsx"))
            .values(adults)
            .flush();
            
        // Add new students
        List<Student> newStudents = List.of(
            new Student("Charlie", 20, "A", LocalDate.now()),
            new Student("Diana", 19, "B", LocalDate.now())
        );
        
        Inserter.insert(Student.class)
            .into(studentFile)
            .values(newStudents)
            .range(students.size() + 2)  // Append after existing data
            .flush();
    }
}
```

### CSV Processing Example

```java
public class CSVProcessor {
    
    public void processLargeCsv() throws Exception {
        // Stream processing for large files
        Stream<Student> studentStream = Selector
            .select(Student.class)
            .from(new File("large_students.csv"))
            .charset(StandardCharsets.UTF_8)
            .executeStream();
            
        // Process in chunks
        List<Student> highPerformers = studentStream
            .filter(s -> "A".equals(s.getGrade()))
            .filter(s -> s.getAge() > 20)
            .limit(100)
            .collect(Collectors.toList());
            
        // Save results
        Inserter.insert(Student.class)
            .into(new File("high_performers.csv"))
            .values(highPerformers)
            .charset(StandardCharsets.UTF_8)
            .flush();
    }
}
```

## 🤝 Contributing

We welcome contributions! Please follow these steps:

1. **Fork** the repository
2. **Create** a feature branch: `git checkout -b feature/amazing-feature`
3. **Commit** your changes: `git commit -m 'Add amazing feature'`
4. **Push** to the branch: `git push origin feature/amazing-feature`
5. **Open** a Pull Request

### Development Setup

```bash
git clone https://github.com/gnahZ-eH/equal.git
cd equal
mvn clean compile
mvn test
```

### Code Style

- Follow Java coding conventions
- Add unit tests for new features
- Update documentation for API changes
- Ensure all tests pass: `mvn test`

## 📄 License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

---

### 📞 Support

- **Issues**: [GitHub Issues](https://github.com/gnahZ-eH/equal/issues)
- **CI/CD**: [GitHub Actions](https://github.com/gnahZ-eH/equal/actions)
- **Code Coverage**: [Codecov](https://codecov.io/gh/gnahZ-eH/equal)

### 📊 Project Stats

- **Language**: Java 8+
- **Build Tool**: Maven
- **Testing**: JUnit 5
- **Code Coverage**: JaCoCo
- **Dependencies**: Apache POI, SLF4J

---