# Getting Started with ProLeap COBOL Parser

This guide will help you get up and running with the ProLeap COBOL Parser quickly.

## Prerequisites

- **Java 17 or higher**
- **Maven 3.6 or higher** (for building from source)
- Basic understanding of Java and COBOL

## Installation

### Maven Dependency

Add the following dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>io.github.uwol</groupId>
    <artifactId>proleap-cobol-parser</artifactId>
    <version>4.0.0</version>
</dependency>
```

### Gradle Dependency

For Gradle projects, add to your `build.gradle`:

```gradle
dependencies {
    implementation 'io.github.uwol:proleap-cobol-parser:4.0.0'
}
```

## Your First COBOL Parse

### Step 1: Create a Simple COBOL File

Create a file named `hello.cbl`:

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. HELLO-WORLD.
       
       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01 WS-MESSAGE PIC X(20) VALUE 'Hello, COBOL World!'.
       
       PROCEDURE DIVISION.
       MAIN-PARAGRAPH.
           DISPLAY WS-MESSAGE.
           STOP RUN.
```

### Step 2: Write Your First Parser

Create a Java class to parse the COBOL file:

```java
import io.proleap.cobol.asg.runner.impl.CobolParserRunnerImpl;
import io.proleap.cobol.asg.runner.CobolParserRunner;
import io.proleap.cobol.asg.metamodel.Program;
import io.proleap.cobol.asg.metamodel.CompilationUnit;
import io.proleap.cobol.asg.metamodel.ProgramUnit;
import io.proleap.cobol.preprocessor.CobolPreprocessor.CobolSourceFormatEnum;

import java.io.File;
import java.io.IOException;

public class FirstCobolParser {
    public static void main(String[] args) {
        try {
            // Create parser instance
            CobolParserRunner runner = new CobolParserRunnerImpl();
            
            // Parse the COBOL file
            File cobolFile = new File("hello.cbl");
            Program program = runner.analyzeFile(cobolFile, CobolSourceFormatEnum.FIXED);
            
            // Access the parsed structure
            CompilationUnit unit = program.getCompilationUnit();
            ProgramUnit programUnit = unit.getProgramUnit();
            
            System.out.println("Successfully parsed COBOL program!");
            System.out.println("Program name: " + unit.getName());
            System.out.println("Number of lines: " + unit.getLines().size());
            
            // Check if divisions exist
            System.out.println("Has Data Division: " + 
                (programUnit.getDataDivision() != null));
            System.out.println("Has Procedure Division: " + 
                (programUnit.getProcedureDivision() != null));
            
        } catch (IOException e) {
            System.err.println("Error parsing COBOL file: " + e.getMessage());
        }
    }
}
```

### Step 3: Run Your Parser

Compile and run your Java program:

```bash
javac -cp "proleap-cobol-parser-4.0.0.jar:." FirstCobolParser.java
java -cp "proleap-cobol-parser-4.0.0.jar:." FirstCobolParser
```

Expected output:
```
Successfully parsed COBOL program!
Program name: HELLO-WORLD
Number of lines: 10
Has Data Division: true
Has Procedure Division: true
```

## Basic Configuration

### Understanding Source Formats

COBOL source code comes in different formats. Choose the right one for your files:

```java
// For traditional mainframe COBOL (80-column format)
CobolSourceFormatEnum.FIXED

// For modern COBOL with variable line lengths
CobolSourceFormatEnum.VARIABLE

// For HP Tandem/NonStop systems
CobolSourceFormatEnum.TANDEM
```

### Setting Up Parameters

For more control over parsing, use `CobolParserParams`:

```java
import io.proleap.cobol.asg.params.impl.CobolParserParamsImpl;
import io.proleap.cobol.asg.params.CobolDialect;

CobolParserParams params = new CobolParserParamsImpl();

// Set the source format
params.setFormat(CobolSourceFormatEnum.FIXED);

// Set the COBOL dialect
params.setDialect(CobolDialect.ANSI85);

// Configure character encoding
params.setCharset(StandardCharsets.UTF_8);

// Parse with custom parameters
Program program = runner.analyzeFile(cobolFile, params);
```

## Working with Program Structure

### Accessing Divisions

```java
ProgramUnit programUnit = program.getCompilationUnit().getProgramUnit();

// Identification Division
IdentificationDivision idDiv = programUnit.getIdentificationDivision();
if (idDiv != null) {
    System.out.println("Program ID: " + idDiv.getProgramId());
}

// Data Division
DataDivision dataDiv = programUnit.getDataDivision();
if (dataDiv != null) {
    WorkingStorageSection wsSection = dataDiv.getWorkingStorageSection();
    if (wsSection != null) {
        List<DataDescriptionEntry> entries = wsSection.getDataDescriptionEntries();
        System.out.println("Working storage variables: " + entries.size());
    }
}

// Procedure Division
ProcedureDivision procDiv = programUnit.getProcedureDivision();
if (procDiv != null) {
    List<Paragraph> paragraphs = procDiv.getParagraphs();
    System.out.println("Paragraphs: " + paragraphs.size());
}
```

### Exploring Data Structures

```java
public static void exploreDataStructures(Program program) {
    DataDivision dataDiv = program.getCompilationUnit()
        .getProgramUnit().getDataDivision();
    
    if (dataDiv == null) return;
    
    WorkingStorageSection wsSection = dataDiv.getWorkingStorageSection();
    if (wsSection != null) {
        System.out.println("Working Storage Variables:");
        
        for (DataDescriptionEntry entry : wsSection.getDataDescriptionEntries()) {
            System.out.printf("  %02d %s", 
                entry.getLevelNumber(), entry.getName());
            
            if (entry.getPictureClause() != null) {
                System.out.printf(" PIC %s", 
                    entry.getPictureClause().getPictureString());
            }
            
            System.out.println();
        }
    }
}
```

