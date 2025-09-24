# ProLeap COBOL Parser API Documentation

**Version:** 4.0.0  
**Description:** An ANTLR4-based parser and grammar for COBOL  
**License:** MIT License

## Table of Contents

1. [Overview](#overview)
2. [Getting Started](#getting-started)
3. [Core APIs](#core-apis)
4. [Configuration](#configuration)
5. [Metamodel](#metamodel)
6. [Examples](#examples)
7. [Error Handling](#error-handling)
8. [Advanced Usage](#advanced-usage)

## Overview

The ProLeap COBOL Parser is a comprehensive Java library for parsing and analyzing COBOL source code. It provides a complete Abstract Syntax Graph (ASG) representation of COBOL programs, supporting various COBOL dialects and source formats.

### Key Features

- **ANTLR4-based parsing** for robust syntax analysis
- **Multiple COBOL dialects** support (ANSI85, MF, OSVS)
- **Various source formats** (Fixed, Variable, Tandem)
- **Complete ASG representation** with type-safe metamodel
- **Preprocessing capabilities** for COPY statements and includes
- **Comprehensive error handling** with detailed diagnostics

### Architecture

The parser consists of several key components:

1. **Parser Runner** - Main entry point for parsing operations
2. **Preprocessor** - Handles COBOL source preprocessing
3. **Metamodel** - Type-safe representation of COBOL constructs
4. **Visitors** - Multi-phase analysis pipeline
5. **Configuration** - Flexible parsing parameters

## Getting Started

### Maven Dependency

```xml
<dependency>
    <groupId>io.github.uwol</groupId>
    <artifactId>proleap-cobol-parser</artifactId>
    <version>4.0.0</version>
</dependency>
```

### Basic Usage

```java
import io.proleap.cobol.asg.runner.impl.CobolParserRunnerImpl;
import io.proleap.cobol.asg.metamodel.Program;
import io.proleap.cobol.preprocessor.CobolPreprocessor.CobolSourceFormatEnum;

// Parse a COBOL file
File cobolFile = new File("example.cbl");
CobolParserRunner runner = new CobolParserRunnerImpl();
Program program = runner.analyzeFile(cobolFile, CobolSourceFormatEnum.FIXED);

// Access parsed content
CompilationUnit unit = program.getCompilationUnit();
ProgramUnit programUnit = unit.getProgramUnit();
```

## Core APIs

### CobolParserRunner

The main interface for parsing COBOL source code.

**Package:** `io.proleap.cobol.asg.runner`

#### Interface Definition

```java
public interface CobolParserRunner {
    Program analyzeCode(String cobolCode, String compilationUnitName, CobolParserParams params) throws IOException;
    Program analyzeFile(File cobolFile, CobolParserParams params) throws IOException;
    Program analyzeFile(File cobolFile, CobolSourceFormatEnum format) throws IOException;
}
```

#### Implementation

**Class:** `io.proleap.cobol.asg.runner.impl.CobolParserRunnerImpl`

#### Methods

##### `analyzeCode(String, String, CobolParserParams)`

Parses COBOL source code from a string.

**Parameters:**
- `cobolCode` - The COBOL source code as a string
- `compilationUnitName` - Name for the compilation unit
- `params` - Parser configuration parameters

**Returns:** `Program` - The parsed program representation

**Throws:** `IOException` - If parsing fails

**Example:**
```java
CobolParserRunner runner = new CobolParserRunnerImpl();
CobolParserParams params = new CobolParserParamsImpl();
params.setFormat(CobolSourceFormatEnum.FIXED);

String cobolCode = """
    IDENTIFICATION DIVISION.
    PROGRAM-ID. HELLO.
    DATA DIVISION.
    WORKING-STORAGE SECTION.
    01 MESSAGE PIC X(20) VALUE 'Hello, World!'.
    """;

Program program = runner.analyzeCode(cobolCode, "HELLO", params);
```

##### `analyzeFile(File, CobolParserParams)`

Parses a COBOL file with custom parameters.

**Parameters:**
- `cobolFile` - The COBOL source file
- `params` - Parser configuration parameters

**Returns:** `Program` - The parsed program representation

**Throws:** `IOException` - If file reading or parsing fails

**Example:**
```java
CobolParserRunner runner = new CobolParserRunnerImpl();
CobolParserParams params = new CobolParserParamsImpl();
params.setFormat(CobolSourceFormatEnum.VARIABLE);
params.setDialect(CobolDialect.ANSI85);

File cobolFile = new File("src/main/cobol/program.cbl");
Program program = runner.analyzeFile(cobolFile, params);
```

##### `analyzeFile(File, CobolSourceFormatEnum)`

Parses a COBOL file with a specific source format (uses default parameters).

**Parameters:**
- `cobolFile` - The COBOL source file
- `format` - The source format (FIXED, VARIABLE, or TANDEM)

**Returns:** `Program` - The parsed program representation

**Throws:** `IOException` - If file reading or parsing fails

**Example:**
```java
CobolParserRunner runner = new CobolParserRunnerImpl();
File cobolFile = new File("legacy.cob");
Program program = runner.analyzeFile(cobolFile, CobolSourceFormatEnum.FIXED);
```

### CobolPreprocessor

Handles COBOL source code preprocessing, including COPY statements and format normalization.

**Package:** `io.proleap.cobol.preprocessor`

#### Interface Definition

```java
public interface CobolPreprocessor {
    String process(File cobolFile, CobolParserParams params) throws IOException;
    String process(String cobolCode, CobolParserParams params);
}
```

#### Implementation

**Class:** `io.proleap.cobol.preprocessor.impl.CobolPreprocessorImpl`

#### Source Format Enumeration

**Enum:** `CobolPreprocessor.CobolSourceFormatEnum`

```java
public enum CobolSourceFormatEnum {
    FIXED,      // Standard ANSI/IBM format (80 characters per line)
    VARIABLE,   // Variable format with optional areas
    TANDEM      // HP Tandem format
}
```

**FIXED Format:**
- Lines 1-6: Sequence area
- Line 7: Indicator field
- Lines 8-12: Area A
- Lines 13-72: Area B
- Lines 73-80: Comments

**VARIABLE Format:**
- Lines 1-6: Sequence area
- Line 7: Indicator field
- Lines 8-12: Optional area A
- Lines 13+: Optional area B

**TANDEM Format:**
- Line 1: Indicator field
- Lines 2-5: Optional area A
- Lines 6-132: Optional area B

#### Methods

##### `process(File, CobolParserParams)`

Preprocesses a COBOL file.

**Parameters:**
- `cobolFile` - The source file to preprocess
- `params` - Preprocessing parameters

**Returns:** `String` - Preprocessed COBOL source code

**Example:**
```java
CobolPreprocessor preprocessor = new CobolPreprocessorImpl();
CobolParserParams params = new CobolParserParamsImpl();
params.setFormat(CobolSourceFormatEnum.FIXED);

File sourceFile = new File("program.cbl");
String preprocessedCode = preprocessor.process(sourceFile, params);
```

##### `process(String, CobolParserParams)`

Preprocesses COBOL source code from a string.

**Parameters:**
- `cobolCode` - The source code to preprocess
- `params` - Preprocessing parameters

**Returns:** `String` - Preprocessed COBOL source code

**Example:**
```java
CobolPreprocessor preprocessor = new CobolPreprocessorImpl();
CobolParserParams params = new CobolParserParamsImpl();

String rawCode = """
         COPY CUSTOMER.
         01 WS-COUNTER PIC 9(3).
    """;

String preprocessedCode = preprocessor.process(rawCode, params);
```

## Configuration

### CobolParserParams

Configuration interface for parser parameters.

**Package:** `io.proleap.cobol.asg.params`

#### Interface Definition

```java
public interface CobolParserParams {
    // Getters
    Charset getCharset();
    List<File> getCopyBookDirectories();
    List<String> getCopyBookExtensions();
    List<File> getCopyBookFiles();
    CobolDialect getDialect();
    CobolSourceFormatEnum getFormat();
    boolean getIgnoreSyntaxErrors();
    
    // Setters
    void setCharset(Charset charset);
    void setCopyBookDirectories(List<File> copyBookDirectories);
    void setCopyBookExtensions(List<String> copyBookExtensions);
    void setCopyBookFiles(List<File> copyBookFiles);
    void setDialect(CobolDialect dialect);
    void setFormat(CobolSourceFormatEnum format);
    void setIgnoreSyntaxErrors(boolean ignoreSyntaxErrors);
}
```

#### Implementation

**Class:** `io.proleap.cobol.asg.params.impl.CobolParserParamsImpl`

#### Configuration Properties

##### `charset`

**Type:** `java.nio.charset.Charset`  
**Default:** `StandardCharsets.UTF_8`  
**Description:** Character encoding for reading source files

**Example:**
```java
CobolParserParams params = new CobolParserParamsImpl();
params.setCharset(StandardCharsets.ISO_8859_1);
```

##### `copyBookDirectories`

**Type:** `List<File>`  
**Default:** `null`  
**Description:** Directories to search for COPY books

**Example:**
```java
List<File> copyDirs = Arrays.asList(
    new File("src/copy"),
    new File("lib/copybooks")
);
params.setCopyBookDirectories(copyDirs);
```

##### `copyBookExtensions`

**Type:** `List<String>`  
**Default:** `null`  
**Description:** File extensions for COPY books

**Example:**
```java
List<String> extensions = Arrays.asList("cpy", "inc", "copy");
params.setCopyBookExtensions(extensions);
```

##### `copyBookFiles`

**Type:** `List<File>`  
**Default:** `null`  
**Description:** Specific COPY book files to include

**Example:**
```java
List<File> copyFiles = Arrays.asList(
    new File("common/CUSTOMER.cpy"),
    new File("common/CONSTANTS.inc")
);
params.setCopyBookFiles(copyFiles);
```

##### `dialect`

**Type:** `CobolDialect`  
**Default:** `null`  
**Description:** COBOL dialect for parsing

**Example:**
```java
params.setDialect(CobolDialect.ANSI85);
```

##### `format`

**Type:** `CobolSourceFormatEnum`  
**Default:** `null`  
**Description:** Source code format

**Example:**
```java
params.setFormat(CobolSourceFormatEnum.FIXED);
```

##### `ignoreSyntaxErrors`

**Type:** `boolean`  
**Default:** `false`  
**Description:** Whether to continue parsing despite syntax errors

**Example:**
```java
params.setIgnoreSyntaxErrors(true); // Continue parsing with errors
```

### CobolDialect

Enumeration of supported COBOL dialects.

**Package:** `io.proleap.cobol.asg.params`

#### Values

```java
public enum CobolDialect {
    ANSI85,  // ANSI-85 COBOL standard
    MF,      // Micro Focus COBOL
    OSVS     // IBM OS/VS COBOL
}
```

#### Usage Examples

```java
// ANSI-85 standard
params.setDialect(CobolDialect.ANSI85);

// Micro Focus extensions
params.setDialect(CobolDialect.MF);

// IBM OS/VS compatibility
params.setDialect(CobolDialect.OSVS);
```

## Metamodel

The metamodel provides a type-safe, object-oriented representation of COBOL program structure.

### Core Interfaces

#### Program

Root interface representing a complete COBOL program.

**Package:** `io.proleap.cobol.asg.metamodel`

```java
public interface Program extends ASGElement {
    ASGElementRegistry getASGElementRegistry();
    CompilationUnit getCompilationUnit();
    CompilationUnit getCompilationUnit(String name);
    List<CompilationUnit> getCompilationUnits();
    void registerCompilationUnit(CompilationUnit compilationUnit);
}
```

**Methods:**

##### `getCompilationUnit()`
Returns the first compilation unit in the program.

##### `getCompilationUnit(String name)`
Returns a specific compilation unit by name.

##### `getCompilationUnits()`
Returns all compilation units in the program.

**Example:**
```java
Program program = runner.analyzeFile(cobolFile, format);
CompilationUnit mainUnit = program.getCompilationUnit("MAINPROG");
List<CompilationUnit> allUnits = program.getCompilationUnits();
```

#### CompilationUnit

Represents a COBOL compilation unit.

**Package:** `io.proleap.cobol.asg.metamodel`

```java
public interface CompilationUnit extends ASGElement, NamedElement {
    ProgramUnit addProgramUnit(ProgramUnitContext ctx);
    List<String> getLines();
    ProgramUnit getProgramUnit();
    List<ProgramUnit> getProgramUnits();
    CommonTokenStream getTokens();
    int incrementFillerCounter();
    void setLines(List<String> lines);
}
```

**Example:**
```java
CompilationUnit unit = program.getCompilationUnit();
ProgramUnit programUnit = unit.getProgramUnit();
List<String> sourceLines = unit.getLines();
```

#### ProgramUnit

Represents a COBOL program unit with its divisions.

**Package:** `io.proleap.cobol.asg.metamodel`

```java
public interface ProgramUnit extends CompilationUnitElement {
    DataDivision addDataDivision(DataDivisionContext ctx);
    EnvironmentDivision addEnvironmentDivision(EnvironmentDivisionContext ctx);
    IdentificationDivision addIdentificationDivision(IdentificationDivisionContext ctx);
    ProcedureDivision addProcedureDivision(ProcedureDivisionContext ctx);
    
    DataDivision getDataDivision();
    EnvironmentDivision getEnvironmentDivision();
    IdentificationDivision getIdentificationDivision();
    ProcedureDivision getProcedureDivision();
}
```

**Example:**
```java
ProgramUnit programUnit = compilationUnit.getProgramUnit();
IdentificationDivision idDiv = programUnit.getIdentificationDivision();
DataDivision dataDiv = programUnit.getDataDivision();
ProcedureDivision procDiv = programUnit.getProcedureDivision();
```

#### ASGElement

Base interface for all Abstract Syntax Graph elements.

**Package:** `io.proleap.cobol.asg.metamodel`

```java
public interface ASGElement extends ModelElement {
    List<ASGElement> getChildren();
    ParserRuleContext getCtx();
    ASGElement getParent();
    Program getProgram();
}
```

**Common Operations:**
```java
ASGElement element = // ... any ASG element
Program program = element.getProgram();
ASGElement parent = element.getParent();
List<ASGElement> children = element.getChildren();
ParserRuleContext ctx = element.getCtx(); // ANTLR parse tree context
```

### Division Interfaces

#### Data Division

Access to data structures and storage definitions.

```java
DataDivision dataDiv = programUnit.getDataDivision();
WorkingStorageSection wsSection = dataDiv.getWorkingStorageSection();
List<DataDescriptionEntry> entries = wsSection.getDataDescriptionEntries();
```

#### Procedure Division

Access to executable statements and procedures.

```java
ProcedureDivision procDiv = programUnit.getProcedureDivision();
List<Statement> statements = procDiv.getStatements();
List<Paragraph> paragraphs = procDiv.getParagraphs();
```

## Examples

### Basic File Parsing

```java
import io.proleap.cobol.asg.runner.impl.CobolParserRunnerImpl;
import io.proleap.cobol.asg.metamodel.*;
import io.proleap.cobol.preprocessor.CobolPreprocessor.CobolSourceFormatEnum;

public class BasicParsingExample {
    public static void main(String[] args) throws IOException {
        // Initialize parser
        CobolParserRunner runner = new CobolParserRunnerImpl();
        
        // Parse file
        File cobolFile = new File("src/main/cobol/CUSTOMER.cbl");
        Program program = runner.analyzeFile(cobolFile, CobolSourceFormatEnum.FIXED);
        
        // Access program structure
        CompilationUnit unit = program.getCompilationUnit("CUSTOMER");
        ProgramUnit programUnit = unit.getProgramUnit();
        
        System.out.println("Program parsed successfully!");
        System.out.println("Program ID: " + programUnit.getIdentificationDivision().getProgramId());
    }
}
```

### Advanced Configuration

```java
import io.proleap.cobol.asg.params.impl.CobolParserParamsImpl;
import io.proleap.cobol.asg.params.CobolDialect;
import java.nio.charset.StandardCharsets;
import java.util.Arrays;

public class AdvancedConfigExample {
    public static void main(String[] args) throws IOException {
        // Create custom parameters
        CobolParserParams params = new CobolParserParamsImpl();
        
        // Configure charset
        params.setCharset(StandardCharsets.ISO_8859_1);
        
        // Configure COPY book handling
        params.setCopyBookDirectories(Arrays.asList(
            new File("src/copybooks"),
            new File("lib/includes")
        ));
        params.setCopyBookExtensions(Arrays.asList("cpy", "inc"));
        
        // Set dialect and format
        params.setDialect(CobolDialect.MF);
        params.setFormat(CobolSourceFormatEnum.VARIABLE);
        
        // Enable error tolerance
        params.setIgnoreSyntaxErrors(true);
        
        // Parse with custom parameters
        CobolParserRunner runner = new CobolParserRunnerImpl();
        Program program = runner.analyzeFile(new File("program.cbl"), params);
        
        System.out.println("Parsed with custom configuration");
    }
}
```

### String-based Parsing

```java
public class StringParsingExample {
    public static void main(String[] args) throws IOException {
        String cobolSource = """
            IDENTIFICATION DIVISION.
            PROGRAM-ID. HELLO-WORLD.
            
            DATA DIVISION.
            WORKING-STORAGE SECTION.
            01 WS-MESSAGE PIC X(20) VALUE 'Hello, COBOL World!'.
            01 WS-COUNTER PIC 9(3) VALUE 0.
            
            PROCEDURE DIVISION.
            MAIN-PARAGRAPH.
                DISPLAY WS-MESSAGE.
                ADD 1 TO WS-COUNTER.
                STOP RUN.
            """;
        
        // Configure parameters
        CobolParserParams params = new CobolParserParamsImpl();
        params.setFormat(CobolSourceFormatEnum.VARIABLE);
        params.setDialect(CobolDialect.ANSI85);
        
        // Parse string
        CobolParserRunner runner = new CobolParserRunnerImpl();
        Program program = runner.analyzeCode(cobolSource, "HELLO-WORLD", params);
        
        // Access parsed elements
        CompilationUnit unit = program.getCompilationUnit("HELLO-WORLD");
        ProgramUnit programUnit = unit.getProgramUnit();
        DataDivision dataDiv = programUnit.getDataDivision();
        
        System.out.println("String parsed successfully!");
    }
}
```

### Analyzing Data Structures

```java
import io.proleap.cobol.asg.metamodel.data.DataDivision;
import io.proleap.cobol.asg.metamodel.data.workingstorage.WorkingStorageSection;
import io.proleap.cobol.asg.metamodel.data.datadescription.DataDescriptionEntry;

public class DataAnalysisExample {
    public static void analyzeDataStructures(Program program) {
        CompilationUnit unit = program.getCompilationUnit();
        ProgramUnit programUnit = unit.getProgramUnit();
        DataDivision dataDiv = programUnit.getDataDivision();
        
        if (dataDiv != null) {
            WorkingStorageSection wsSection = dataDiv.getWorkingStorageSection();
            if (wsSection != null) {
                List<DataDescriptionEntry> entries = wsSection.getDataDescriptionEntries();
                
                System.out.println("Working Storage Variables:");
                for (DataDescriptionEntry entry : entries) {
                    System.out.printf("- Level %02d: %s%n", 
                        entry.getLevelNumber(), 
                        entry.getName());
                }
            }
        }
    }
}
```

### Multiple File Processing

```java
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.stream.Stream;

public class BatchProcessingExample {
    public static void processCobolDirectory(Path directory) throws IOException {
        CobolParserRunner runner = new CobolParserRunnerImpl();
        
        try (Stream<Path> paths = Files.walk(directory)) {
            paths.filter(path -> path.toString().endsWith(".cbl"))
                 .forEach(path -> {
                     try {
                         System.out.println("Processing: " + path.getFileName());
                         Program program = runner.analyzeFile(
                             path.toFile(), 
                             CobolSourceFormatEnum.FIXED
                         );
                         
                         // Process each program
                         analyzeProgram(program);
                         
                     } catch (IOException e) {
                         System.err.println("Error processing " + path + ": " + e.getMessage());
                     }
                 });
        }
    }
    
    private static void analyzeProgram(Program program) {
        // Custom analysis logic here
        CompilationUnit unit = program.getCompilationUnit();
        if (unit != null) {
            System.out.println("Successfully parsed: " + unit.getName());
        }
    }
}
```

## Error Handling

### CobolParserException

Runtime exception thrown when parsing fails.

**Package:** `io.proleap.cobol.asg.exception`

```java
public class CobolParserException extends RuntimeException {
    public CobolParserException(String message);
}
```

### Error Handling Strategies

#### Strict Error Handling (Default)

```java
CobolParserParams params = new CobolParserParamsImpl();
params.setIgnoreSyntaxErrors(false); // Default: strict mode

try {
    Program program = runner.analyzeFile(cobolFile, params);
    // Process successful parse
} catch (CobolParserException e) {
    System.err.println("Parse error: " + e.getMessage());
    // Handle parse failure
} catch (IOException e) {
    System.err.println("I/O error: " + e.getMessage());
    // Handle file I/O issues
}
```

#### Tolerant Error Handling

```java
CobolParserParams params = new CobolParserParamsImpl();
params.setIgnoreSyntaxErrors(true); // Continue despite errors

try {
    Program program = runner.analyzeFile(cobolFile, params);
    // Program may contain partial information
    // Some elements might be null or incomplete
    
    if (program.getCompilationUnit() != null) {
        // Process what was successfully parsed
    }
} catch (IOException e) {
    System.err.println("I/O error: " + e.getMessage());
}
```

#### Custom Error Listener

```java
import io.proleap.cobol.asg.runner.ThrowingErrorListener;
import org.antlr.v4.runtime.BaseErrorListener;
import org.antlr.v4.runtime.RecognitionException;
import org.antlr.v4.runtime.Recognizer;

public class CustomErrorListener extends BaseErrorListener {
    private List<String> errors = new ArrayList<>();
    
    @Override
    public void syntaxError(Recognizer<?, ?> recognizer, Object offendingSymbol,
                          int line, int charPositionInLine, String msg,
                          RecognitionException e) {
        String error = String.format("Line %d:%d - %s", line, charPositionInLine, msg);
        errors.add(error);
        System.err.println("Syntax error: " + error);
    }
    
    public List<String> getErrors() {
        return new ArrayList<>(errors);
    }
    
    public boolean hasErrors() {
        return !errors.isEmpty();
    }
}
```

### Common Error Scenarios

#### File Not Found

```java
try {
    Program program = runner.analyzeFile(new File("nonexistent.cbl"), format);
} catch (CobolParserException e) {
    if (e.getMessage().contains("Could not find file")) {
        System.err.println("COBOL file does not exist");
    }
} catch (IOException e) {
    System.err.println("File access error: " + e.getMessage());
}
```

#### Syntax Errors

```java
// Enable error tolerance for partially valid files
CobolParserParams params = new CobolParserParamsImpl();
params.setIgnoreSyntaxErrors(true);

Program program = runner.analyzeFile(cobolFile, params);

// Check if parsing was complete
if (program.getCompilationUnit() == null) {
    System.err.println("Warning: No compilation unit found, severe syntax errors");
} else {
    System.out.println("Partial parsing successful");
}
```

## Advanced Usage

### Custom Visitors

The parser uses a multi-phase visitor pattern for analysis. You can create custom visitors for specialized processing.

```java
import io.proleap.cobol.asg.visitor.ParserVisitor;
import org.antlr.v4.runtime.tree.ParseTree;

public class CustomAnalysisVisitor implements ParserVisitor {
    private final Program program;
    
    public CustomAnalysisVisitor(Program program) {
        this.program = program;
    }
    
    @Override
    public Boolean visit(ParseTree parseTree) {
        // Custom analysis logic
        // Return true to continue visiting children
        return true;
    }
}
```

### Preprocessing Only

```java
// Use preprocessor independently
CobolPreprocessor preprocessor = new CobolPreprocessorImpl();
CobolParserParams params = new CobolParserParamsImpl();
params.setFormat(CobolSourceFormatEnum.FIXED);

// Preprocess without parsing
String preprocessedCode = preprocessor.process(cobolFile, params);

// Now you can inspect the preprocessed code
System.out.println("Preprocessed COBOL:");
System.out.println(preprocessedCode);
```

### ASG Navigation

```java
import io.proleap.cobol.asg.metamodel.ASGElement;

public class ASGNavigationExample {
    
    public static void walkASG(ASGElement element, int depth) {
        String indent = "  ".repeat(depth);
        System.out.println(indent + element.getClass().getSimpleName());
        
        // Visit all children
        for (ASGElement child : element.getChildren()) {
            walkASG(child, depth + 1);
        }
    }
    
    public static void analyzeProgram(Program program) {
        System.out.println("ASG Structure:");
        walkASG(program, 0);
    }
}
```

### Performance Considerations

#### Memory Management

```java
// For large files or batch processing
CobolParserParams params = new CobolParserParamsImpl();
params.setIgnoreSyntaxErrors(true); // Faster parsing

// Process files individually to manage memory
for (File cobolFile : cobolFiles) {
    Program program = runner.analyzeFile(cobolFile, params);
    processProgram(program);
    
    // Program can be garbage collected after processing
    program = null;
    System.gc(); // Suggest garbage collection for large batches
}
```

#### Parallel Processing

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ForkJoinPool;

public class ParallelProcessingExample {
    
    public static void processFilesInParallel(List<File> cobolFiles) {
        ForkJoinPool customThreadPool = new ForkJoinPool(4);
        
        try {
            List<CompletableFuture<Program>> futures = cobolFiles.stream()
                .map(file -> CompletableFuture.supplyAsync(() -> {
                    try {
                        CobolParserRunner runner = new CobolParserRunnerImpl();
                        return runner.analyzeFile(file, CobolSourceFormatEnum.FIXED);
                    } catch (IOException e) {
                        throw new RuntimeException(e);
                    }
                }, customThreadPool))
                .collect(Collectors.toList());
            
            // Wait for all to complete
            CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
                           .join();
            
            // Process results
            for (CompletableFuture<Program> future : futures) {
                Program program = future.get();
                analyzeProgram(program);
            }
            
        } catch (Exception e) {
            System.err.println("Parallel processing error: " + e.getMessage());
        } finally {
            customThreadPool.shutdown();
        }
    }
}
```

---

## API Reference Summary

### Main Interfaces

| Interface | Package | Purpose |
|-----------|---------|---------|
| `CobolParserRunner` | `io.proleap.cobol.asg.runner` | Main parsing interface |
| `CobolPreprocessor` | `io.proleap.cobol.preprocessor` | Source preprocessing |
| `CobolParserParams` | `io.proleap.cobol.asg.params` | Configuration parameters |
| `Program` | `io.proleap.cobol.asg.metamodel` | Root program representation |
| `CompilationUnit` | `io.proleap.cobol.asg.metamodel` | Compilation unit representation |
| `ProgramUnit` | `io.proleap.cobol.asg.metamodel` | Program unit with divisions |

### Key Enumerations

| Enum | Values | Purpose |
|------|--------|---------|
| `CobolDialect` | `ANSI85`, `MF`, `OSVS` | COBOL dialect support |
| `CobolSourceFormatEnum` | `FIXED`, `VARIABLE`, `TANDEM` | Source format specification |

### Main Implementation Classes

| Class | Interface | Package |
|-------|-----------|---------|
| `CobolParserRunnerImpl` | `CobolParserRunner` | `io.proleap.cobol.asg.runner.impl` |
| `CobolPreprocessorImpl` | `CobolPreprocessor` | `io.proleap.cobol.preprocessor.impl` |
| `CobolParserParamsImpl` | `CobolParserParams` | `io.proleap.cobol.asg.params.impl` |

### Exception Classes

| Class | Package | Purpose |
|-------|---------|---------|
| `CobolParserException` | `io.proleap.cobol.asg.exception` | Runtime parsing errors |

---

*This documentation covers the core public APIs of the ProLeap COBOL Parser. For additional details about specific metamodel interfaces or advanced customization, refer to the JavaDoc documentation or source code.*