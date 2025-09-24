# ProLeap COBOL Parser - Documentation Index

## 📑 Complete Documentation Suite

This comprehensive documentation covers all aspects of using the ProLeap COBOL Parser, from basic setup to advanced use cases.

### 📖 Core Documentation

| Document | Description | Target Audience |
|----------|-------------|-----------------|
| **[Getting Started](GETTING_STARTED.md)** | Step-by-step tutorial for first-time users | Beginners |
| **[API Documentation](API_DOCUMENTATION.md)** | Complete API reference with detailed explanations | All users |
| **[Usage Examples](EXAMPLES.md)** | Practical code examples for common scenarios | Intermediate users |
| **[Quick Reference](QUICK_REFERENCE.md)** | Essential patterns and configuration guide | Experienced users |

### 🎯 Choose Your Starting Point

#### 👶 New to COBOL Parsing?
Start with the **[Getting Started Guide](GETTING_STARTED.md)**
- Basic setup and installation
- Your first parser implementation
- Simple examples with explanations
- Common configuration patterns

#### 🔍 Need Specific API Information?
Check the **[API Documentation](API_DOCUMENTATION.md)**
- Complete interface documentation
- Method signatures and parameters
- Configuration options
- Error handling strategies

#### 💡 Looking for Implementation Ideas?
Browse the **[Usage Examples](EXAMPLES.md)**
- Real-world scenarios
- Performance optimization
- Advanced use cases
- Best practices

#### ⚡ Need Quick Answers?
Use the **[Quick Reference](QUICK_REFERENCE.md)**
- Configuration cheat sheet
- Common patterns
- Troubleshooting guide
- Essential utilities

## 🏗️ What You Can Build

### Static Analysis Tools
- **Code Quality Analyzers**: Detect code smells and violations
- **Metrics Calculators**: Measure complexity, maintainability, and size
- **Documentation Generators**: Auto-generate program documentation
- **Dependency Analyzers**: Map relationships between programs

### Migration and Modernization
- **Dialect Converters**: Convert between COBOL dialects
- **Format Transformers**: Change source code formats
- **Legacy Code Analyzers**: Understand existing codebases
- **Modernization Tools**: Prepare code for cloud migration

### Development Tools
- **Syntax Validators**: Check code correctness
- **Structure Browsers**: Navigate large codebases
- **Cross-Reference Generators**: Find variable and paragraph usage
- **Impact Analysis Tools**: Understand change effects

## 🚀 Core Capabilities

### Parser Features
- **Multi-Dialect Support**: ANSI-85, Micro Focus, IBM OS/VS
- **Format Flexibility**: Fixed, Variable, HP Tandem formats
- **Error Tolerance**: Continue parsing despite syntax errors
- **Complete ASG**: Full program structure representation

### Processing Features
- **COPY Book Support**: Automatic include processing
- **Batch Processing**: Handle multiple files efficiently
- **Parallel Processing**: Leverage multi-core systems
- **Memory Optimization**: Process large codebases

### Integration Features
- **Maven Integration**: Easy dependency management
- **Java 17+ Support**: Modern Java compatibility
- **ANTLR4 Based**: Robust parsing foundation
- **Open Source**: MIT licensed

## 📋 API Overview

### Main Entry Points

```java
// Primary parser interface
CobolParserRunner runner = new CobolParserRunnerImpl();

// Configuration
CobolParserParams params = new CobolParserParamsImpl();

// Preprocessor (if needed separately)
CobolPreprocessor preprocessor = new CobolPreprocessorImpl();
```

### Core Methods

```java
// Parse file with format
Program program = runner.analyzeFile(file, CobolSourceFormatEnum.FIXED);

// Parse file with parameters
Program program = runner.analyzeFile(file, params);

// Parse string content
Program program = runner.analyzeCode(content, name, params);
```

### Result Navigation

```java
// Access program structure
CompilationUnit unit = program.getCompilationUnit();
ProgramUnit programUnit = unit.getProgramUnit();

// Access divisions
DataDivision dataDiv = programUnit.getDataDivision();
ProcedureDivision procDiv = programUnit.getProcedureDivision();
```

## 🛠️ Configuration Quick Start

### Basic Configuration

```java
CobolParserParams params = new CobolParserParamsImpl();
params.setFormat(CobolSourceFormatEnum.FIXED);
params.setDialect(CobolDialect.ANSI85);
params.setCharset(StandardCharsets.UTF_8);
```

### Advanced Configuration

```java
// With COPY book support
params.setCopyBookDirectories(Arrays.asList(new File("copybooks")));
params.setCopyBookExtensions(Arrays.asList("cpy", "inc"));

// Error tolerance
params.setIgnoreSyntaxErrors(true);
```

## 🔧 Common Patterns

### File Processing

```java
// Single file
Program program = runner.analyzeFile(cobolFile, format);

// Multiple files
for (File file : cobolFiles) {
    Program program = runner.analyzeFile(file, params);
    processProgram(program);
}

// Directory processing
Files.walk(directory)
     .filter(path -> path.toString().endsWith(".cbl"))
     .forEach(path -> processFile(path));
```

### Error Handling

```java
try {
    Program program = runner.analyzeFile(file, params);
    // Process successfully parsed program
} catch (CobolParserException e) {
    // Handle parsing errors
} catch (IOException e) {
    // Handle file I/O errors
}
```

### Data Access

```java
// Working storage variables
WorkingStorageSection ws = dataDiv.getWorkingStorageSection();
List<DataDescriptionEntry> entries = ws.getDataDescriptionEntries();

// Procedure paragraphs
List<Paragraph> paragraphs = procDiv.getParagraphs();

// Statements
List<Statement> statements = procDiv.getStatements();
```

## 📊 Performance Guidelines

### Memory Management
- Process files individually for large batches
- Release program references after processing
- Use System.gc() for very large files

### Optimization
- Enable error tolerance for faster parsing
- Use parallel processing for multiple files
- Configure appropriate buffer sizes

### Scalability
- Implement streaming for large datasets
- Use connection pooling for database operations
- Monitor memory usage during batch processing

## 🆘 Troubleshooting

### Common Issues
- **Parse Errors**: Try different source formats or enable error tolerance
- **Memory Issues**: Process files individually and manage references
- **COPY Book Errors**: Configure copy book directories and extensions
- **Encoding Issues**: Set appropriate character encoding

### Debug Steps
1. Verify file exists and is readable
2. Try different source format settings
3. Enable error tolerance mode
4. Check copy book configuration
5. Validate character encoding

## 🤝 Contributing

The ProLeap COBOL Parser is open source. Ways to contribute:

- **Report Issues**: Found a bug? Report it on GitHub
- **Submit Examples**: Share your use cases and implementations
- **Improve Documentation**: Help make these docs even better
- **Code Contributions**: Submit pull requests for enhancements

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

## 🎯 Next Steps

1. **Start with [Getting Started](GETTING_STARTED.md)** if you're new
2. **Explore [Examples](EXAMPLES.md)** for practical implementations
3. **Reference [API Documentation](API_DOCUMENTATION.md)** for detailed information
4. **Use [Quick Reference](QUICK_REFERENCE.md)** for daily development

Happy COBOL parsing! 🚀