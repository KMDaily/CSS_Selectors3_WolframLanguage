# CSS Selectors 3 in the Wolfram Language
Implementation of CSS Selectors 3 for Wolfram Language SymbolicXML expressions.

## Testing on HTML source
Being a little meta, let's test this against the [WC3 page for Selectors Level 3](https://www.w3.org/TR/selectors-3/).

```
In[] := document = Import["https://www.w3.org/TR/selectors-3/", "XMLObject"];
```

### Look for elements that belong to classes that contain the letter 'h'.
```
In[] := Position[document, Selector["[class*=h]"]]
Out[] = {{2, 3, 2, 3, 2}, {2, 3, 2, 3, 484}, {2, 3, 2, 3, 488}, {2, 3, 2, 3, 2, 3, 11}}

In[] := Extract[document, %][[All, 1 ;; 2]] // Column
Out[] = {
 {XMLElement["div", {"class" -> "head"}]},
 {XMLElement["dl", {"class" -> "bibliography"}]},
 {XMLElement["dl", {"class" -> "bibliography"}]},
 {XMLElement["p", {"class" -> "copyright"}]}
}
```

### Look for elements of class '.no-num'
```
In[] := Extract[document, Position[document, Selector[".no-num"]]] // Column
Out[] = {
 {XMLElement["h2", {"class" -> "no-num no-toc", "id" -> "abstract"}, {"Abstract"}]},
 {XMLElement["h2", {"class" -> "no-num no-toc", "id" -> "status"}, {"Status of this document"}]},
 {XMLElement["h2", {"class" -> "no-num no-toc", "id" -> "contents"}, {"Table of contents"}]},
 {XMLElement["h2", {"class" -> "no-num no-toc"}, {"W3C Recommendation 06 November 2018"}]}
}
```

### Check specificity of the selector
```
In[] := Selector[document, "[class~=a] b > *:link"]["Specificity"]
Out[] = {0, 2, 1}

In[] := Selector[document, "[class~=a] b > :not(p)"]["Specificity"]
Out[] = {0, 1, 2}

In[] := Selector[document, "#welcome"]["Specificity"]
Out[] = {1, 0, 0}
```

## Testing on XML source
```
In[] := str = "<html xml:lang='zh'><head><title>Test</title></head><body \
        xmlns='http://www.w3.org/1999/xhtml'><p lang='en' class='red' \
        myid='unique'>Here is some math.</p><p><m:math \
        xmlns:m='http://www.w3.org/1998/Math/MathML'><m:mi \
        m:title='cat'>x</m:mi><m:mo>+</m:mo><m:mn>1</m:mn></m:math></p></body>\
        \n</html>";
     
In[] := obj = ImportString[str, "XML"];
```

### Namespace
If the selector does not specify a namespace, then the namespace is ignored:
```
In[] := Selector[str, "mo"]
Out[] = <|"Specificity" -> {0, 0, 1}, "Elements" -> {{2, 3, 2, 3, 2, 3, 1, 3, 2}}|>
```
If a namespace is given in the selector, then you need to provide the prefix's expansion rule. Otherwise the selector won't match any element.
```
In[] := Selector[str, "m|mo"]
Out[] = <|"Specificity" -> {0, 0, 1}, "Elements" -> {}|>

In[] := Selector[str, "m|mo", "Namespaces" -> {"m" -> "http://www.w3.org/1998/Math/MathML"}]
Out[] = <|"Specificity" -> {0, 0, 1}, "Elements" -> {{2, 3, 2, 3, 2, 3, 1, 3, 2}}|>
```

### ID
XML can define its own unique ID tags. Use the "ID" option to indicate what tag name is in use. This is equivalent to using the attribute selector but with higher specificity.
```
In[] := Selector[str, "#unique", "ID" -> "myid"]
Out[] = <|"Specificity" -> {1, 0, 0}, "Elements" -> {{2, 3, 2, 3, 1}}|>

In[] := Selector[str, "[myid=unique]"]
Out[] = <|"Specificity" -> {0, 1, 0}, "Elements" -> {{2, 3, 2, 3, 1}}|>
```

### Case sensitivity
XML is case-sensitive, but the Selectors3 package is not by default. Use the "CaseInsensitive" option to enforce case sensitivity.
```
In[] := Selector[str, "[myID=unique]", "CaseInsensitive" -> True]
Out[] = <|"Specificity" -> {0, 1, 0}, "Elements" -> {{2, 3, 2, 3, 1}}|>

In[] := Selector[str, "[myID=unique]", "CaseInsensitive" -> False]
Out[] = <|"Specificity" -> {0, 1, 0}, "Elements" -> {}|>
```
You can specify the case-sensitivity separately for attribute name and value.
```
In[] := Selector[str, "[myID=Unique]", "CaseInsensitive" -> {"AttributeName" -> True, "AttributeValue" -> False}]
Out[] = <|"Specificity" -> {0, 1, 0}, "Elements" -> {}|>

In[] := Selector[str, "[myID=Unique]", "CaseInsensitive" -> {"AttributeName" -> False, "AttributeValue" -> True}]
Out[] = <|"Specificity" -> {0, 1, 0}, "Elements" -> {}|>

In[] := Selector[str, "[myID=Unique]", "CaseInsensitive" -> {"AttributeName" -> True, "AttributeValue" -> True}]
Out[] = <|"Specificity" -> {0, 1, 0}, "Elements" -> {{2, 3, 2, 3, 1}}|>
```
You can specify the case-sensitivity separately for type.
```
In[] := Selector[str, "P", "CaseInsensitive" -> {"Type" -> True}]
Out[] = <|"Specificity" -> {0, 0, 1}, "Elements" -> {{2, 3, 2, 3, 1}, {2, 3, 2, 3, 2}}|>

In[] := Selector[str, "P", "CaseInsensitive" -> {"Type" -> False}]
Out[] = <|"Specificity" -> {0, 0, 1}, "Elements" -> {}|>
```
