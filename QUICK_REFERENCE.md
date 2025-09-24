# ProLeap COBOL Parser - Quick Reference Guide

## Essential Imports

```java
// Core classes
import io.proleap.cobol.asg.runner.impl.CobolParserRunnerImpl;
import io.proleap.cobol.asg.runner.CobolParserRunner;
import io.proleap.cobol.asg.params.impl.CobolParserParamsImpl;
import io.proleap.cobol.asg.params.CobolParserParams;
import io.proleap.cobol.asg.params.CobolDialect;

// Metamodel
import io.proleap.cobol.asg.metamodel.Program;
import io.proleap.cobol.asg.metamodel.CompilationUnit;
import io.proleap.cobol.asg.metamodel.ProgramUnit;

// Preprocessor
import io.proleap.cobol.preprocessor.CobolPreprocessor.CobolSourceFormatEnum;

// Exceptions
import io.proleap.cobol.asg.exception.CobolParserException;
```

## Quick Start Templates

### Parse a File (Minimal)

```java
CobolParserRunner runner = new CobolParserRunnerImpl();
Program program = runner.analyzeFile(
    new File("program.cbl"), 
    CobolSourceFormatEnum.FIXED
);
```

### Parse with Configuration

```java
CobolParserParams params = new CobolParserParamsImpl();
params.setFormat(CobolSourceFormatEnum.FIXED);
params.setDialect(CobolDialect.ANSI85);

CobolParserRunner runner = new CobolParserRunnerImpl();
Program program = runner.analyzeFile(cobolFile, params);
```

### Parse String Content

```java
String cobolCode = "IDENTIFICATION DIVISION.\nPROGRAM-ID. TEST.";
CobolParserParams params = new CobolParserParamsImpl();
params.setFormat(CobolSourceFormatEnum.VARIABLE);

CobolParserRunner runner = new CobolParserRunnerImpl();
Program program = runner.analyzeCode(cobolCode, "TEST", params);
```

## Configuration Cheat Sheet

### Source Formats

| Format | Description | Use Case |
|--------|-------------|----------|
| `FIXED` | 80-column format with fixed areas | Legacy mainframe COBOL |
| `VARIABLE` | Variable length with optional areas | Modern COBOL |
| `TANDEM` | HP Tandem format | HP NonStop systems |

### Dialects

| Dialect | Description |
|---------|-------------|
| `ANSI85` | ANSI-85 standard |
| `MF` | Micro Focus extensions |
| `OSVS` | IBM OS/VS compatibility |

### Common Configuration Patterns

```java
// Strict ANSI-85
CobolParserParams strict = new CobolParserParamsImpl();
strict.setFormat(CobolSourceFormatEnum.FIXED);
strict.setDialect(CobolDialect.ANSI85);
strict.setIgnoreSyntaxErrors(false);

// Tolerant parsing
CobolParserParams tolerant = new CobolParserParamsImpl();
tolerant.setFormat(CobolSourceFormatEnum.VARIABLE);
tolerant.setIgnoreSyntaxErrors(true);

// With COPY books
CobolParserParams withCopyBooks = new CobolParserParamsImpl();
withCopyBooks.setCopyBookDirectories(Arrays.asList(new File("copybooks")));
withCopyBooks.setCopyBookExtensions(Arrays.asList("cpy", "inc"));
```

## Common Access Patterns

### Program Structure Access

```java
Program program = runner.analyzeFile(file, format);
CompilationUnit unit = program.getCompilationUnit();
ProgramUnit programUnit = unit.getProgramUnit();

// Divisions
IdentificationDivision idDiv = programUnit.getIdentificationDivision();
EnvironmentDivision envDiv = programUnit.getEnvironmentDivision();
DataDivision dataDiv = programUnit.getDataDivision();
ProcedureDivision procDiv = programUnit.getProcedureDivision();
```

### Data Division Access

