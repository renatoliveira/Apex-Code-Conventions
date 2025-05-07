# Salesforce Apex Code Conventions

## 1. Class naming

* CamelCase starting from uppercase letter only.
* Singular nouns combinations only.
* Keep classes named using the full name of the entity they primarily affect and postfix with class type.
  Class types:
  * Batch
  * Controller
  * Extension
  * Schedule
  * WebService
  * Exception
  * Interface
  * Test
  * TriggerHandler
* Test class names are the exact copies of tested class names postfixed with “Test”.
* Controller class names are the exact copies of its pages names postfixed with “Controller”.

|Bad :(                     |Good!                       |
|---------------------------|----------------------------|
|HomePageCtrl               |HomePageController          |
|OppTriggerHandler          |OpportunityTriggerHandler   |
|PopulateOrders_Batch       |OrderPopulationBatch        |
|TestAccountsWebServices    |AccountWebServiceTest       |

## 2. Variable naming

* `camelCase` only. For types that represent multiple items, such as sets and lists, use a pluralized version of a name that represents what that variable is being used for. The exception is for maps, which don't necessarily need to be pluralized.
* No abbreviations, except for loop variables. No acronyms.
* Postfix collections with collection type. Use singular noun forms.
* Use “To” between key and value descriptions in map names (unnecessary for standard maps from object primary key Id to object).
* Use clear, meaningful names.

|Bad :(                     |Good!                       |Might be good depending on context|
|---------------------------|----------------------------|---|
|InsertAccount              |accountToInsert             |account1|
|test_opp1                  |firstTestOpportunity        |opportunity1|
|PopulateOrders_Batch       |OrderPopulationBatch        |OrderPopulateJobBatch|
|TestAccountsWebServices    |AccountWebServiceTest       |AccountWSTest|
|opportunitiesToUpdate      |opportunityToUpdateList     |`newOpportunities` or `changedOpportunities`|
|fieldsMapping              |jsonFieldToInternalFieldMap |jsonData|
|uniqueData                 |orderSet                    |uniqueOrderSet|

## 3. Method naming

* CamelCase starting from lowercase letter only.
* Should start with a verb.
* Leave “get” and “set” prefixes for getters and setters only. Use “obtain” or "select" and “specify” or "assign", respectively, instead. Other common verbs might include "filter", "sort", "compare", "start", etc.

## 4. Constants naming

* Constants (i.e. variables defined as `static final`) should be in uppercase with words separated by underscores:

```apex
public static final String PAYPAL_LOGIN_URL = 'https://login.paypal.com/';
```

## 5. SOQL and SOSL queries

* SOQL and SOSL keywords should all be uppercase.
* Fields to retrieve should be on separate lines, with the separating comma on the same line as the field, unless there are just a few fields being retrieved. In that case it is acceptable to have the query on a single line (see convention 7.1, about line length).

```apex
// An acceptable one-liner
List<Contact> contacts = [SELECT Id, Name FROM Contact WHERE AccountId IN :accountIdSet];
```

```apex
// In this example the line takes 89 characters, which is below the limit imposed by convention 7.1 (line length).
List<Contact> contacts = [
    SELECT
    Id,
    FirstName,
    LastName,
    Phone,
    Email,
    CustomField__c,
    AnotherField__c
    FROM Contact
    WHERE Name != NULL
];
```

In this situation the query alone would take more than 100 characters, excluding whitespace and variable declaration. To comply with convention 7.1 we format it to one field and condition per line. This makes it easier to edit which fields and conditions are being used on this query.

## 6. Comments

Methods should have comments if they are publicly available for callers to use (that is: if they are `public` or `global`). The idea here is that for tools that use a language server protocol (LSP) to provide code completion, they are able to pick up the documentation comment from the public method. Private methods CAN have documentation too, depending on the complexity of what they do. It is encouraged to make a comment on what the method does. That is, however, optional. Remember that **someone else** might be maintaining the code. Use Javadoc commenting style (<http://www.oracle.com/technetwork/java/javase/documentation/index-137868.html>) which can then be consumed by ApexDocs (<https://gitlab.com/StevenWCox/sfapexdoc>) to generate documentation easily, but is also picked up by the Salesforce Extensions.

``` java
// Classes should be commented as follows:
/**
 * Description of class.
 * Why it exists, what it does, which module it belongs to.
 */
public class SomeClass {
    // Methods should be commented as follows:
    /**
     * Description of what the method does.
     * @param param1 Description of param1
     * @param param2 Description of param2
     * @return String [output desc]
     * @see AnotherClass
     */
    public static String someFunction(
        String param1,
        String param2
    ) {
        // Method body goes here..
        return '';
    }
}
```

If the method DOES NOT return anything, the @return line shouldn't exist on the comment.

## 7. Indentation

Four spaces should be used as the unit of indentation. The exact construction of the indentation should be the space.

### 7.1 Line Length

Avoid lines longer than 80 characters, for consistency with IDEs and text editors. If a line exceeds that number of characters, it should be broken into two or more lines, depending on the situation (see below).

### 7.2 Wrapping Lines

When an expression will not fit on a single line, break it according to these general principles:

* Break after a comma.
* Break before an operator.
* Prefer higher-level breaks to lower-level breaks.
* Align the new line with the beginning of the expression at the same level on the previous line.
* If the above rules lead to confusing code or to code that is squished up against the right margin, just indent 8 spaces (2 tabs) instead.
* If a method signature is to great, break it so that each parameter goes to its own line.

#### Examples

##### 7.2.1 Method Signatures

```java
private class SomeClass {
    public static void doSomethingReallyComplicated(
        Integer sumOfSomething,
        Decimal anotherSum,
        String nameOfThatThing
    ) {
        Object someObject = SomeOtherClass.getObject();

        // ...
    }
}
```

##### 7.2.2 Instance Declarations

The method line above has 95 characters (counting the 4 whitespaces before `public`), and it applies the first principle of breaking after a comma. Note that by using an extra indentation on the signature on the lower line we avoid the confusion of where the scope actually starts.

```java
public with sharing class SomeClass {
    // ...
    public void someMethod () {
        AnotherClass.AClassChildren bigVariableName
            = new AnotherClass.AClassChildren(this.someVariable);
    }
}
```

The instance declaration on line 3 would use 105 characters, but with the principle of breaing before an operator it will use 65 characters on the lower line.

##### 7.2.3.1 Conditionals - Bail before if possible

When declaring IF statements, start with the conditions that can skip the process, or bail before running unecessary code.

```java
// DON'T DO THIS
if (condition1 && condition2) {
    doSomething02();
}
if (condition1 && condition3) {
    doSomething03();
}
if (!condition1) {
    continue;
}
```

```java
//DO THIS
if (!condition1) {
    continue; // If condition1 is required for the rest of the steps
};

// The code will only get here if condition1 is true:
if (condition2) {
    doSomething02();
}

if (condition3) {
    doSomething03();
}
```

##### 7.2.3.2 Conditionals - Formatting

```java
// DON'T DO THIS
if ((condition1 && condition2)
    || (condition3 && condition4)
    || !(condition5 && condition6)) {
    doSomething();
}
```

```java
//DO THIS
if ((condition1 && condition2)
        || (condition3 && condition4)
        || !(condition5 && condition6)) {
    doSomething();
}
```

Bad wraps like the one on the first example make easy to miss the part where the method's signature ends and when the scope actually begins. Good wraps help to make those clearer, and not so easy to miss.

##### 7.2.4 Ternary Expressions

```java
// if it fits on the 100 character limit:
result = (aLongExpression) ? something : anotherThing;

// if it doesn't fit the 100 character limit use this:
result = (aVeryLongExtremelyLongExpression) ? something
                                            : anotherThing;

// or this:
result = (aVeryLongExtremelyLongExpression)
         ? something
         : anotherThing;

```

As exemplified above, there are three ways to format ternary expressions.

### 7.3 Blank Lines

Use blank lines to separate contexts and lines with different types of code execution.

Empty lines should be added before and after blocks (almost any context of execution that starts and ends with a pair of brackets (`{ ... }`)). They should also be used to separate different types of code execution statements. That means that assignments should be separated from method calls, for example. The exception is when the block is at the beginning of another block (a conditional or loop is the opened right at the beginning of an execution context, as shown in example 7.3.1).

#### Examples

##### 7.3.1 Block Separation

```apex
// BAD example
// notice how there's no space between methods
public static void doSomething() {
    // something
}
public static Integer selectCount() {
    // select logic
    if (condition) {
        return 1;
    } // notice how there's no blank line after the block
    return 0;
}
```

```apex
// GOOD example
public static void doSomething() {
    // something
}

public static Integer selectCount() {
    // select logic ...
    if (condition) {
        return 1;
    }

    return 0;
}
```

##### 7.3.2 Separation By Execution Type

```apex
// BAD example
Id someId = getMyId();
String objectPrefix = someId.getSObjectType().getDescribe().getKeyPrefix(); // this is valid because although it is calling methods through `.getDescribe()`, the final operation is an *assignment*.
myMethod(objectPrefix); // this is not valid because its a method call mixed with variable assignments above
```

```apex
// GOOD example
Id someId = getMyId();
String objectPrefix = someId.getSObjectType().getDescribe().getKeyPrefix(); // this is valid because although it is calling methods through `.getDescribe()`, the final operation is an *assignment*.

myMethod(objectPrefix); // this is valid because because there's a blank line separating the statement types
```

## 8. White Space

Blank lines improve readability by setting off sections of code that are logically related.

Two blank lines should be used in the following:

* Between sections of a source file
* Between class and interface definitions

One blank line should be used in the following:

* Between methods
* Between the local variables in a method and its first statement
* Before a block or a single line comment

### 8.2 Blank Spaces

Blank spaces should be used in the following circumstances:

* A keyword followed by a parenthesis should be separated by a single space.

```java
while(true){ // WRONG
    ...
}

while(true) { // ACCEPTABLE
    ...
}

while (true) { // CORRECT
    ...
}
```

* A blank space should appear after commas in argument lists.

```java
method(Object anObjectArg,Integer anIntegerArg){ // WRONG
    ...
}

method(Object anObjectArg, Integer anIntegerArg) { // CORRECT
    ...
}
```

* Expressions inside a `for` statement:

```java
for (Integer i=0;i<10;i++) { // WRONG
    ...
}

for(Integer i = 0; i < 10; i++) { // ACCEPTABLE
    ...
}

for (Integer i = 0; i < 10; i++) { // CORRECT
    ...
}
```

* Object casts:

```java
ObjectA objInstance=(ObjectA)anotherType; // WRONG

ObjectA objInstance = (ObjectA)anotherType; // ACCEPTABLE

ObjectA objInstance = (ObjectA) anotherType; // CORRECT
```
