# Forms
Forms adapter for InterSystems Cache.

# Installation

Import classes and create web app with `Form.REST.Main` broker.

# Metadata generation

Form class compilation triggers metadata gereration for compiled form.
To validate and regenerate metadata for all forms execute this code in a terminal:

```
Do ##class(Form.Util.Init).validateMetadata()
Do $System.OBJ.Compile("Form.Adaptor","cb")
```


# Requests


| URL                                  | Type   | Description                            |
|--------------------------------------|--------|----------------------------------------|
| test                                 | GET    | Test request                           |
| logout                               | GET    | End current session                    |
| form/info                            | GET    | List of all availible forms            |
| form/info/:class                     | GET    | Form metainformation                   |
| form/field/:class                    | POST   | Add field to form                      |
| form/field/:class                    | PUT    | Modify form field                      |
| form/field/:class/:property          | DELETE | Delete  form field                     |
| form/object/:class/:id               | GET    | Retrieve form object                   |
| form/object/:class                   | POST   | Create form object                     |
| form/object/:class/:id               | PUT    | Update form object from dynamic object |
| form/object/:class                   | PUT    | Update form object from object         |
| form/object/:class/:id               | DELETE | Delete form object                     |
| form/objects/:class/:query           | GET    | (SQL) Get all members for the form by query|
| form/file/:class/:id/:property       | POST   | Add files to this property             |
| form/file/:class/:id/:property       | DELETE | Delete all files from this property    |
| form/file/:class/:id/:property/:name | DELETE | Delete one file from property          |
| form/file/:class/:id/:property/:name | GET    | Download one file from property        |

For POST/PUT requests see method descriptions for request body samples

# SQL requests

`GET http://localhost:57772/forms/form/objects/Form.Test.Simple/info?size=2&page=1&orderby=text`

`GET http://localhost:57772/forms/form/objects/Form.Test.Simple/all?orderby=text+desc`

Note, that for SQL access user must have relevant SQL privileges (SELECT on form table).

## Query types

It's a second REST (not URL) parameter in `form/objects/:class/:query` request, it determines the contents of query between SELECT and FROM.
Currently new query types can be specified as parameters in `Form.REST.Objects` class. 

| Query    |  Description         |
|----------|----------------------|
| all      | all information      |
| info     | displayName and id   |
| infoclass| displayName, id, class|

You can define your own class with queries. To define your own query named `myq`:
  1. Define a class 
  2. Define there a `MYQ` parameter or `queryMYQ` class method. Parameter takes precedence over the method. 
  3. Method or param must return the part of SQL query between SELECT and FROM
  4. Execute in a terminal: `Do ##class(For.Settings).setSetting("queryclass", YourClassName)`

Method signature is: `ClassMethod queryMYQ(class As %String) As %String` 

You can define a class-specific query. To define your own class query named `myq`:
  1. Define a `queryMYQ` class method in your form class
  2. Method signature is: `ClassMethod queryMYQ() As %String` 
  3. Method must return the part of SQL query between SELECT and FROM

RESTForms looks for a query named  `myq` in the following paths (till first hit):
  1.  Class method `queryMYQ` in your form class
  2.  Parameter `MYQ` in your query class
  3.  Class method `queryMYQ` in your query class
  4.  Parameter `MYQ` in `Form.REST.Objects` class
  5.  Class method `queryMYQ` in `Form.REST.Objects` class

## URL arguments:

| Argument | Sample Value       | Description     |
|----------|--------------------|-----------------|
| size     | 2                  | page size       |
| page     | 1                  | page number     |
| filter   | Value+contains+W   | WHERE clause    |
| orderby  | Value+desc         | ORDER BY clause |

## ORDER BY clause

Value can be: `Column` or `Column+desc`. Column is a column from the sql table or a colum number.

## WHERE clause 

In a format: `Column+condition+Value`. 

Several conditions are possible: `Column+condition+Value+Column2+condition2+Value2`.

Conditions:

| URL            | SQL         |
|----------------|-------------|
| neq            | !=          |
| eq             | =           |
| gte            | >=          |
| gt             | >           |
| lte            | <=          |
| lt             | <           |
| startswith     | %STARTSWITH |
| contains       | [           |
| doesnotcontain | '[           |


#Samples

See `Form.Test.Simple` and other forms in `Form.Test` package for samples. 
To remove test forms permanently from your local repository 
  1. Enter `Form\Test` directory from git bash
  2. `git update-index --assume-unchanged $(git ls-files | tr '\n' ' ')`
  3. Delete `Form\Test` directory

# Object creation

For `Form.TestForm`.

`POST http://localhost:57772/forms/form/object/Form.Test.Simple`
Headers must contain `Content-Type` and (probably) authorization

```
Content-Type: application/json
Authorization: Basic Base64String
```  

Body:
```
{
        "_class":"Form.Test.Simple",
        "text":3
}
```

# Object update

For `Form.TestForm`.

`PUT http://localhost:57772/forms/form/object/Form.Test.Simple`

Body:
```
{
        "_class":"Form.Test.Simple",
        "_id":3,
        "text":4444
}
```

