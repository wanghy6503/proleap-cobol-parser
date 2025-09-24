# Metamodel (ASG Overview)

The ASG (Abstract Semantic Graph) represents COBOL programs with resolved semantics on top of the parse tree. It includes program structure, divisions, sections, data descriptions, and procedure statements.

Key entry points:

- `io.proleap.cobol.asg.metamodel.Program`
- `io.proleap.cobol.asg.metamodel.CompilationUnit`
- `io.proleap.cobol.asg.metamodel.ProgramUnit`
- `io.proleap.cobol.asg.metamodel.data.*`
- `io.proleap.cobol.asg.metamodel.procedure.*`

For exhaustive types and methods, see the API Reference.

## Example navigation

```java
io.proleap.cobol.asg.metamodel.Program program = // ... from ParserRunner
for (io.proleap.cobol.asg.metamodel.CompilationUnit cu : program.getCompilationUnits()) {
  io.proleap.cobol.asg.metamodel.ProgramUnit pu = cu.getProgramUnit();
  io.proleap.cobol.asg.metamodel.data.DataDivision dd = pu.getDataDivision();
  if (dd != null && dd.getWorkingStorageSection() != null) {
    io.proleap.cobol.asg.metamodel.data.datadescription.DataDescriptionEntry entry = dd.getWorkingStorageSection().getDataDescriptionEntry("ITEMS");
    if (entry != null) {
      System.out.println(entry.getLevelNumber());
    }
  }
}
```