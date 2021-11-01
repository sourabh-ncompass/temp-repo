**API Development Best Practices**

### Table of Contents

- [Naming Conventions](#naming-conventions)
- [Response](#response)
- [Validation](#validation)
- [Configuration And Environment Variables](#configuration-and-environment-variables)
- [Modularization](#modularization)
- [Code Formatting](#code-formatting)
- [SQL Queries](#sql-queries)

## Naming Conventions

- Important points to be considered while naming a variable, class or methods.

  - Use nouns for Class and type names.
  - Names for methods should contain a verb.
  - If a method returns a value indicating whether something is true or not, then start the method name with “is”.
  - If a method returns an object’s property, then start the name with “get”.
  - If a method sets a property, then start with “set”.
  - Avoid use of single identifiers for multiple purposes.
  - Use of descriptive names that defines the usability itself.

<a name="naming-conventions--foldersAndFiles"></a><a name="1.1"></a>

- [1.1](#naming-conventions--foldersAndFiles) **Folders And Files:** Use Spinal Case or Kebab Case for naming Folders and Files.

  - Bad:

  ```
  errorhandler.js;
  ```

  - Good:

  ```
  error-handler.js;
  ```

<a name="naming-conventions--classAndDataTypes"></a><a name="1.2"></a>

- [1.2](#naming-conventions--classAndDataTypes) **Class and Custom Data Types:** Use Pascal Case for naming Classes and Custom Data Types.

  - Bad:

  ```javascript
  class cutomerrorclass {}
  ```

  - Good:

  ```javascript
  class CustomErrorClass {}
  ```

  <a name="naming-conventions--variablesAndFunctions"></a><a name="1.3"></a>

- [1.3](#naming-conventions--variablesAndFunctions) **Variables and Functions:** Use Camel Case for naming variables and methods.

  - Bad:

  ```javascript
  const a = (n) => {
    const result = n * n;
    return result;
  };
  ```

  - Good:

  ```javascript
  const calculateSquare = (num) => {
    const squareResult = num * num;
    return squareResult;
  };
  ```

## Response

- The response should have appropriate _HTTP Status_ along with the body with _success_, _message_, and _data_ parameters. The response _headers_ should include the appropriate _Content-Encoding_ and _Content-Type_.

<a name="response--valid"></a><a name="2.1"></a>

- [2.1](#response--valid) **Valid Response Structure:**

  - Sample Response:

  ```
  Status: 2XX
  Headers:
    Content-Type: application/json
    Content-Encoding: gzip
  Body:
    {
      "success": true,
      "message": "",
      "data": {}
    }
  ```

<a name="response--error"></a><a name="2.2"></a>

- [2.2](#response--error) **Error Response Structure:**

  - Sample Error Response:

  ```
  Status: XXX         // 4XX or 5XX
  Headers:
    Content-Type: application/json
    Content-Encoding: gzip
  Body:
    {
      "success": false,
      "message": "",  // error message
      "data": {}      // error data
    }
  ```

## Validation

- Important key points -

  - All Incoming Request Params must be validated.

<a name="validation--functions"></a><a name="3.1"></a>

- [3.1](#validation--functions) **Validation Functions:** The validation functions should return an object containing following keys: _isValid, message_. Here, _isValid_ is a boolean value that is _true_ if validation is successful and _false_ if it is unsuccessful whereas _message_ will contain the _error message_ in case of _unsuccessful_ validation.

  - Sample Code:

  ```javascript
  const validationForExport = (body) => {
    let validationResponse = {
      isValid: true,
      message: "",
    };

    const schema = joi.object({
      startDate: joi.date().iso().required(),
    });

    const validationResult = schema.validate(body);

    if (validationResult.error) {
      validationResponse.isValid = false;
      validationResponse.message = validationResult.error.details[0].message;
    }

    return validationResponse;
  };
  ```

## Configuration And Environment Variables

- Important key points to be considered while dealing with sensitive configuration file or credentials and environment variables -

  - _Do not hardcode_ any _config related parameters in code_. Always have a _config file_ and use data from the file.
  - _Do not share_ or push your _config or credential key_ related files in Github repository or any other repository management system.

<a name="configuration-and-environment-variables--handling"></a><a name="4.1"></a>

- [4.1](#configuration-and-environment-variables--handling) **Environment Variables and Configuration Files:** Do not hardcode any security credentials. Always use a Configuration file or set Environment Variables for the sensitive credentials and configurations.

  - Bad:

  ```javascript
  const dbConfig = {
    connectionLimit: 100,
    host: dev-slave.c1239xjb02812i.us-east-1.rds.amazonaws.com,
    user: admin,
    password: HelloWorld,
    database: merchant,
  };

  // or,

  const dbConfig = {
    connectionLimit: 100,
    host: dev-slave.c1239xjb02812i.us-east-1.rds.amazonaws.com,
    user: admin,
    password: HelloWorld,
    database: merchant,
  };
  ```

  - Good:

  ```javascript
  import "dotenv/config";
  const dbConfig = {
    connectionLimit: process.env.CONNECTION_LIMIT,
    host: process.env.DB_HOST,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_DATABASE,
  };

  // or,

  import * as dbCreds from "./db_credentials.json";
  const dbConfig = {
    connectionLimit: dbCreds.connectionLimit,
    host: dbCreds.host,
    user: dbCreds.user,
    password: dbCreds.password,
    database: dbCreds.database,
  };
  ```

## Modularization

- Important key points to be considered while modularizing the program and functions -

  - _Line of code_ must not exceed more _200 - 250_, if exceeds break into subcomponents.
  - _Functions_ must not exceed _40_ lines. Break function into _modules_ or _sub functions_.
  - _Avoid deep nesting_. Too many nesting levels make code harder to read and follow.
  - _Divide_ the program into several _sub-modules_ where each _sub-module_ contains something necessary to execute only _one aspect of the desired functionality_.

<a name="modularization--modulesAndFunctions"></a><a name="5.1"></a>

- [5.1](#modularization--modulesAndFunctions) **Modules and Modular Functions:**

  - Bad:

    - loan-check.js

    ```javascript
    // Check User Loan Approval Status
    const isLoanApproved = async (userName) => {
      // Check if user is valid or not
      let query = "SELECT COUNT(*) FROM users WHERE username = ?";
      const queryResult = await mysql
        .query(userName, [userName])
        .then((result) => {
          return result;
        })
        .catch((err) => {
          return err;
        });

      if (queryResult instanceof Error) {
        // Code Logic, if error occurred
      } else {
        const isValid = queryResult === 1 ? true : false;

        // If user is valid
        if (isValid) {
          // Get application number of the user
          query =
            "SELECT application_number FROM userDetails WHERE username = ?";
          const userApplicationNumber = await mysql
            .query(query, [userName])
            .then((result) => {
              return result;
            })
            .catch((err) => {
              return err;
            });

          if (userApplicationNumber instanceof Error) {
            // Code Logic, if error occurred
          } else {
            // Check user loan status
            query =
              "SELECT status FROM loanStatus WHERE application_number = ?";
            const loanStatus = await mysql
              .query(query, [userApplicationNumber])
              .then((result) => {
                return result;
              })
              .catch((err) => {
                return err;
              });

            if (loanStatus instanceof Error) {
              // Code Logic, if error occurred
            } else {
              return loanStatus;
            }
          }
        } else {
          // Code Logic, if user is not valid
        }
      }
    };
    ```

  - Good:

    - loan-check.js

    ```javascript
    // Check User Loan Approval Status
    const isLoanApproved = async (userName) => {
      const isValid = isUserValid(userName);

      // If user is not valid
      if (!isValid) {
        return;
      }
      const userApplicationNumber = await getApplicationNumber(userName);
      const loanStatus = isLoanApproved(userApplicationNumber);
      return loanStatus;
    };
    ```

    - user-module.js

    ```javascript
    // Check if user is valid or not
    const isUserValid = async (userName) => {
      let query = "SELECT COUNT(*) FROM users WHERE username = ?";
      const queryResult = await mysql
        .query(userName, [userName])
        .then((result) => {
          return result;
        })
        .catch((err) => {
          // Code Logic, if error occurred
        });
      const isValid = queryResult === 1 ? true : false;
      return isValid;
    };

    // Get application number of the user
    const getApplicationNumber = (userName) => {
      query = "SELECT application_number FROM userDetails WHERE username = ?";
      const userApplicationNumber = await mysql
        .query(query, [userName])
        .then((result) => {
          return result;
        })
        .catch((err) => {
          // Code Logic, if error occurred
        });
      return userApplicationNumber;
    };

    // Check user loan status
    const isLoanApproved = (userApplicationNumber) => {
      query = "SELECT loan_status FROM loanStatus WHERE application_number = ?";
      const loanStatus = await mysql
        .query(query, [userApplicationNumber])
        .then((result) => {
          return result;
        })
        .catch((err) => {
          // Code Logic, if error occurred
        });
      return loanStatus;
    };
    ```

## Code Formatting

- The code always looks better when it is in a proper format. Important key points -

  - Use appropriate single space before and after operators, variables and parameters.
  - Code must well formatted and tab structured according to the respective languages.
  - Plugin like _Prettier_, need to be used for formatting.
  - Use _ESLINT_ for JS applications.

<a name="code-formatting--program"></a><a name="6.1"></a>

- [6.1](#code-formatting--program) **Javascript Code:**

  - Bad:

  ```
  var x=4; var y=2;
  function add(a,b){return a+b;}
  var result;
  result =add(x,y);
  ```

  - Good:

  ```javascript
  var x = 4;
  var y = 2;
  function add(a, b) {
    return a + b;
  }
  var result;
  result = add(x, y);
  ```

## SQL Queries

- Important key points when dealing with _Mysql queries_ -

  - SQL Query should not have _string formating_ or _string bindings_.
  - Use prepared statements in case of _Raw Query_.
  - _Query Builder_ must constructed well to handle query conditions.
  - Avoid running queries within loops.
  - Construct bulk queries and have a single query execution.

<a name="sql-queries--queryConstruction"></a><a name="7.1"></a>

- [7.1](#sql-queries--queryConstruction) **Query Construction:**

  - Bad:

  ```javascript
  const insertStudentRecord = (students) => {
    let query = "INSERT INTO STUDENTS (NAME) VALUES ?";

    // Insert individual record inside loop
    for (let i = 0; i <= students.length; i++) {
      mysql.query(query, [students[i]]);
    }
  };
  ```

  - Good:

  ```javascript
  const insertStudentRecord = (students) => {
    let query = "INSERT INTO STUDENTS (NAME) VALUES ?";
    let studentsQueryParams = [];

    students.forEach((student) => {
      studentsQueryParams.push([student]);
    });

    // Bulk Insert
    mysql.query(query, [studentsQueryParams]);
  };
  ```