```java
DataDivision dataDiv = programUnit.getDataDivision();

// Working Storage
WorkingStorageSection wsSection = dataDiv.getWorkingStorageSection();
List<DataDescriptionEntry> wsEntries = wsSection.getDataDescriptionEntries();

// File Section
FileSection fileSection = dataDiv.getFileSection();
List<FileDescriptionEntry> fileEntries = fileSection.getFileDescriptionEntries();

// Linkage Section
LinkageSection linkageSection = dataDiv.getLinkageSection();
List<DataDescriptionEntry> linkageEntries = linkageSection.getDataDescriptionEntries();
```

### Procedure Division Access

```java
ProcedureDivision procDiv = programUnit.getProcedureDivision();

// Paragraphs
List<Paragraph> paragraphs = procDiv.getParagraphs();

// Statements
List<Statement> statements = procDiv.getStatements();

// Sections
List<Section> sections = procDiv.getSections();
```

## Error Handling Patterns

### Basic Try-Catch

```java
try {
    Program program = runner.analyzeFile(file, format);
    // Process program
} catch (CobolParserException e) {
    System.err.println("Parse error: " + e.getMessage());
} catch (IOException e) {
    System.err.println("I/O error: " + e.getMessage());
}
```

### Tolerant Parsing

```java
CobolParserParams params = new CobolParserParamsImpl();
params.setIgnoreSyntaxErrors(true);

try {
    Program program = runner.analyzeFile(file, params);
    if (program.getCompilationUnit() != null) {
        // Partial success
    }
} catch (IOException e) {
    // Handle I/O errors
}
```

## Common Utility Methods

### File Type Detection

```java
public static CobolSourceFormatEnum detectFormat(File file) {
    String name = file.getName().toLowerCase();
    if (name.endsWith(".cob") || name.endsWith(".cbl")) {
        return CobolSourceFormatEnum.FIXED;
    } else if (name.endsWith(".eco")) {
        return CobolSourceFormatEnum.VARIABLE;
    }
    return CobolSourceFormatEnum.FIXED; // Default
}
```

### Null-Safe Access

```java
public static boolean hasDataDivision(Program program) {
    CompilationUnit unit = program.getCompilationUnit();
    if (unit == null) return false;
    
    ProgramUnit programUnit = unit.getProgramUnit();
    if (programUnit == null) return false;
    
    return programUnit.getDataDivision() != null;
}
```

### Data Item Counter

```java
public static int countDataItems(DataDivision dataDiv) {
    int count = 0;
    
    WorkingStorageSection ws = dataDiv.getWorkingStorageSection();
    if (ws != null) {
        count += ws.getDataDescriptionEntries().size();
    }
    
    LinkageSection linkage = dataDiv.getLinkageSection();
    if (linkage != null) {
        count += linkage.getDataDescriptionEntries().size();
    }
    
    return count;
}
```

## Performance Tips

### Memory Management

```java
// Process files individually to avoid memory accumulation
for (File file : cobolFiles) {
    Program program = runner.analyzeFile(file, params);
    processProgram(program);
    program = null; // Release reference
    System.gc(); // Suggest GC for large files
}
```

### Parallel Processing

```java
cobolFiles.parallelStream().forEach(file -> {
    try {
        CobolParserRunner runner = new CobolParserRunnerImpl();
        Program program = runner.analyzeFile(file, format);
        processProgram(program);
    } catch (IOException e) {
        System.err.println("Error: " + e.getMessage());
    }
});
```

### Batch Processing

```java
CobolParserParams params = new CobolParserParamsImpl();
params.setIgnoreSyntaxErrors(true); // Faster parsing
params.setFormat(CobolSourceFormatEnum.FIXED);

CobolParserRunner runner = new CobolParserRunnerImpl();

for (File file : largeFileList) {
    try {
        Program program = runner.analyzeFile(file, params);
        // Minimal processing
        String name = program.getCompilationUnit().getName();
        System.out.println("Processed: " + name);
    } catch (Exception e) {
        System.err.println("Skipped " + file.getName() + ": " + e.getMessage());
    }
}
```

## Debugging and Inspection

### Print Program Structure

