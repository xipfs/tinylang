# tinylang


## 目的

利用 [Antlr4](http://www.antlr.org/) 设计一门简单的语言。

Antlr4 参考书目:

![Antlr4 权威指南](https://img12.360buyimg.com/n1/jfs/t5587/200/3113046617/215952/83fff3e4/59376ed2N7fd008f8.jpg)


![编程语言实现模式](https://img10.360buyimg.com/n1/19311/8e4d4199-7889-41fb-bff9-429e18868f6f.jpg)


## 语法

参考 [Christoph Mallon Jörg Herter October 21, 2009](langspec.pdf)

## BNF

```
BNF for TinyLang
NON-TERMINALS
Goal	::=	MainClass ( ClassDeclaration )* <EOF>
MainClass	::=	"class" Identifier "{" "public" "static" "void" "main" "(" "String" "[" "]" Identifier ")" "{" Statement "}" "}"
ClassDeclaration	::=	"class" Identifier ( "extends" Identifier )? "{" ( VarDeclaration )* ( MethodDeclaration )* "}"
VarDeclaration	::=	Type Identifier ";"
MethodDeclaration	::=	"public" Type Identifier "(" ( Type Identifier ( "," Type Identifier )* )? ")" "{" ( VarDeclaration )* ( Statement )* "return" Expression ";" "}"
Type	::=	"int" "[" "]"
|	"boolean"
|	"int"
|	Identifier
Statement	::=	"{" ( Statement )* "}"
|	"if" "(" Expression ")" Statement "else" Statement
|	"while" "(" Expression ")" Statement
|	"System.out.println" "(" Expression ")" ";"
|	Identifier "=" Expression ";"
|	Identifier "[" Expression "]" "=" Expression ";"
Expression	::=	Expression ( "&&" | "<" | "+" | "-" | "*" ) Expression
|	Expression "[" Expression "]"
|	Expression "." "length"
|	Expression "." Identifier "(" ( Expression ( "," Expression )* )? ")"
|	<INTEGER_LITERAL>
|	"true"
|	"false"
|	Identifier
|	"this"
|	"new" "int" "[" Expression "]"
|	"new" Identifier "(" ")"
|	"!" Expression
|	"(" Expression ")"
Identifier	::=	<IDENTIFIER>
```

## Antlr4 文法

```
grammar TinyJava;

goal	
:	mainClass classDeclaration* EOF
;


mainClass	
:	'class' Identifier '{' 'public' 'static' 'void' 'main' '(' 'String' '[' ']' Identifier ')' '{' statement '}' '}';

classDeclaration	
:	'class' Identifier ( 'extends' Identifier )? '{' fieldDeclaration* methodDeclaration* '}';

fieldDeclaration
:	varDeclaration ;

localDeclaration
:	varDeclaration ;

varDeclaration	
:	type Identifier ';';

methodDeclaration	
:	'public' type Identifier '(' parameterList? ')' '{' methodBody '}';

parameterList
:   parameter (',' parameter)*
;

parameter
:   type Identifier
;

methodBody
:	localDeclaration* statement* RETURN expression ';'
;

type	
:	'int' '[' ']'
|	'boolean'
|	'int'
|	Identifier
;	

statement	
:	'{' statement* '}'
#nestedStatement
|	'if' LP expression RP ifBlock 'else' elseBlock
#ifElseStatement
|	'while' LP expression RP whileBlock
#whileStatement
|	'System.out.println' LP  expression RP ';'
#printStatement
|	Identifier EQ expression ';'
#variableAssignmentStatement
|	Identifier LSB expression RSB EQ expression ';'
#arrayAssignmentStatement
;	

ifBlock
:	statement
;

elseBlock
:	statement
;

whileBlock
:	statement
;

expression
:   expression LSB expression RSB
# arrayAccessExpression

|   expression DOTLENGTH
# arrayLengthExpression

|   expression '.' Identifier '(' ( expression ( ',' expression )* )? ')'
# methodCallExpression

|   NOT expression
# notExpression

|   'new' 'int' LSB expression RSB
# arrayInstantiationExpression

|   'new' Identifier '(' ')'
# objectInstantiationExpression

|	expression POWER expression
# powExpression

|   expression TIMES expression
# mulExpression

|   expression PLUS expression
# addExpression

|   expression MINUS expression
# subExpression

|   expression LT expression
# ltExpression  

|   expression AND expression
# andExpression

|   IntegerLiteral
# intLitExpression

|   BooleanLiteral
# booleanLitExpression

|   Identifier
# identifierExpression

|   'this'
# thisExpression

|   '(' expression ')'
# parenExpression
;

AND:'&&';
LT:'<';
PLUS:'+';
MINUS:'-';
TIMES:'*';
POWER:'**';
NOT:'!';
LSB:'[';
RSB:']';
DOTLENGTH:'.length';
LP:'(';
RP:')';
RETURN: 'return';
EQ: '=';

BooleanLiteral
:	'true'
|	'false'
;

Identifier
:	JavaLetter JavaLetterOrDigit*
;

fragment
JavaLetter
:	[a-zA-Z$_] // these are the 'java letters' below 0xFF
;

fragment
JavaLetterOrDigit
:	[a-zA-Z0-9$_] // these are the 'java letters or digits' below 0xFF
;

IntegerLiteral
:	DecimalIntegerLiteral
;

fragment
DecimalIntegerLiteral
:	DecimalNumeral IntegertypeSuffix?
;

fragment
IntegertypeSuffix
:	[lL]
;

fragment
DecimalNumeral
	:	'0'
|	NonZeroDigit (Digits? | Underscores Digits)
	;

	fragment
	Digits
	:	Digit (DigitsAndUnderscores? Digit)?
	;

	fragment
	Digit
	:	'0'
	|	NonZeroDigit
	;

	fragment
	NonZeroDigit
	:	[1-9]
	;

	fragment
	DigitsAndUnderscores
	:	DigitOrUnderscore+
	;

	fragment
	DigitOrUnderscore
	:	Digit
	|	'_'
	;

	fragment
	Underscores
	:	'_'+
	;

	WS
	:   [ \r\t\n]+ -> skip
	;   

	MULTILINE_COMMENT
	:  '/*' .*? '*/' -> skip
	;
	LINE_COMMENT
	:  '//' .*? '\n' -> skip
	;


```


## 根据文法生成代码

1. 下载 [Antlr4](http://www.antlr.org/download/antlr-4.7.1-complete.jar)

2. 设置 antlr4 的系统环境变量 `classpath` 和 `path`

![path](path.png)

![classpath](classpath.png)

3. 新建运行脚本 `antlr4.bat` 和 `grun.bat`，放置于任意目录，如 `d:/tools/antlr4`


```
# antlr4.bat
java org.antlr.v4.Tool %*
```

```
# grun.bat
java org.antlr.v4.gui.TestRig %*
```
   
4. 根据 g4 文件生成代码
   ```
   java org.antlr.v4.Tool -listener -visitor TinyJava.g4
   ``` 
   ![tinylang](tinylang.png)


