# Parameters

`io.proleap.cobol.asg.params.CobolParserParams` controls parsing and preprocessing behavior.

## Public API

```17:49:src/main/java/io/proleap/cobol/asg/params/CobolParserParams.java
public interface CobolParserParams {
	Charset getCharset();
	List<File> getCopyBookDirectories();
	List<String> getCopyBookExtensions();
	List<File> getCopyBookFiles();
	CobolDialect getDialect();
	CobolSourceFormatEnum getFormat();
	boolean getIgnoreSyntaxErrors();
	void setCharset(Charset charset);
	void setCopyBookDirectories(List<File> copyBookDirectories);
	void setCopyBookExtensions(List<String> copyBookExtensions);
	void setCopyBookFiles(List<File> copyBookFiles);
	void setDialect(CobolDialect dialect);
	void setFormat(CobolSourceFormatEnum format);
	void setIgnoreSyntaxErrors(boolean ignoreSyntaxErrors);
}
```

Default implementation: `io.proleap.cobol.asg.params.impl.CobolParserParamsImpl`

### Dialect

```11:13:src/main/java/io/proleap/cobol/asg/params/CobolDialect.java
public enum CobolDialect {
	ANSI85, MF, OSVS
}
```

## Examples

```java
io.proleap.cobol.asg.params.impl.CobolParserParamsImpl params = new io.proleap.cobol.asg.params.impl.CobolParserParamsImpl();
params.setFormat(io.proleap.cobol.preprocessor.CobolPreprocessor.CobolSourceFormatEnum.VARIABLE);
params.setDialect(io.proleap.cobol.asg.params.CobolDialect.MF);
params.setIgnoreSyntaxErrors(false);
params.setCopyBookDirectories(java.util.Arrays.asList(new java.io.File("/path/to/copybooks")));
```