```java
public static void printProgramStructure(Program program) {
    CompilationUnit unit = program.getCompilationUnit();
    if (unit == null) {
        System.out.println("No compilation unit found");
        return;
    }
    
    System.out.println("Program: " + unit.getName());
    System.out.println("Lines: " + unit.getLines().size());
    
    ProgramUnit programUnit = unit.getProgramUnit();
    if (programUnit != null) {
        System.out.println("Has ID Division: " + (programUnit.getIdentificationDivision() != null));
        System.out.println("Has Environment Division: " + (programUnit.getEnvironmentDivision() != null));
        System.out.println("Has Data Division: " + (programUnit.getDataDivision() != null));
        System.out.println("Has Procedure Division: " + (programUnit.getProcedureDivision() != null));
    }
}
```

### ASG Tree Walker

```java
public static void walkASG(ASGElement element, int depth) {
    String indent = "  ".repeat(depth);
    System.out.println(indent + element.getClass().getSimpleName());
    
    for (ASGElement child : element.getChildren()) {
        walkASG(child, depth + 1);
    }
}
```

## Common Patterns by Use Case

### Static Analysis Tool

```java
public class StaticAnalyzer {
    public void analyzeFile(File cobolFile) {
        try {
            CobolParserParams params = new CobolParserParamsImpl();
            params.setFormat(CobolSourceFormatEnum.FIXED);
            params.setIgnoreSyntaxErrors(true);
            
            CobolParserRunner runner = new CobolParserRunnerImpl();
            Program program = runner.analyzeFile(cobolFile, params);
            
            // Extract metrics
            ProgramMetrics metrics = calculateMetrics(program);
            generateReport(cobolFile.getName(), metrics);
            
        } catch (IOException e) {
            System.err.println("Analysis failed for " + cobolFile.getName());
        }
    }
}
```

### Code Migration Tool

```java
public class MigrationTool {
    public void migrateProgram(File sourceFile, File targetFile) {
        try {
            // Parse source
            CobolParserRunner runner = new CobolParserRunnerImpl();
            Program program = runner.analyzeFile(sourceFile, CobolSourceFormatEnum.FIXED);
            
            // Transform ASG
            Program transformedProgram = transformForModernCOBOL(program);
            
            // Generate target code
            generateModernCOBOL(transformedProgram, targetFile);
            
        } catch (IOException e) {
            System.err.println("Migration failed: " + e.getMessage());
        }
    }
}
```

### Documentation Generator

```java
public class DocumentationGenerator {
    public void generateDocs(List<File> cobolFiles, File outputDir) {
        CobolParserRunner runner = new CobolParserRunnerImpl();
        
        for (File file : cobolFiles) {
            try {
                Program program = runner.analyzeFile(file, CobolSourceFormatEnum.FIXED);
                generateProgramDoc(program, outputDir);
            } catch (IOException e) {
                System.err.println("Doc generation failed for " + file.getName());
            }
        }
    }
}
```

## Troubleshooting

### Common Issues and Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| `CobolParserException` | Syntax errors | Use `setIgnoreSyntaxErrors(true)` |
| `FileNotFoundException` | Wrong path | Check file exists and permissions |
| `OutOfMemoryError` | Large files | Process files individually, call `System.gc()` |
| Wrong line endings | Format mismatch | Try different `CobolSourceFormatEnum` values |
| COPY book errors | Missing includes | Set `copyBookDirectories` and `copyBookExtensions` |

### Debug Configuration

```java
// Enable detailed logging
CobolParserParams debugParams = new CobolParserParamsImpl();
debugParams.setFormat(CobolSourceFormatEnum.VARIABLE);
debugParams.setIgnoreSyntaxErrors(true);
debugParams.setCharset(StandardCharsets.UTF_8);

// Try multiple formats if one fails
CobolSourceFormatEnum[] formats = {
    CobolSourceFormatEnum.FIXED,
    CobolSourceFormatEnum.VARIABLE,
    CobolSourceFormatEnum.TANDEM
};

for (CobolSourceFormatEnum format : formats) {
    try {
        debugParams.setFormat(format);
        Program program = runner.analyzeFile(file, debugParams);
        System.out.println("Success with format: " + format);
        break;
    } catch (Exception e) {
        System.out.println("Failed with format " + format + ": " + e.getMessage());
    }
}
```

---

This quick reference provides the essential patterns and configurations needed for most common use cases with the ProLeap COBOL Parser.