## Error Handling

### Basic Error Handling

Always wrap parsing operations in try-catch blocks:

```java
try {
    Program program = runner.analyzeFile(cobolFile, format);
    // Process the parsed program
    
} catch (CobolParserException e) {
    // COBOL syntax or parsing errors
    System.err.println("Parse error: " + e.getMessage());
    
} catch (IOException e) {
    // File I/O errors
    System.err.println("File error: " + e.getMessage());
    
} catch (Exception e) {
    // Any other unexpected errors
    System.err.println("Unexpected error: " + e.getMessage());
    e.printStackTrace();
}
```

### Tolerant Parsing

For files with syntax errors, enable tolerant parsing:

```java
CobolParserParams params = new CobolParserParamsImpl();
params.setFormat(CobolSourceFormatEnum.FIXED);
params.setIgnoreSyntaxErrors(true); // Continue despite errors

try {
    Program program = runner.analyzeFile(cobolFile, params);
    
    // Check if parsing was successful
    if (program.getCompilationUnit() != null) {
        System.out.println("Partial parsing successful");
        // Process what was successfully parsed
    } else {
        System.out.println("Parsing failed completely");
    }
    
} catch (IOException e) {
    System.err.println("File error: " + e.getMessage());
}
```

## Common Patterns

### Processing Multiple Files

```java
import java.nio.file.*;
import java.util.stream.Stream;

public void processCobolDirectory(String directoryPath) {
    Path dir = Paths.get(directoryPath);
    CobolParserRunner runner = new CobolParserRunnerImpl();
    
    try (Stream<Path> paths = Files.walk(dir)) {
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
                     System.err.println("Error processing " + path + 
                         ": " + e.getMessage());
                 }
             });
    } catch (IOException e) {
        System.err.println("Error walking directory: " + e.getMessage());
    }
}
```

### Working with COPY Books

```java
import java.util.Arrays;

CobolParserParams params = new CobolParserParamsImpl();

// Set directories to search for COPY books
params.setCopyBookDirectories(Arrays.asList(
    new File("src/copybooks"),
    new File("lib/includes")
));

// Set file extensions for COPY books
params.setCopyBookExtensions(Arrays.asList("cpy", "inc", "copy"));

// Parse with COPY book support
Program program = runner.analyzeFile(cobolFile, params);
```

### Extracting Program Information

```java
public class ProgramInfo {
    public static void printProgramSummary(Program program) {
        CompilationUnit unit = program.getCompilationUnit();
        
        System.out.println("=== Program Summary ===");
        System.out.println("Name: " + unit.getName());
        System.out.println("Lines: " + unit.getLines().size());
        
        ProgramUnit programUnit = unit.getProgramUnit();
        
        // Data Division info
        DataDivision dataDiv = programUnit.getDataDivision();
        if (dataDiv != null) {
            int dataItems = countDataItems(dataDiv);
            System.out.println("Data items: " + dataItems);
        }
        
        // Procedure Division info
        ProcedureDivision procDiv = programUnit.getProcedureDivision();
        if (procDiv != null) {
            System.out.println("Paragraphs: " + procDiv.getParagraphs().size());
            System.out.println("Statements: " + procDiv.getStatements().size());
        }
    }
    
    private static int countDataItems(DataDivision dataDiv) {
        int count = 0;
        
        WorkingStorageSection ws = dataDiv.getWorkingStorageSection();
        if (ws != null) {
            count += ws.getDataDescriptionEntries().size();
        }
        
        FileSection fs = dataDiv.getFileSection();
        if (fs != null) {
            count += fs.getFileDescriptionEntries().size();
        }
        
        LinkageSection ls = dataDiv.getLinkageSection();
        if (ls != null) {
            count += ls.getDataDescriptionEntries().size();
        }
        
        return count;
    }
}
```

## Next Steps

Now that you have the basics working, explore these advanced topics:

1. **[API Documentation](API_DOCUMENTATION.md)** - Complete API reference
2. **[Examples](EXAMPLES.md)** - Comprehensive usage examples
3. **[Quick Reference](QUICK_REFERENCE.md)** - Common patterns and utilities

### Explore Advanced Features

- **Custom Visitors**: Create specialized analysis visitors
- **ASG Navigation**: Walk the Abstract Syntax Graph
- **Performance Optimization**: Handle large codebases efficiently
- **Error Recovery**: Build robust parsing pipelines

### Build Your First Application

Consider building one of these common applications:

- **Code Analyzer**: Extract metrics from COBOL programs
- **Documentation Generator**: Auto-generate program documentation
- **Migration Tool**: Convert between COBOL dialects
- **Dependency Analyzer**: Find relationships between programs

## Getting Help

- **Documentation**: Check the comprehensive API documentation
- **Examples**: Look at the examples for common patterns
- **Source Code**: The parser is open source - examine the implementation
- **Community**: Engage with other users and contributors

## Tips for Success

1. **Start Simple**: Begin with basic file parsing before advanced features
2. **Handle Errors**: Always implement proper error handling
3. **Test Different Formats**: Try different source formats if parsing fails
4. **Use Tolerant Parsing**: Enable error tolerance for problematic files
5. **Monitor Memory**: Be mindful of memory usage with large files

Happy COBOL parsing!