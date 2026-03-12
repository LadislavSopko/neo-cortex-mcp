# MBEL v5 Grammar [IMMUTABLE]

§MBEL:5.0
@purpose::AIMemoryEncoding{compression%75,fidelity%100}

## Operators [IMMUTABLE]

### Core Set (27)
```
[Temporal:4]
>   past/did/changed/was
@   present/current/now/is
?   future/todo/will/planned
≈   around/circa/approximate/roughly

[State:4]
✓   complete/done/success/working
✗   failed/broken/error/bug
!   critical/important/priority/alert
⚡  active/urgent/in-progress/hot

[Relation:6]
::  is/defines/equals/becomes
→   leads_to/causes/then/produces
←   comes_from/because/source/from
↔   mutual/bidirectional/syncs/relates
+   and/with/combines/includes
-   remove/delete/without/except

[Structure:5]
[]  section/category/namespace/group
{}  metadata/attributes/properties/details
()  note/comment/aside/remark
|   or/alternative/choice/option
<>  variant/template/placeholder/variable

[Quantification:3]
#   number/count/quantity/amount
%   percentage/ratio/proportion/part
~   approximately/range/between/around

[Logic:3]
&   AND/requires/must-have/with
||  OR/either/can-be/allows
¬   NOT/exclude/prevent/disable

[Meta:2]
©   source/origin/author/credit
§   version/revision/protocol/schema
```

### Grammar Rules [IMMUTABLE]
1. NoArticles(a/the/an)
2. CamelCase→MultiWord
3. ImplicitSubject{contextClear}
4. OperatorsOnly¬Punctuation
5. Newline::StatementSeparator
6. LatestOverridesPrevious
7. LeftToRight→Composition