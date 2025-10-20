---
parser: v2
auto_validation: true
time: 10
tags: [ tutorial>beginner, software-product-function>sap-hana-cloud--data-lake, software-product>sap-hana-cloud, programming-tool>node-js]
primary_tag: software-product-function>sap-hana-cloud--data-lake
---

# Connect to Data Lake Relational Engine Using the Node.js Driver
<!-- description --> Create and debug a Node.js application that connects to data lake Relational Engine.

## Prerequisites
 - You have completed the first tutorial in this group.

## You will learn
  - How to install Node.js and the data lake Relational Engine Node.js driver
  - How to create and debug a Node.js application
  - How to use both the synchronous and asynchronous driver interfaces

## Intro
Node.js provides a JavaScript runtime outside of the browser and uses an asynchronous event driven programming model.  For more details, see [Introduction to Node.js](https://nodejs.org/en/learn/getting-started/introduction-to-nodejs).  

---

### Install Node.js
Ensure you have Node.js installed and check its version. Enter the following command:

```Shell
node -v  
```  

If Node.js is installed, the currently installed version is returned, such as v22.16.0.

If Node.js is not installed, download the long-term support (LTS) version of the Node.js installer from [Download Node.js](https://nodejs.org/en/download/).

>If an install for Node.js is not provided on Linux, you may choose to install it via a package manager. For more details, please navigate to [this link](https://nodejs.org/en/download/package-manager/).

---

>During the installation, there is no need to install Chocolatey.  
>
>![Chocolatey](Chocolatey.png)


### Install the data lake Relational Engine client for Node.js
The Node.js driver covered in this tutorial is [@sap\iq-client](https://www.npmjs.com/package/@sap/iq-client) which supports the latest Node.js versions and includes a promise library.  An alternate driver is the [SQL Anywhere](https://github.com/sqlanywhere/node-sqlanywhere) driver.  

1. Open a new Shell and create a folder named `node` and enter the newly created directory.

    ```Shell (Microsoft Windows)
    mkdir %HOMEPATH%\DataLakeClientsTutorial\node
    cd %HOMEPATH%\DataLakeClientsTutorial\node
    ```

    ```Shell (Linux)
    mkdir $HOME/DataLakeClientsTutorial/node
    cd $HOME/DataLakeClientsTutorial/node
    ```

2. Initialize the project and install the `@sap\iq-client` driver from npm.

    ```Shell 
    npm init -y
    npm install @sap/iq-client
    ```

3. The following command lists the Node.js modules that are now installed locally into the `DataLakeClientsTutorial\node` folder.

    ```Shell
    npm list
    ```

    ![npm list](npm-list.png)


### Create a synchronous Node.js application that queries SAP data lake Relational Engine
1. Create a new file named `nodeQuery.js` in an editor.

    ```Shell (Microsoft Windows)
    notepad nodeQuery.js
    ```

    Substitute `pico` below for your preferred text editor.  

    ```Shell (Linux or Mac)
    pico nodeQuery.js
    ```

2. Add the code below to `nodeQuery.js` and update the **host** variable.

    ```JavaScript
    'use strict';
    const { PerformanceObserver, performance } = require('perf_hooks');
    var t0;
    var util = require('util');
    var datalakeRE = require('@sap/iq-client');

    var connOptions = {
        host: 'XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXX.iq.hdl.trial-XXXX.hanacloud.ondemand.com:443',
        uID: 'USER1',
        pwd: 'Password1',
        enc: 'TLS{tls_type=rsa;direct=yes}',
    };

    //Synchronous example querying a table
    var connection = datalakeRE.createConnection();
    connection.connect(connOptions);

    var sql = 'select TITLE, FIRSTNAME, NAME from HOTELS.CUSTOMER;';
    t0 = performance.now();
    var result = connection.exec(sql);
    console.log(util.inspect(result, { colors: false }));
    var t1 = performance.now();
    console.log("time in ms " +  (t1 - t0));
    connection.disconnect();
    ```  

4. Run the app.  

    ```Shell
    node nodeQuery.js
    ```

    ![Running nodeQuery.js](node-query-sync.png)

    If an error appears such as Error: `libdbcapi_r.so` is missing, its location can be specified using an environment variable such as IQ_DBCAPI_DIR.

    Note the above app makes use of some of the data lake Relational Engine client Node.js driver methods, such as [connect](https://help.sap.com/docs/hana-cloud-data-lake/developer-guide-for-data-lake-relational-engine/connect-string-object-function-method), [exec](https://help.sap.com/docs/hana-cloud-data-lake/developer-guide-for-data-lake-relational-engine/exec-ute-string-array-object-function-method) and [disconnect](https://help.sap.com/docs/hana-cloud-data-lake/developer-guide-for-data-lake-relational-engine/disconnect-close-end-function-method).

    Two examples showing the drivers methods being used asynchronously are shown in the next two steps.

    >To enable debug logging of the Node.js client, enter the following command and then rerun the app.

    >```Shell (Microsoft Windows)
    >set DEBUG=*
    >node nodeQuery.js
    >```  

    Linux or Mac
    
    >```Shell
    >export DEBUG=*
    >node nodeQuery.js
    >```  

    > ![debug output](debug-flag.png)  

    > The value of the environment variable DEBUG can be seen and removed with the commands below.  

    >```Shell (Microsoft Windows)
    >set DEBUG
    >set DEBUG=
    >set DEBUG
    >```   

    Linux or Mac

    >```Shell (Linux)
    >printenv | grep DEBUG
    >unset DEBUG
    >printenv | grep DEBUG
    >```  

### Create an asynchronous app that uses callbacks
Asynchronous programming enables non-blocking code execution which is demonstrated in the below example.

1. Open a file named `nodeQueryCallback.js` in an editor.

    ```Shell (Microsoft Windows)
    notepad nodeQueryCallback.js
    ```

    Substitute `pico` below for your preferred text editor.  

    ```Shell (Linux)
    pico nodeQueryCallback.js
    ```

2. Add the code below to `nodeQueryCallback.js` and update the host variable.

    ```JavaScript
    'use strict';
    const { PerformanceObserver, performance } = require('perf_hooks');
    var t0;
    var util = require('util');
    var datalakeRE = require('@sap/iq-client');

    var connOptions = {
        host: 'XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXX.iq.hdl.trial-XXXX.hanacloud.ondemand.com:443',
        uID: 'USER1',
        pwd: 'Password1',
        enc: 'TLS{tls_type=rsa;direct=yes}',
    };

    //Asynchronous example calling a stored procedure with callbacks
    var connection = datalakeRE.createConnection();

    connection.connect(connOptions, function(err) {
        if (err) {
            return console.error(err);
        }
        //Prepared statement example
        const statement = connection.prepare('CALL HOTELS.SHOW_RESERVATIONS(?,?)');
        const parameters = [11, '2020-12-24'];
        var results = statement.execQuery(parameters, function(err, results) {
            if (err) {
                return console.error(err);
            }
            processResults(results, function(err) {
                if (err) {
                    return console.error(err);
                }
                results.close(function(err) {
                    if (err) {
                        return console.error(err);
                    }
                    statement.drop(function(err) {
                        if (err) {
                            return console.error(err);
                        }
                        return connection.disconnect(function(err) {
                            if (err) {
                                return console.error(err);
                            }
                        });
                    });
                });
            });
        });
    });

    function processResults(results, cb) {
        results.next(function (err, hasValues) {
            if (err) {
                return console.error(err);
            }
            if (hasValues) {
                results.getValues(function (err, row) {
                    console.log(util.inspect(row, { colors: false }));
                    processResults(results, cb);
                });
            }
            else {
                return cb();
            }
        });
    }
    ```  

3. Run the app.  

    ```Shell
    node nodeQueryCallback.js
    ```
    ![Running nodeQueryCallback.js](Node-query-callback.png)

    Notice that asynchronous method calls use callback functions.


### Create an asynchronous app that uses promises
The Node.js driver for the data lake Relational Engine client provides support for promises.  The following example demonstrates this.  Notice that there is less nesting of code then the previous example.

1. Open a file named `nodeQueryPromise.js` in an editor.

    ```Shell (Microsoft Windows)
    notepad nodeQueryPromise.js
    ```

    Substitute `pico` below for your preferred text editor.  

    ```Shell (Linux or Mac)
    pico nodeQueryPromise.js
    ```

2. Add the code below to `nodeQueryPromise.js`.  

    ```JavaScript
    'use strict';
    const { PerformanceObserver, performance } = require('perf_hooks');
    var t0;
    var util = require('util');
    var datalakeRE = require('@sap/iq-client');
    var PromiseModule = require('@sap/iq-client/extension/Promise.js');

    var connOptions = {
        //Specify the connection parameters
        host: 'XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXX.iq.hdl.trial-XXXX.hanacloud.ondemand.com:443',
        uid: 'USER1',
        pwd: 'Password1',
        enc: 'TLS{tls_type=rsa;direct=yes}',
    };

    //Asynchronous example calling a stored procedure that uses the promise module
    var connection = datalakeRE.createConnection();
    var statement;

    PromiseModule.connect(connection, connOptions)
        .then(() => {
             //Prepared statement example
             return PromiseModule.prepare(connection, 'CALL HOTELS.SHOW_RESERVATIONS(?,?)');
        })
        .then((stmt) => {
            statement = stmt;
            const parameters = [11, '2020-12-24'];
            return PromiseModule.executeQuery(stmt, parameters);
        })
        .then((results) => {
            return processResults(results);
        })
        .then((results) => {
            return PromiseModule.close(results);
        })
        .then(() => {
            return PromiseModule.drop(statement);
        })
        .then(() => {
            PromiseModule.disconnect(connection);
        })
        .catch(err =>  {
            console.error(err);
        });

    function processResults(results) {
        return new Promise((resolve, reject) => {
        var done = false;
            PromiseModule.next(results)
                .then((hasValues) => {
                    if (hasValues) {
                        return PromiseModule.getValues(results);
                    }
                    else {
                        done = true;
                    }
                })
                .then((values) => {
                    if (done) {
                        resolve(results);
                    }
                    else {
                        console.log(util.inspect(values, { colors: false }));
                        processResults(results)
                        .then((results) => {
                            resolve(results);
                        });
                    }
                })
                .catch (err => {
                    reject(err);
                });
        })
    }    
    ```  

4. Run the app.  

    ```Shell
    node nodeQueryPromise.js
    ```
    ![Running nodeQueryPromise.js](Node-query-promise.png)

    The above code makes use of the [promise module](https://help.sap.com/docs/hana-cloud-data-lake/developer-guide-for-data-lake-relational-engine/promise-module).  Additional details on promises can be found at [Using Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises).


### Debug the application
Visual Studio Code can run and debug a Node.js application.  It is a lightweight but powerful source code editor which is available on Windows, macOS and Linux. Ensure that Node.js is added to your path in environment variables such as 	C:\Program Files\nodejs.

1. If required, download [Visual Studio Code.](https://code.visualstudio.com/Download).

2. In Visual Studio Code, choose **File | Add Folder to Workspace** and then add the `DataLakeClientsTutorial` folder.

    ![Workspace](workspace.png)

3. Open the file `nodeQuery.js`.

4. Place a breakpoint inside the `connection.exec` callback.  Select **Run | Start Debugging | Node.js**.  

    Notice that the debug view becomes active.  

    Notice that the program stops running at the breakpoint that was set. Observe the variable values in the leftmost pane.  Step through code.

    ![VS Code Debugging](debugging.png)

    If "Can't find Node.js binary 'node': path does not exist" error pops up, open a Shell and run the following command.
    ```Shell
    code .
    ```
    Then restart VSCode.


### Knowledge check
Congratulations! You have created and debugged a Node.js application that connects to and queries an SAP data lake Relational Engine database.

---
