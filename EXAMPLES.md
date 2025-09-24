# ProLeap COBOL Parser - Usage Examples

This document provides comprehensive examples demonstrating how to use the ProLeap COBOL Parser API for various common scenarios.

## Table of Contents

1. [Quick Start Examples](#quick-start-examples)
2. [Configuration Examples](#configuration-examples)
3. [File Processing Examples](#file-processing-examples)
4. [Data Structure Analysis](#data-structure-analysis)
5. [Procedure Analysis](#procedure-analysis)
6. [Error Handling Examples](#error-handling-examples)
7. [Advanced Use Cases](#advanced-use-cases)
8. [Performance Optimization](#performance-optimization)

## Quick Start Examples

### Basic File Parsing

```java
import io.proleap.cobol.asg.runner.impl.CobolParserRunnerImpl;
import io.proleap.cobol.asg.metamodel.Program;
import io.proleap.cobol.asg.metamodel.CompilationUnit;
import io.proleap.cobol.preprocessor.CobolPreprocessor.CobolSourceFormatEnum;

import java.io.File;
import java.io.IOException;

public class BasicParsingExample {
    public static void main(String[] args) {
        try {
            // Create parser instance
            CobolParserRunner runner = new CobolParserRunnerImpl();
            
            // Parse a COBOL file
            File cobolFile = new File("examples/CUSTOMER.cbl");
            Program program = runner.analyzeFile(cobolFile, CobolSourceFormatEnum.FIXED);
            
            // Access the parsed program
            CompilationUnit unit = program.getCompilationUnit();
            System.out.println("Successfully parsed: " + unit.getName());
            
        } catch (IOException e) {
            System.err.println("Error parsing file: " + e.getMessage());
        }
    }
}
```

### Parsing COBOL Code from String

```java
import io.proleap.cobol.asg.params.impl.CobolParserParamsImpl;
import io.proleap.cobol.asg.params.CobolParserParams;

public class StringParsingExample {
    public static void main(String[] args) {
        String cobolCode = """
            IDENTIFICATION DIVISION.
            PROGRAM-ID. HELLO-WORLD.
            
            DATA DIVISION.
            WORKING-STORAGE SECTION.
            01 WS-MESSAGE PIC X(30) VALUE 'Hello from COBOL Parser!'.
            
            PROCEDURE DIVISION.
            MAIN-LOGIC.
                DISPLAY WS-MESSAGE.
                STOP RUN.
            """;
        
        try {
            CobolParserRunner runner = new CobolParserRunnerImpl();
            CobolParserParams params = new CobolParserParamsImpl();
            params.setFormat(CobolSourceFormatEnum.VARIABLE);
            
            Program program = runner.analyzeCode(cobolCode, "HELLO-WORLD", params);
            
            CompilationUnit unit = program.getCompilationUnit("HELLO-WORLD");
            System.out.println("Parsed program: " + unit.getName());
            
        } catch (IOException e) {
            System.err.println("Error: " + e.getMessage());
        }
    }
}
```

## Configuration Examples

### Basic Configuration Setup

```java
import io.proleap.cobol.asg.params.impl.CobolParserParamsImpl;
import io.proleap.cobol.asg.params.CobolDialect;
import java.nio.charset.StandardCharsets;

public class BasicConfigurationExample {
    public static CobolParserParams createBasicParams() {
        CobolParserParams params = new CobolParserParamsImpl();
        
        // Set source format
        params.setFormat(CobolSourceFormatEnum.FIXED);
        
        // Set COBOL dialect
        params.setDialect(CobolDialect.ANSI85);
        
        // Set character encoding
        params.setCharset(StandardCharsets.UTF_8);
        
        return params;
    }
}
```

### Advanced Configuration with COPY Books

```java
import java.util.Arrays;
import java.util.List;

public class AdvancedConfigurationExample {
    public static CobolParserParams createAdvancedParams() {
        CobolParserParams params = new CobolParserParamsImpl();
        
        // Configure COPY book directories
        List<File> copyBookDirs = Arrays.asList(
            new File("src/main/cobol/copybooks"),
            new File("lib/includes"),
            new File("/system/copybooks")
        );
        params.setCopyBookDirectories(copyBookDirs);
        
        // Configure COPY book file extensions
        List<String> extensions = Arrays.asList("cpy", "inc", "copy", "cbl");
        params.setCopyBookExtensions(extensions);
        
        // Configure specific COPY book files
        List<File> copyBookFiles = Arrays.asList(
            new File("common/CONSTANTS.cpy"),
            new File("common/SQLCA.inc")
        );
        params.setCopyBookFiles(copyBookFiles);
        
        // Set format and dialect
        params.setFormat(CobolSourceFormatEnum.VARIABLE);
        params.setDialect(CobolDialect.MF); // Micro Focus
        
        // Configure error handling
        params.setIgnoreSyntaxErrors(false); // Strict parsing
        
        return params;
    }
}
```

### Dialect-Specific Configuration

```java
public class DialectConfigurationExamples {
    
    // ANSI-85 Standard Configuration
    public static CobolParserParams createAnsi85Config() {
        CobolParserParams params = new CobolParserParamsImpl();
        params.setDialect(CobolDialect.ANSI85);
        params.setFormat(CobolSourceFormatEnum.FIXED);
        params.setCharset(StandardCharsets.US_ASCII);
        return params;
    }
    
    // Micro Focus Configuration
    public static CobolParserParams createMicroFocusConfig() {
        CobolParserParams params = new CobolParserParamsImpl();
        params.setDialect(CobolDialect.MF);
        params.setFormat(CobolSourceFormatEnum.VARIABLE);
        params.setCharset(StandardCharsets.UTF_8);
        return params;
    }
    
    // IBM OS/VS Configuration
    public static CobolParserParams createOSVSConfig() {
        CobolParserParams params = new CobolParserParamsImpl();
        params.setDialect(CobolDialect.OSVS);
        params.setFormat(CobolSourceFormatEnum.FIXED);
        params.setCharset(StandardCharsets.IBM500); // EBCDIC
        return params;
    }
}
```

## File Processing Examples

### Single File Processing

```java
import java.nio.file.Path;
import java.nio.file.Paths;

public class SingleFileProcessingExample {
    
    public static void processCobolFile(String filePath) {
        try {
            Path path = Paths.get(filePath);
            File cobolFile = path.toFile();
            
            if (!cobolFile.exists()) {
                System.err.println("File not found: " + filePath);
                return;
            }
            
            // Determine format based on file extension or content
            CobolSourceFormatEnum format = determineFormat(cobolFile);
            
            CobolParserRunner runner = new CobolParserRunnerImpl();
            Program program = runner.analyzeFile(cobolFile, format);
            
            System.out.println("Processing: " + cobolFile.getName());
            analyzeProgram(program);
            
        } catch (IOException e) {
            System.err.println("Error processing " + filePath + ": " + e.getMessage());
        }
    }
    
    private static CobolSourceFormatEnum determineFormat(File file) {
        String name = file.getName().toLowerCase();
        if (name.endsWith(".cob") || name.endsWith(".cbl")) {
            return CobolSourceFormatEnum.FIXED;
        } else if (name.endsWith(".eco")) {
            return CobolSourceFormatEnum.VARIABLE;
        } else {
            return CobolSourceFormatEnum.FIXED; // Default
        }
    }
    
    private static void analyzeProgram(Program program) {
        CompilationUnit unit = program.getCompilationUnit();
        if (unit != null) {
            System.out.println("  Program units: " + unit.getProgramUnits().size());
        }
    }
}
```

### Batch File Processing

```java
import java.nio.file.*;
import java.util.stream.Stream;

public class BatchProcessingExample {
    
    public static void processDirectory(String directoryPath) {
        Path dir = Paths.get(directoryPath);
        
        if (!Files.isDirectory(dir)) {
            System.err.println("Not a directory: " + directoryPath);
            return;
        }
        
        CobolParserRunner runner = new CobolParserRunnerImpl();
        CobolParserParams params = createBatchProcessingParams();
        
        try (Stream<Path> paths = Files.walk(dir)) {
            paths.filter(path -> isCobolFile(path))
                 .forEach(path -> processFile(runner, path, params));
        } catch (IOException e) {
            System.err.println("Error processing directory: " + e.getMessage());
        }
    }
    
    private static boolean isCobolFile(Path path) {
        if (!Files.isRegularFile(path)) return false;
        
        String fileName = path.getFileName().toString().toLowerCase();
        return fileName.endsWith(".cbl") || 
               fileName.endsWith(".cob") || 
               fileName.endsWith(".eco");
    }
    
    private static void processFile(CobolParserRunner runner, Path path, CobolParserParams params) {
        try {
            System.out.println("Processing: " + path.getFileName());
            
            Program program = runner.analyzeFile(path.toFile(), params);
            
            // Perform analysis
            generateReport(program, path);
            
        } catch (Exception e) {
            System.err.println("Error processing " + path + ": " + e.getMessage());
        }
    }
    
    private static CobolParserParams createBatchProcessingParams() {
        CobolParserParams params = new CobolParserParamsImpl();
        params.setFormat(CobolSourceFormatEnum.FIXED);
        params.setIgnoreSyntaxErrors(true); // Continue on errors
        return params;
    }
    
    private static void generateReport(Program program, Path path) {
        CompilationUnit unit = program.getCompilationUnit();
        if (unit != null) {
            System.out.printf("  %s: %d program units%n", 
                path.getFileName(), unit.getProgramUnits().size());
        }
    }
}
```

### Recursive Directory Processing

```java
import java.util.concurrent.atomic.AtomicInteger;

public class RecursiveProcessingExample {
    
    public static void processDirectoryRecursively(String rootPath) {
        Path root = Paths.get(rootPath);
        AtomicInteger fileCount = new AtomicInteger(0);
        AtomicInteger successCount = new AtomicInteger(0);
        
        CobolParserRunner runner = new CobolParserRunnerImpl();
        
        try (Stream<Path> paths = Files.find(root, Integer.MAX_VALUE,
                (path, attrs) -> attrs.isRegularFile() && isCobolFile(path))) {
            
            paths.forEach(path -> {
                fileCount.incrementAndGet();
                try {
                    Program program = runner.analyzeFile(path.toFile(), 
                        CobolSourceFormatEnum.FIXED);
                    successCount.incrementAndGet();
                    
                    System.out.println("✓ " + path);
                    
                } catch (IOException e) {
                    System.err.println("✗ " + path + " - " + e.getMessage());
                }
            });
            
        } catch (IOException e) {
            System.err.println("Error walking directory: " + e.getMessage());
        }
        
        System.out.printf("Processed %d files, %d successful%n", 
            fileCount.get(), successCount.get());
    }
    
    private static boolean isCobolFile(Path path) {
        String name = path.getFileName().toString().toLowerCase();
        return name.endsWith(".cbl") || name.endsWith(".cob") || name.endsWith(".eco");
    }
}
```

## Data Structure Analysis

### Working Storage Analysis

```java
import io.proleap.cobol.asg.metamodel.data.DataDivision;
import io.proleap.cobol.asg.metamodel.data.workingstorage.WorkingStorageSection;
import io.proleap.cobol.asg.metamodel.data.datadescription.DataDescriptionEntry;

public class WorkingStorageAnalysisExample {
    
    public static void analyzeWorkingStorage(Program program) {
        CompilationUnit unit = program.getCompilationUnit();
        if (unit == null) return;
        
        ProgramUnit programUnit = unit.getProgramUnit();
        if (programUnit == null) return;
        
        DataDivision dataDiv = programUnit.getDataDivision();
        if (dataDiv == null) {
            System.out.println("No DATA DIVISION found");
            return;
        }
        
        WorkingStorageSection wsSection = dataDiv.getWorkingStorageSection();
        if (wsSection == null) {
            System.out.println("No WORKING-STORAGE SECTION found");
            return;
        }
        
        List<DataDescriptionEntry> entries = wsSection.getDataDescriptionEntries();
        System.out.println("Working Storage Variables:");
        System.out.println("========================");
        
        for (DataDescriptionEntry entry : entries) {
            analyzeDataDescriptionEntry(entry, 0);
        }
    }
    
    private static void analyzeDataDescriptionEntry(DataDescriptionEntry entry, int indent) {
        String indentStr = "  ".repeat(indent);
        
        System.out.printf("%sLevel %02d: %s%n", 
            indentStr, entry.getLevelNumber(), entry.getName());
        
        // Analyze picture clause if present
        if (entry.getPictureClause() != null) {
            System.out.printf("%s  Picture: %s%n", 
                indentStr, entry.getPictureClause().getPictureString());
        }
        
        // Analyze value clause if present
        if (entry.getValueClause() != null) {
            System.out.printf("%s  Value: %s%n", 
                indentStr, entry.getValueClause().toString());
        }
        
        // Recursively analyze subordinate entries
        for (DataDescriptionEntry subordinate : entry.getDataDescriptionEntries()) {
            analyzeDataDescriptionEntry(subordinate, indent + 1);
        }
    }
}
```

### File Description Analysis

```java
import io.proleap.cobol.asg.metamodel.data.file.FileSection;
import io.proleap.cobol.asg.metamodel.data.file.FileDescriptionEntry;

public class FileAnalysisExample {
    
    public static void analyzeFiles(Program program) {
        CompilationUnit unit = program.getCompilationUnit();
        ProgramUnit programUnit = unit.getProgramUnit();
        DataDivision dataDiv = programUnit.getDataDivision();
        
        if (dataDiv == null) return;
        
        FileSection fileSection = dataDiv.getFileSection();
        if (fileSection == null) {
            System.out.println("No FILE SECTION found");
            return;
        }
        
        List<FileDescriptionEntry> fileEntries = fileSection.getFileDescriptionEntries();
        System.out.println("File Descriptions:");
        System.out.println("=================");
        
        for (FileDescriptionEntry fileEntry : fileEntries) {
            System.out.printf("File: %s%n", fileEntry.getName());
            
            // Analyze record descriptions
            List<DataDescriptionEntry> records = fileEntry.getDataDescriptionEntries();
            for (DataDescriptionEntry record : records) {
                System.out.printf("  Record: %s%n", record.getName());
                analyzeRecord(record, 2);
            }
        }
    }
    
    private static void analyzeRecord(DataDescriptionEntry record, int indent) {
        String indentStr = "  ".repeat(indent);
        
        for (DataDescriptionEntry field : record.getDataDescriptionEntries()) {
            System.out.printf("%s%s (Level %02d)%n", 
                indentStr, field.getName(), field.getLevelNumber());
            
            if (field.getPictureClause() != null) {
                System.out.printf("%s  PIC %s%n", 
                    indentStr, field.getPictureClause().getPictureString());
            }
        }
    }
}
```

### Linkage Section Analysis

```java
import io.proleap.cobol.asg.metamodel.data.linkage.LinkageSection;

public class LinkageAnalysisExample {
    
    public static void analyzeLinkage(Program program) {
        CompilationUnit unit = program.getCompilationUnit();
        ProgramUnit programUnit = unit.getProgramUnit();
        DataDivision dataDiv = programUnit.getDataDivision();
        
        if (dataDiv == null) return;
        
        LinkageSection linkageSection = dataDiv.getLinkageSection();
        if (linkageSection == null) {
            System.out.println("No LINKAGE SECTION found");
            return;
        }
        
        List<DataDescriptionEntry> linkageEntries = linkageSection.getDataDescriptionEntries();
        System.out.println("Linkage Section Parameters:");
        System.out.println("==========================");
        
        for (DataDescriptionEntry entry : linkageEntries) {
            System.out.printf("Parameter: %s (Level %02d)%n", 
                entry.getName(), entry.getLevelNumber());
            
            if (entry.getPictureClause() != null) {
                System.out.printf("  Picture: %s%n", 
                    entry.getPictureClause().getPictureString());
            }
        }
    }
}
```

## Procedure Analysis

### Procedure Division Analysis

```java
import io.proleap.cobol.asg.metamodel.procedure.ProcedureDivision;
import io.proleap.cobol.asg.metamodel.procedure.Paragraph;
import io.proleap.cobol.asg.metamodel.procedure.Statement;

public class ProcedureAnalysisExample {
    
    public static void analyzeProcedures(Program program) {
        CompilationUnit unit = program.getCompilationUnit();
        ProgramUnit programUnit = unit.getProgramUnit();
        
        ProcedureDivision procDiv = programUnit.getProcedureDivision();
        if (procDiv == null) {
            System.out.println("No PROCEDURE DIVISION found");
            return;
        }
        
        System.out.println("Procedure Division Analysis:");
        System.out.println("===========================");
        
        // Analyze paragraphs
        List<Paragraph> paragraphs = procDiv.getParagraphs();
        System.out.printf("Found %d paragraphs:%n", paragraphs.size());
        
        for (Paragraph paragraph : paragraphs) {
            analyzeParagraph(paragraph);
        }
        
        // Analyze statements
        List<Statement> statements = procDiv.getStatements();
        System.out.printf("Found %d statements:%n", statements.size());
    }
    
    private static void analyzeParagraph(Paragraph paragraph) {
        System.out.printf("Paragraph: %s%n", paragraph.getName());
        
        List<Statement> statements = paragraph.getStatements();
        System.out.printf("  Statements: %d%n", statements.size());
        
        for (Statement stmt : statements) {
            analyzeStatement(stmt, 1);
        }
    }
    
    private static void analyzeStatement(Statement stmt, int indent) {
        String indentStr = "  ".repeat(indent);
        System.out.printf("%sStatement: %s%n", 
            indentStr, stmt.getClass().getSimpleName());
    }
}
```

### Statement Type Analysis

```java
import io.proleap.cobol.asg.metamodel.procedure.display.DisplayStatement;
import io.proleap.cobol.asg.metamodel.procedure.move.MoveStatement;
import io.proleap.cobol.asg.metamodel.procedure.call.CallStatement;

public class StatementTypeAnalysisExample {
    
    public static void analyzeStatementTypes(Program program) {
        CompilationUnit unit = program.getCompilationUnit();
        ProgramUnit programUnit = unit.getProgramUnit();
        ProcedureDivision procDiv = programUnit.getProcedureDivision();
        
        if (procDiv == null) return;
        
        Map<String, Integer> statementCounts = new HashMap<>();
        
        List<Statement> statements = procDiv.getStatements();
        for (Statement stmt : statements) {
            String type = getStatementType(stmt);
            statementCounts.merge(type, 1, Integer::sum);
        }
        
        System.out.println("Statement Type Distribution:");
        System.out.println("===========================");
        
        statementCounts.entrySet().stream()
            .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
            .forEach(entry -> 
                System.out.printf("%s: %d%n", entry.getKey(), entry.getValue()));
    }
    
    private static String getStatementType(Statement stmt) {
        if (stmt instanceof DisplayStatement) return "DISPLAY";
        if (stmt instanceof MoveStatement) return "MOVE";
        if (stmt instanceof CallStatement) return "CALL";
        // Add more statement types as needed
        return stmt.getClass().getSimpleName();
    }
}
```

## Error Handling Examples

### Basic Error Handling

```java
public class BasicErrorHandlingExample {
    
    public static void parseWithErrorHandling(File cobolFile) {
        CobolParserRunner runner = new CobolParserRunnerImpl();
        
        try {
            Program program = runner.analyzeFile(cobolFile, CobolSourceFormatEnum.FIXED);
            System.out.println("Parse successful!");
            
        } catch (CobolParserException e) {
            System.err.println("COBOL Parse Error: " + e.getMessage());
            // Handle parsing-specific errors
            
        } catch (IOException e) {
            System.err.println("I/O Error: " + e.getMessage());
            // Handle file access errors
            
        } catch (Exception e) {
            System.err.println("Unexpected error: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

### Tolerant Parsing with Error Collection

```java
import java.util.ArrayList;
import java.util.List;

public class TolerantParsingExample {
    
    public static class ParseResult {
        private final Program program;
        private final List<String> warnings;
        private final boolean hasErrors;
        
        public ParseResult(Program program, List<String> warnings, boolean hasErrors) {
            this.program = program;
            this.warnings = new ArrayList<>(warnings);
            this.hasErrors = hasErrors;
        }
        
        // Getters
        public Program getProgram() { return program; }
        public List<String> getWarnings() { return warnings; }
        public boolean hasErrors() { return hasErrors; }
        public boolean isSuccessful() { return program != null; }
    }
    
    public static ParseResult parseWithTolerance(File cobolFile) {
        List<String> warnings = new ArrayList<>();
        boolean hasErrors = false;
        Program program = null;
        
        try {
            // First try strict parsing
            CobolParserRunner runner = new CobolParserRunnerImpl();
            program = runner.analyzeFile(cobolFile, CobolSourceFormatEnum.FIXED);
            
        } catch (CobolParserException e) {
            warnings.add("Syntax errors detected: " + e.getMessage());
            hasErrors = true;
            
            // Try tolerant parsing
            try {
                CobolParserParams params = new CobolParserParamsImpl();
                params.setFormat(CobolSourceFormatEnum.FIXED);
                params.setIgnoreSyntaxErrors(true);
                
                CobolParserRunner runner = new CobolParserRunnerImpl();
                program = runner.analyzeFile(cobolFile, params);
                
                warnings.add("Parsed with syntax error tolerance");
                
            } catch (Exception e2) {
                warnings.add("Failed even with tolerant parsing: " + e2.getMessage());
            }
            
        } catch (IOException e) {
            warnings.add("I/O error: " + e.getMessage());
            hasErrors = true;
        }
        
        return new ParseResult(program, warnings, hasErrors);
    }
}
```

### Error Recovery and Reporting

```java
public class ErrorRecoveryExample {
    
    public static class ParseReport {
        private final String fileName;
        private final boolean successful;
        private final List<String> errors;
        private final List<String> warnings;
        private final Program program;
        
        public ParseReport(String fileName, boolean successful, 
                          List<String> errors, List<String> warnings, Program program) {
            this.fileName = fileName;
            this.successful = successful;
            this.errors = new ArrayList<>(errors);
            this.warnings = new ArrayList<>(warnings);
            this.program = program;
        }
        
        public void printReport() {
            System.out.println("Parse Report for: " + fileName);
            System.out.println("=================" + "=".repeat(fileName.length()));
            System.out.println("Status: " + (successful ? "SUCCESS" : "FAILED"));
            
            if (!warnings.isEmpty()) {
                System.out.println("Warnings:");
                warnings.forEach(w -> System.out.println("  - " + w));
            }
            
            if (!errors.isEmpty()) {
                System.out.println("Errors:");
                errors.forEach(e -> System.out.println("  - " + e));
            }
            
            if (program != null) {
                CompilationUnit unit = program.getCompilationUnit();
                if (unit != null) {
                    System.out.println("Program units found: " + unit.getProgramUnits().size());
                }
            }
            
            System.out.println();
        }
    }
    
    public static ParseReport parseWithFullErrorHandling(File cobolFile) {
        List<String> errors = new ArrayList<>();
        List<String> warnings = new ArrayList<>();
        Program program = null;
        boolean successful = false;
        
        // Validate file first
        if (!cobolFile.exists()) {
            errors.add("File does not exist: " + cobolFile.getAbsolutePath());
            return new ParseReport(cobolFile.getName(), false, errors, warnings, null);
        }
        
        if (!cobolFile.canRead()) {
            errors.add("Cannot read file: " + cobolFile.getAbsolutePath());
            return new ParseReport(cobolFile.getName(), false, errors, warnings, null);
        }
        
        try {
            // Attempt parsing with different strategies
            program = attemptParsing(cobolFile, errors, warnings);
            successful = (program != null);
            
        } catch (Exception e) {
            errors.add("Unexpected error: " + e.getMessage());
        }
        
        return new ParseReport(cobolFile.getName(), successful, errors, warnings, program);
    }
    
    private static Program attemptParsing(File cobolFile, List<String> errors, List<String> warnings) 
            throws IOException {
        
        CobolParserRunner runner = new CobolParserRunnerImpl();
        
        // Strategy 1: Try FIXED format, strict parsing
        try {
            return runner.analyzeFile(cobolFile, CobolSourceFormatEnum.FIXED);
        } catch (CobolParserException e) {
            warnings.add("FIXED format failed: " + e.getMessage());
        }
        
        // Strategy 2: Try VARIABLE format, strict parsing
        try {
            return runner.analyzeFile(cobolFile, CobolSourceFormatEnum.VARIABLE);
        } catch (CobolParserException e) {
            warnings.add("VARIABLE format failed: " + e.getMessage());
        }
        
        // Strategy 3: Try TANDEM format, strict parsing
        try {
            return runner.analyzeFile(cobolFile, CobolSourceFormatEnum.TANDEM);
        } catch (CobolParserException e) {
            warnings.add("TANDEM format failed: " + e.getMessage());
        }
        
        // Strategy 4: Try with error tolerance
        try {
            CobolParserParams params = new CobolParserParamsImpl();
            params.setFormat(CobolSourceFormatEnum.FIXED);
            params.setIgnoreSyntaxErrors(true);
            
            Program program = runner.analyzeFile(cobolFile, params);
            warnings.add("Parsed with error tolerance");
            return program;
            
        } catch (Exception e) {
            errors.add("All parsing strategies failed: " + e.getMessage());
        }
        
        return null;
    }
}
```

## Advanced Use Cases

### Custom ASG Visitor

```java
import io.proleap.cobol.asg.visitor.ParserVisitor;
import org.antlr.v4.runtime.tree.ParseTree;

public class CustomVisitorExample {
    
    public static class DataItemCollector implements ParserVisitor {
        private final List<String> dataItems = new ArrayList<>();
        private final Program program;
        
        public DataItemCollector(Program program) {
            this.program = program;
        }
        
        @Override
        public Boolean visit(ParseTree parseTree) {
            // Custom logic to collect data items
            String text = parseTree.getText();
            
            // Look for data item patterns
            if (text.matches("\\d{2}\\s+[A-Z][A-Z0-9-]*")) {
                dataItems.add(text);
            }
            
            return true; // Continue visiting children
        }
        
        public List<String> getDataItems() {
            return new ArrayList<>(dataItems);
        }
    }
    
    public static List<String> extractDataItems(Program program) {
        CompilationUnit unit = program.getCompilationUnit();
        if (unit == null) return Collections.emptyList();
        
        DataItemCollector collector = new DataItemCollector(program);
        collector.visit(unit.getCtx());
        
        return collector.getDataItems();
    }
}
```

### Program Metrics Calculation

```java
public class ProgramMetricsExample {
    
    public static class ProgramMetrics {
        private int lineCount;
        private int paragraphCount;
        private int statementCount;
        private int dataItemCount;
        private int copyBookCount;
        
        // Getters and setters
        public int getLineCount() { return lineCount; }
        public void setLineCount(int lineCount) { this.lineCount = lineCount; }
        
        public int getParagraphCount() { return paragraphCount; }
        public void setParagraphCount(int paragraphCount) { this.paragraphCount = paragraphCount; }
        
        public int getStatementCount() { return statementCount; }
        public void setStatementCount(int statementCount) { this.statementCount = statementCount; }
        
        public int getDataItemCount() { return dataItemCount; }
        public void setDataItemCount(int dataItemCount) { this.dataItemCount = dataItemCount; }
        
        public int getCopyBookCount() { return copyBookCount; }
        public void setCopyBookCount(int copyBookCount) { this.copyBookCount = copyBookCount; }
        
        @Override
        public String toString() {
            return String.format(
                "Program Metrics:%n" +
                "  Lines of code: %d%n" +
                "  Paragraphs: %d%n" +
                "  Statements: %d%n" +
                "  Data items: %d%n" +
                "  Copy books: %d%n",
                lineCount, paragraphCount, statementCount, dataItemCount, copyBookCount
            );
        }
    }
    
    public static ProgramMetrics calculateMetrics(Program program) {
        ProgramMetrics metrics = new ProgramMetrics();
        
        CompilationUnit unit = program.getCompilationUnit();
        if (unit == null) return metrics;
        
        // Line count
        metrics.setLineCount(unit.getLines().size());
        
        ProgramUnit programUnit = unit.getProgramUnit();
        if (programUnit == null) return metrics;
        
        // Data items count
        DataDivision dataDiv = programUnit.getDataDivision();
        if (dataDiv != null) {
            metrics.setDataItemCount(countDataItems(dataDiv));
        }
        
        // Procedure metrics
        ProcedureDivision procDiv = programUnit.getProcedureDivision();
        if (procDiv != null) {
            metrics.setParagraphCount(procDiv.getParagraphs().size());
            metrics.setStatementCount(procDiv.getStatements().size());
        }
        
        return metrics;
    }
    
    private static int countDataItems(DataDivision dataDiv) {
        int count = 0;
        
        // Count working storage items
        WorkingStorageSection wsSection = dataDiv.getWorkingStorageSection();
        if (wsSection != null) {
            count += countDataDescriptionEntries(wsSection.getDataDescriptionEntries());
        }
        
        // Count linkage items
        LinkageSection linkageSection = dataDiv.getLinkageSection();
        if (linkageSection != null) {
            count += countDataDescriptionEntries(linkageSection.getDataDescriptionEntries());
        }
        
        // Count file items
        FileSection fileSection = dataDiv.getFileSection();
        if (fileSection != null) {
            for (FileDescriptionEntry fileEntry : fileSection.getFileDescriptionEntries()) {
                count += countDataDescriptionEntries(fileEntry.getDataDescriptionEntries());
            }
        }
        
        return count;
    }
    
    private static int countDataDescriptionEntries(List<DataDescriptionEntry> entries) {
        int count = entries.size();
        for (DataDescriptionEntry entry : entries) {
            count += countDataDescriptionEntries(entry.getDataDescriptionEntries());
        }
        return count;
    }
}
```

## Performance Optimization

### Memory-Efficient Processing

```java
import java.lang.management.ManagementFactory;
import java.lang.management.MemoryMXBean;

public class MemoryEfficientProcessingExample {
    
    public static void processLargeFiles(List<File> cobolFiles) {
        CobolParserRunner runner = new CobolParserRunnerImpl();
        CobolParserParams params = createOptimizedParams();
        
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        
        for (File file : cobolFiles) {
            try {
                // Monitor memory before processing
                long memoryBefore = memoryBean.getHeapMemoryUsage().getUsed();
                
                System.out.println("Processing: " + file.getName());
                Program program = runner.analyzeFile(file, params);
                
                // Process immediately and release reference
                processProgram(program);
                program = null; // Release reference
                
                // Suggest garbage collection for large files
                if (file.length() > 1024 * 1024) { // > 1MB
                    System.gc();
                }
                
                long memoryAfter = memoryBean.getHeapMemoryUsage().getUsed();
                long memoryUsed = (memoryAfter - memoryBefore) / 1024 / 1024; // MB
                
                System.out.printf("  Memory used: %d MB%n", memoryUsed);
                
            } catch (IOException e) {
                System.err.println("Error processing " + file.getName() + ": " + e.getMessage());
            }
        }
    }
    
    private static CobolParserParams createOptimizedParams() {
        CobolParserParams params = new CobolParserParamsImpl();
        params.setFormat(CobolSourceFormatEnum.FIXED);
        params.setIgnoreSyntaxErrors(true); // Faster parsing
        return params;
    }
    
    private static void processProgram(Program program) {
        // Do minimal processing to avoid memory retention
        CompilationUnit unit = program.getCompilationUnit();
        if (unit != null) {
            System.out.println("  Processed: " + unit.getName());
        }
    }
}
```

### Parallel Processing

```java
import java.util.concurrent.*;
import java.util.stream.Collectors;

public class ParallelProcessingExample {
    
    public static void processFilesInParallel(List<File> cobolFiles, int threadCount) {
        ExecutorService executor = Executors.newFixedThreadPool(threadCount);
        
        try {
            // Submit all tasks
            List<Future<ParseResult>> futures = cobolFiles.stream()
                .map(file -> executor.submit(() -> parseFile(file)))
                .collect(Collectors.toList());
            
            // Collect results
            List<ParseResult> results = new ArrayList<>();
            for (Future<ParseResult> future : futures) {
                try {
                    ParseResult result = future.get();
                    results.add(result);
                } catch (ExecutionException e) {
                    System.err.println("Execution error: " + e.getCause().getMessage());
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
            
            // Process results
            printSummary(results);
            
        } finally {
            executor.shutdown();
            try {
                if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
                    executor.shutdownNow();
                }
            } catch (InterruptedException e) {
                executor.shutdownNow();
                Thread.currentThread().interrupt();
            }
        }
    }
    
    private static ParseResult parseFile(File file) {
        long startTime = System.currentTimeMillis();
        
        try {
            CobolParserRunner runner = new CobolParserRunnerImpl();
            Program program = runner.analyzeFile(file, CobolSourceFormatEnum.FIXED);
            
            long duration = System.currentTimeMillis() - startTime;
            return new ParseResult(file.getName(), true, duration, null);
            
        } catch (IOException e) {
            long duration = System.currentTimeMillis() - startTime;
            return new ParseResult(file.getName(), false, duration, e.getMessage());
        }
    }
    
    private static class ParseResult {
        private final String fileName;
        private final boolean successful;
        private final long durationMs;
        private final String errorMessage;
        
        public ParseResult(String fileName, boolean successful, long durationMs, String errorMessage) {
            this.fileName = fileName;
            this.successful = successful;
            this.durationMs = durationMs;
            this.errorMessage = errorMessage;
        }
        
        // Getters
        public String getFileName() { return fileName; }
        public boolean isSuccessful() { return successful; }
        public long getDurationMs() { return durationMs; }
        public String getErrorMessage() { return errorMessage; }
    }
    
    private static void printSummary(List<ParseResult> results) {
        long totalTime = results.stream().mapToLong(ParseResult::getDurationMs).sum();
        long successCount = results.stream().mapToLong(r -> r.isSuccessful() ? 1 : 0).sum();
        
        System.out.println("Parallel Processing Summary:");
        System.out.println("===========================");
        System.out.printf("Total files: %d%n", results.size());
        System.out.printf("Successful: %d%n", successCount);
        System.out.printf("Failed: %d%n", results.size() - successCount);
        System.out.printf("Total time: %d ms%n", totalTime);
        System.out.printf("Average time per file: %.2f ms%n", 
            (double) totalTime / results.size());
    }
}
```

### Streaming Processing for Large Datasets

```java
import java.nio.file.Files;
import java.util.stream.Stream;

public class StreamingProcessingExample {
    
    public static void processLargeDataset(Path rootDirectory) {
        CobolParserRunner runner = new CobolParserRunnerImpl();
        
        try (Stream<Path> pathStream = Files.find(rootDirectory, Integer.MAX_VALUE,
                (path, attrs) -> attrs.isRegularFile() && isCobolFile(path))) {
            
            pathStream
                .parallel() // Enable parallel processing
                .forEach(path -> {
                    try {
                        // Process each file in stream
                        processFileStreaming(runner, path);
                    } catch (Exception e) {
                        System.err.println("Error processing " + path + ": " + e.getMessage());
                    }
                });
                
        } catch (IOException e) {
            System.err.println("Error walking directory: " + e.getMessage());
        }
    }
    
    private static void processFileStreaming(CobolParserRunner runner, Path path) throws IOException {
        // Use minimal memory approach
        CobolParserParams params = new CobolParserParamsImpl();
        params.setFormat(CobolSourceFormatEnum.FIXED);
        params.setIgnoreSyntaxErrors(true);
        
        Program program = runner.analyzeFile(path.toFile(), params);
        
        // Extract only essential information
        String fileName = path.getFileName().toString();
        int programUnits = 0;
        
        CompilationUnit unit = program.getCompilationUnit();
        if (unit != null) {
            programUnits = unit.getProgramUnits().size();
        }
        
        // Log result and release memory immediately
        System.out.printf("%s: %d program units%n", fileName, programUnits);
        
        // Explicit cleanup
        program = null;
        unit = null;
    }
    
    private static boolean isCobolFile(Path path) {
        String fileName = path.getFileName().toString().toLowerCase();
        return fileName.endsWith(".cbl") || fileName.endsWith(".cob") || fileName.endsWith(".eco");
    }
}
```

---

These examples demonstrate the comprehensive capabilities of the ProLeap COBOL Parser API. Each example can be adapted and combined to meet specific requirements for COBOL source code analysis and processing.