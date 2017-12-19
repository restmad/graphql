# B. Appendix: Grammar Summary

SourceCharacter :: /[\u0009\u000A\u000D\u0020-\uFFFF]/


## Ignored Tokens

Ignored ::
  - UnicodeBOM
  - WhiteSpace
  - LineTerminator
  - Comment
  - Comma

UnicodeBOM :: "Byte Order Mark (U+FEFF)"

WhiteSpace ::
  - "Horizontal Tab (U+0009)"
  - "Space (U+0020)"

LineTerminator ::
  - "New Line (U+000A)"
  - "Carriage Return (U+000D)" [ lookahead ! "New Line (U+000A)" ]
  - "Carriage Return (U+000D)" "New Line (U+000A)"

Comment :: `#` CommentChar*

CommentChar :: SourceCharacter but not LineTerminator

Comma :: ,


## Lexical Tokens

Token ::
  - Punctuator
  - Name
  - IntValue
  - FloatValue
  - StringValue

Punctuator :: one of ! $ ( ) ... : = @ [ ] { | }

Name :: /[_A-Za-z][_0-9A-Za-z]*/

IntValue :: IntegerPart

IntegerPart ::
  - NegativeSign? 0
  - NegativeSign? NonZeroDigit Digit*

NegativeSign :: -

Digit :: one of 0 1 2 3 4 5 6 7 8 9

NonZeroDigit :: Digit but not `0`

FloatValue ::
  - IntegerPart FractionalPart
  - IntegerPart ExponentPart
  - IntegerPart FractionalPart ExponentPart

FractionalPart :: . Digit+

ExponentPart :: ExponentIndicator Sign? Digit+

ExponentIndicator :: one of `e` `E`

Sign :: one of + -

StringValue ::
  - `"` StringCharacter* `"`
  - `"""` BlockStringCharacter* `"""`

StringCharacter ::
  - SourceCharacter but not `"` or \ or LineTerminator
  - \u EscapedUnicode
  - \ EscapedCharacter

EscapedUnicode :: /[0-9A-Fa-f]{4}/

EscapedCharacter :: one of `"` \ `/` b f n r t

BlockStringCharacter ::
  - SourceCharacter but not `"""` or `\"""`
  - `\"""`

Note: Block string values are interpretted to exclude blank initial and trailing
lines and uniform indentation with {BlockStringValue()}.


## Document

Document : Definition+

Definition :
  - OperationDefinition
  - FragmentDefinition
  - TypeSystemDefinition

OperationDefinition :
  - SelectionSet
  - OperationType Name? VariableDefinitions? Directives? SelectionSet

OperationType : one of query mutation subscription

SelectionSet : { Selection+ }

Selection :
  - Field
  - FragmentSpread
  - InlineFragment

Field : Alias? Name Arguments? Directives? SelectionSet?

Alias : Name :

Arguments : ( Argument+ )

Argument : Name : Value

FragmentSpread : ... FragmentName Directives?

InlineFragment : ... TypeCondition? Directives? SelectionSet

FragmentDefinition : fragment FragmentName TypeCondition Directives? SelectionSet

FragmentName : Name but not `on`

TypeCondition : on NamedType

Value[Const] :
  - [~Const] Variable
  - IntValue
  - FloatValue
  - StringValue
  - BooleanValue
  - NullValue
  - EnumValue
  - ListValue[?Const]
  - ObjectValue[?Const]

BooleanValue : one of `true` `false`

NullValue : `null`

EnumValue : Name but not `true`, `false` or `null`

ListValue[Const] :
  - [ ]
  - [ Value[?Const]+ ]

ObjectValue[Const] :
  - { }
  - { ObjectField[?Const]+ }

ObjectField[Const] : Name : Value[?Const]

VariableDefinitions : ( VariableDefinition+ )

VariableDefinition : Variable : Type DefaultValue?

Variable : $ Name

DefaultValue : = Value[Const]

Type :
  - NamedType
  - ListType
  - NonNullType

NamedType : Name

ListType : [ Type ]

NonNullType :
  - NamedType !
  - ListType !

Directives : Directive+

Directive : @ Name Arguments?

TypeSystemDefinition :
  - SchemaDefinition
  - TypeDefinition
  - DirectiveDefinition
  - TypeExtension

SchemaDefinition : schema Directives? { OperationTypeDefinition+ }

OperationTypeDefinition : OperationType : NamedType

TypeDefinition :
  - ScalarTypeDefinition
  - ObjectTypeDefinition
  - InterfaceTypeDefinition
  - UnionTypeDefinition
  - EnumTypeDefinition
  - InputObjectTypeDefinition

ScalarTypeDefinition : scalar Name Directives?

ObjectTypeDefinition : type Name ImplementsInterfaces? Directives? FieldDefinitions

ImplementsInterfaces : implements NamedType+

FieldDefinitions : { FieldDefinition+ }

FieldDefinition : Name ArgumentsDefinition? : Type Directives?

ArgumentsDefinition : ( InputValueDefinition+ )

InputValueDefinition : Name : Type DefaultValue? Directives?

InterfaceTypeDefinition : interface Name Directives? FieldDefinitions

UnionTypeDefinition : union Name Directives? = UnionMembers

UnionMembers :
  - NamedType
  - UnionMembers | NamedType

EnumTypeDefinition : enum Name Directives? { EnumValueDefinition+ }

EnumValueDefinition : EnumValue Directives?

EnumValue : Name

InputObjectTypeDefinition : input Name Directives? { InputValueDefinition+ }

DirectiveDefinition : directive @ Name ArgumentsDefinition? on DirectiveLocations

DirectiveLocations :
  - Name
  - DirectiveLocations | Name

TypeExtension : extend ObjectTypeExtension

ObjectTypeExtension : type Name ImplementsInterfaces? Directives? FieldDefinitions?
