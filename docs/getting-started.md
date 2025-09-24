# Getting Started

## Installation (Maven)

Add the dependency to your `pom.xml`:

```xml
<dependency>
  <groupId>io.github.uwol</groupId>
  <artifactId>proleap-cobol-parser</artifactId>
  <version>4.0.0</version>
</dependency>
```

Build requirements:

- JDK 17+
- Maven 3+

## Parse COBOL and build the ASG

```java
// Generate ASG from COBOL code on disk
java.io.File inputFile = new java.io.File("src/test/resources/io/proleap/cobol/asg/HelloWorld.cbl");
io.proleap.cobol.preprocessor.CobolPreprocessor.CobolSourceFormatEnum format = io.proleap.cobol.preprocessor.CobolPreprocessor.CobolSourceFormatEnum.TANDEM;
io.proleap.cobol.asg.metamodel.Program program = new io.proleap.cobol.asg.runner.impl.CobolParserRunnerImpl().analyzeFile(inputFile, format);

// Navigate the ASG
io.proleap.cobol.asg.metamodel.CompilationUnit compilationUnit = program.getCompilationUnit("HelloWorld");
io.proleap.cobol.asg.metamodel.ProgramUnit programUnit = compilationUnit.getProgramUnit();
io.proleap.cobol.asg.metamodel.data.DataDivision dataDivision = programUnit.getDataDivision();
io.proleap.cobol.asg.metamodel.data.datadescription.DataDescriptionEntry dataDescriptionEntry = dataDivision.getWorkingStorageSection().getDataDescriptionEntry("ITEMS");
Integer levelNumber = dataDescriptionEntry.getLevelNumber();
```

## Traverse the AST with a Visitor

```java
// Build the Program as above, then:
io.proleap.cobol.CobolBaseVisitor<Boolean> visitor = new io.proleap.cobol.CobolBaseVisitor<Boolean>() {
  @Override
  public Boolean visitDataDescriptionEntryFormat1(final io.proleap.cobol.CobolParser.DataDescriptionEntryFormat1Context ctx) {
    io.proleap.cobol.asg.metamodel.data.datadescription.DataDescriptionEntry entry = (io.proleap.cobol.asg.metamodel.data.datadescription.DataDescriptionEntry) program.getASGElementRegistry().getASGElement(ctx);
    String name = entry.getName();
    return visitChildren(ctx);
  }
};

for (final io.proleap.cobol.asg.metamodel.CompilationUnit compilationUnit : program.getCompilationUnits()) {
  visitor.visit(compilationUnit.getCtx());
}
```

## Generate API Reference

- Generate Javadoc locally:

```bash
mvn -DskipTests javadoc:javadoc
```

- Open `target/site/apidocs/index.html`, or copy to this docs site:

```bash
mkdir -p docs && rm -rf docs/api && cp -r target/site/apidocs docs/api
```