---
parser: v2
auto_validation: true
time: 15
tags: [tutorial>beginner, software-product-function>sap-hana-cloud--data-lake, software-product>sap-hana-cloud]
primary_tag: software-product-function>sap-hana-cloud--data-lake
---

# Connect to Data Lake Relational Engine Using the Go Driver
<!-- description --> Create and debug a Go application that connects to data lake Relational Engine. 

## Prerequisites
 - You have completed the first tutorial in this group.

## You will learn
  - How to install Go
  - How to create and debug a Go application that queries a data lake Relational Engine

## Intro  
Go is an open-source programming language developed by Google to increase productivity among programmers. For more information, see the [Go Documentation](https://golang.org/doc/).

---

### Install Go
The first step is to check if Go is installed, and if so, which version. To do so, enter the following command:

```Shell
go version
```

![go version linux](version2.png)

If Go is installed, then it will return the currently installed version, such as 1.23.4

If it is not installed, download it from [Download Go](https://golang.org/dl/), run the installer, follow the provided instructions, and ensure that Go is in your path.

On Linux, follow the instructions for the appropriate Linux version: [Installing Go for openSUSE](https://en.opensuse.org/SDB:Go).

>Note: A new shell window must be opened for the system to recognize the Go installation and for executing any future Go commands. 

### Configure the environment
The data lake Relational Engine Client interface for Go, like the other data lake Relational Engine client interfaces (except JDBC), makes use of a C library named SQLDBC.  

The Go driver loads the SQLDBC library  named `libdbcapiHDB` using [cgo](https://golang.org/cmd/cgo/).  For further information on the following steps, consult [Go (golang) Driver](https://help.sap.com/docs/hana-cloud-data-lake/client-interfaces/go-golang-driver) in the SAP HANA Cloud, Data Lake Client Interfaces Reference Guide. In order to use the Go Driver, a 64-bit `gcc` compiler is required.

1. To check if a 64-bit `gcc` compiler is installed, run the following command:

    ```Shell
    gcc --version
    ```

    On Windows (if needed), download the compiler from [MinGW](https://www.mingw-w64.org/downloads/). Under **WinLibs.com**, follow the link to [winlibs.com](https://winlibs.com/).  Navigate to the **Release versions** section. Find the latest release version of the Zip archive (UCRT runtime) for Win64 – x86_64. Then extract the folder. 

    ![download minGW from WinLibs](winLibsMinGW.png)

    If command prompt isn't displaying the installed version after running the version check command, manually add the `bin` folder to your path by setting it in your System environment variables.

    >On Windows, search **Edit the System Environment Variables** and click on **Environment Variables...**.
    >
    ![Edit Environment Variables](editEnvironmentVariables.png)

    Look for the `Path` environment variable and double click to edit. Select **Browse** and manually browse through your File Explorer to find the bin folder.

    ![Add bin to path](add-bin-to-path.png)  

    On Linux (if needed), install the System GNU C compiler for your version of Linux.  Note that if you are using openSUSE, minGW is included in the installation for Go through YaST.
    
    ![gcc 64-bit](gccLinux.png)

2. Examine the Go environment by running the below command:

    ```Shell
    go env
    ```

    Notice that GOROOT is set to a location such as `C:\go` or `/usr/lib64/go/1.23`.  This is the location that the Go SDK is installed to.  

    GOPATH is set to a location such as `C:\Users\user\go` or `$HOME/go` and defines the root of your workspace which stores your codebase.

3. Set the `CGO_LDFLAGS` environment variable to point to the location of the HDLRE client library as shown below.

    On Windows, add a **NEW** variable. Set the variable name to **CGO_LDFLAGS** and the value as the location of `dbcapi` library: `C:\SAP\dlclient\IQ-17_1\Bin64\dbcapi.dll`

    ![Set Environment Variables](setEnvVar.png)

    >It is also possible on Microsoft Windows to set this using the SETX command from a shell.


    On Linux, check if the following variable are defined.  
    
    ```Shell (Linux)
    echo $CGO_LDFLAGS
    echo $LD_LIBRARY_PATH
    ```

    If needed, open the '.bashrc' or '.bash_profile' and add the following lines.  Note that the path may be different depending on the data lake client install used.

    ```Shell (Linux)
    pico .bashrc
    export CGO_LDFLAGS=$HOME/sap/dlclient/IQ-17_1/lib64/libdbcapi_r.so
    export CGO_LDFLAGS=$HOME/sap/hdlclient/lib64/libdbcapi_r.so
    export LD_LIBRARY_PATH=$HOME/sap/dlclient/IQ-17_1/lib64
    export LD_LIBRARY_PATH=$HOME/sap/hdlclient/lib64
    ```
    
    ![.bash_profile contents](bashProfileAfterCGO.png)

4. Navigate to the driver folder and create a Go module.  Note that the path may be different depending on the data lake client install used.

    ```Shell (Windows)
    cd %IQDIR17%\sdk\golang\SAP\go-hdlre\driver
    go mod init "SAP/go-hdlre/driver"
    go mod tidy
    ```
    
    ```Shell (Linux)
    cd $IQDIR17/sdk/golang-hdlre/src/SAP/go-hdlre/driver
    cd $IQDIR17/sdk/golang/SAP/go-hdlre/driver/
    go mod init "SAP/go-hdlre/driver"
    go mod tidy
    ```
    ![createModule](createModule.png)

    The contents of the data lake Client folder is not writeable so you may need to change the permissions on the driver folder or copy files to a new location. 

### Create a Go application that queries an SAP data lake Relational Engine
1. In a shell, create a folder named `go`, enter the newly created directory, and open a file named `goQuery.go` in an editor.
    
    ```Shell (Windows)
    mkdir %HOMEPATH%\DataLakeClientsTutorial\go
    cd %HOMEPATH%\DataLakeClientsTutorial\go
    notepad goQuery.go
    ```

    ```Shell (Linux)
    mkdir -p ~/DataLakeClientsTutorial/go
    cd ~/DataLakeClientsTutorial/go
    pico goQuery.go
    ```

2. Add the code below to `goQuery.go` and then modify the connectString variable.

    ```Go Code
    package main

    import (
      "fmt"
    	"database/sql"
    	"log"

      _ "SAP/go-hdlre/driver"
      )

    func main() {
      //specify the connection parameters
      connectString := "hdlre://User1:Password1@xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.iq.hdl.prod-xxxx.hanacloud.ondemand.com:443?enc='TLS{tls_type=rsa;direct=yes}'"

      fmt.Println("Connect String is " + connectString)

      db, err := sql.Open("hdlre", connectString)
      if err != nil {
        log.Fatal(err)
        return
      }
      defer db.Close()

      rows, err := db.Query("SELECT NAME, ADDRESS from HOTELS.CUSTOMER")
      if err != nil {
        log.Fatal(err)
      }
      defer rows.Close()

      var lastName string
      var address string
      for rows.Next() {
        err = rows.Scan(&lastName, &address)
        if err != nil {
          log.Fatal(err)
        }
        fmt.Println(lastName + ": " + address)
      }

      err = rows.Err()
      if err != nil {
      	log.Fatal(err)
      }
    }
    ```

    Once the `goQuery.go` file has been updated, save and close the file.

3. Create another go module and modify its contents:

    ```Shell (Windows)
    go mod init "go/goQuery"
    go mod tidy
    notepad go.mod
    ```   


    ```Shell (Linux)
    go mod init "go/goQuery"
    go mod tidy
    pico go.mod
    ```

4. Add the code below to `go.mod` under the go 1.23.4 (version) line:
    
    >Ensure you have the correct path to the driver folder. The path depends on your installation.  Note that two example locations are provided. Choose the one that's closest to your installation and edit it if necessary.

    ```Code (Windows)
    replace SAP/go-hdlre/driver v0.1.0 => C:\SAP\dlclient\IQ-17_1\SDK\golang\SAP\go-hdlre\driver   
    replace SAP/go-hdlre/driver v0.1.0 => C:\SAP\hdlclient\sdk\golang\SAP\go-hdlre\driver
    require SAP/go-hdlre/driver v0.1.0 
    ```
    
    ```Code (Linux)
    replace SAP/go-hdlre/driver v0.1.0 => /home/name/sap/dlclient/IQ-17_1/sdk/golang-hdlre/src/SAP/go-hdlre/driver 
    replace SAP/go-hdlre/driver v0.1.0 => /home/name/sap/hdlclient/sdk/golang/SAP/go-hdlre/driver
    require SAP/go-hdlre/driver v0.1.0 
    ```

    ![go.mod contents](goModContents.png)

5. Run the application or build and run the application:

    ```Shell
    go run goQuery.go
    ```

    ```Shell Microsoft Windows
    go build goQuery.go
    goQuery.exe
    ```

    ![Result](results.png)

For more information on the API's used, consult the SAP HANA Cloud, data lake connection specific properties at [Connect from Go to Data Lake Relational Engine](https://help.sap.com/docs/SAP_HANA_DATA_LAKE/a895964984f210158925ce02750eb580/0b55e305d26941c191c71eaa07f72bb5.html), [Go Database/SQL Tutorial](http://go-database-sql.org/index.html), and [Package SQL](https://golang.org/pkg/database/sql/)


### Debug the application
Visual Studio Code provides plugins for Go and can be used to debug an application.

1. If you have not already done so, download [Visual Studio Code](https://code.visualstudio.com/Download).

2. If you have not already done so, in Visual Studio Code, choose **File | Add Folder to Workspace**, and then add the `DataLakeClientsTutorial` folder.

    ![Open Workspace](workspace.png)

3. Open the file `goQuery.go`.

    ![Go Extension](GoExtension.png)

    Visual Studio Code will recognize the `go` file extension and will suggest installing the Go for Visual Studio Code extension.  Click **Install**.

4. Place a breakpoint.

    ![SetBreakpoint](SetGoBreakpoint.png)

5. Select **Run | Start Debugging**.

    Notice that the program stops running at the breakpoint that was set.

    Observe the variable values in the leftmost pane.  Step through the code.

    ![Breakpoint](GoBreakpoint.png)  

    >Note that debugging can also be performed from the command line using [Delve](https://github.com/go-delve/delve ).



### Knowledge check
Congratulations! You have now created and debugged a Go application that connects to and queries a data lake Relational Engine.



---
