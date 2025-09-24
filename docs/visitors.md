# Visitors

Visitors are used to traverse the ANTLR parse tree (AST). This project provides a `ParserVisitor` abstraction and several concrete visitors that build the ASG and analyze sections.

## Public API

```13:16:src/main/java/io/proleap/cobol/asg/visitor/ParserVisitor.java
public interface ParserVisitor {
	Boolean visit(ParseTree parseTree);
}
```

Key built-in visitors include:

- `CobolCompilationUnitVisitorImpl`
- `CobolProgramUnitVisitorImpl`
- `CobolDataDivisionStep1VisitorImpl`
- `CobolDataDivisionStep2VisitorImpl`
- `CobolProcedureDivisionVisitorImpl`
- `CobolProcedureStatementVisitorImpl`

These are orchestrated inside `CobolParserRunnerImpl`.

## Example: custom visitor using generated base visitor

```java
io.proleap.cobol.CobolBaseVisitor<Boolean> visitor = new io.proleap.cobol.CobolBaseVisitor<Boolean>() {
  @Override
  public Boolean visitDataDescriptionEntryFormat1(final io.proleap.cobol.CobolParser.DataDescriptionEntryFormat1Context ctx) {
    io.proleap.cobol.asg.metamodel.data.datadescription.DataDescriptionEntry entry = (io.proleap.cobol.asg.metamodel.data.datadescription.DataDescriptionEntry) program.getASGElementRegistry().getASGElement(ctx);
    System.out.println(entry.getName());
    return visitChildren(ctx);
  }
};

for (final io.proleap.cobol.asg.metamodel.CompilationUnit compilationUnit : program.getCompilationUnits()) {
  visitor.visit(compilationUnit.getCtx());
}
```