# Parser Runner

The `io.proleap.cobol.asg.runner.CobolParserRunner` is the primary entry point to parse COBOL and build the ASG.

## Public API

```19:25:src/main/java/io/proleap/cobol/asg/runner/CobolParserRunner.java
public interface CobolParserRunner {

	Program analyzeCode(String cobolCode, String compilationUnitName, CobolParserParams params) throws IOException;

	Program analyzeFile(File cobolFile, CobolParserParams params) throws IOException;

	Program analyzeFile(File cobolFile, CobolSourceFormatEnum format) throws IOException;
}
```

Default implementation: `io.proleap.cobol.asg.runner.impl.CobolParserRunnerImpl`

## Examples

### Minimal (auto params from format)

```java
File file = new File("/path/to/program.cbl");
io.proleap.cobol.asg.metamodel.Program program = new io.proleap.cobol.asg.runner.impl.CobolParserRunnerImpl()
  .analyzeFile(file, io.proleap.cobol.preprocessor.CobolPreprocessor.CobolSourceFormatEnum.FIXED);
```

### Custom parameters

```java
File file = new File("/path/to/program.cbl");
io.proleap.cobol.asg.params.impl.CobolParserParamsImpl params = new io.proleap.cobol.asg.params.impl.CobolParserParamsImpl();
params.setFormat(io.proleap.cobol.preprocessor.CobolPreprocessor.CobolSourceFormatEnum.VARIABLE);
params.setIgnoreSyntaxErrors(false);
params.setCopyBookDirectories(java.util.Arrays.asList(file.getParentFile()));

io.proleap.cobol.asg.metamodel.Program program = new io.proleap.cobol.asg.runner.impl.CobolParserRunnerImpl()
  .analyzeFile(file, params);
```