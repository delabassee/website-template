## Introduction

The main feature of the Servlet 4.0 API is to provide HTTP/2 support. In this exerice, we will see what it takes to HTTP/2 enbale the simple Web application that we have developed in the previous exercice.

Check the URL of the application, it connect in clear to the port 8080. In Chrome, change the url to use *https* instead of *http* and to connect to port *8181* instead of port *8080*. Now connect to that updated URL.

Chrome will complain by saying that "*Your connection is not private*"; click on *ADVANCED* and then on *Procceed to locahost (unsafe)*.

**PIC5-1**

Login as *ed/ED*, you should see the 'main screen' and a small blue lightning at the top right corner. This is Chrome HTTP/2 indicator,  blue means that the connection is server over HTTP/2. If you click on it, you will be able to details on the HTTP/2 connection (streams, frames, etc.). This is really all it takes to HTTP/2 enable an exisiting Servlet application, just deploy on a Servlet 4 container! 

**PIC5-2**
