# Preprocessor

The `io.proleap.cobol.preprocessor.CobolPreprocessor` prepares COBOL source for parsing. It processes COPY/REPLACE/CBL/PROCESS directives and extracts `EXEC SQL`, `EXEC SQLIMS`, and `EXEC CICS` blocks as text.

## Public API

```17:109:src/main/java/io/proleap/cobol/preprocessor/CobolPreprocessor.java
public interface CobolPreprocessor {

	public enum CobolSourceFormatEnum {
		// ... see source for details ...
	}

	String process(File cobolFile, CobolParserParams params) throws IOException;

	String process(String cobolCode, CobolParserParams params);
}
```

### Source formats

`CobolPreprocessor.CobolSourceFormatEnum` defines the layout used by your source:

- `FIXED`: ANSI/IBM 80-column fixed format (sequence 1-6, indicator 7, A 8-12, B 13-72, comments 73-80)
- `TANDEM`: HP Tandem format (indicator 1, optional A 2-5, optional B 6-132)
- `VARIABLE`: Variable format (sequence 1-6, indicator 7, optional A 8-12, optional B 13-*)

## Examples

Process a file and obtain preprocessed text:

```java
File file = new File("/path/to/program.cbl");
io.proleap.cobol.asg.params.impl.CobolParserParamsImpl params = new io.proleap.cobol.asg.params.impl.CobolParserParamsImpl();
params.setFormat(io.proleap.cobol.preprocessor.CobolPreprocessor.CobolSourceFormatEnum.FIXED);

String text = new io.proleap.cobol.preprocessor.impl.CobolPreprocessorImpl()
  .process(file, params);
```