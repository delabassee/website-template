
## Initial setup

Start a new Maven project and select *"Web Application"*

Give it a meaningful name, ex. *"hol-security"*, click *"Next"*

Select *"GlassFish"* as the application server and *"Java EE7 Web"*, we will update the Java EE API version later. Validate and you should now have an empty project.


Select the projects's *pom.xml*, under *"Project Files"*. Now you can update the project to use Java EE 8 APIs but just updating the version number of the *<javaee-web-api>* dependency.

:bulb: This Lab is only using APIs from the Java EE 8 Profile.

```xml
 <dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-api</artifactId>
    <version>8.0</version>
    <scope>provided</scope>
 </dependency>
```

## Create the Web application

We will not create a Servlet-based Web application that we will later on secure using the new Java EE Security API (JSR 375).

Right click on the project, select *"new"* and *"servlet"*.
Make sure to specify a package where you code will reside, eg. *"org.j1hol"* and click *"next"* then *"finish"*.

:bulb: We need to make sure to enable CDI. For that, we need to create an empty beans.xml in the WEB-INF directory. Beans.xml can also be used to customise the CDI container; when it's empty, it's just a marker file that tell the application server that CDI is used.

In the project, locate the WEB-INF directory under "Web Pages" and right click to create an empty file called beans.xml

:bulb: Even tough the file looks empty, it's not as it's size is 1 byte. That means that GlassFish will parse it so it has to be a valid XML file. Just add *<xml/>* to make sure that it is indeed a valid XML file. So instead of an empty file, we have have created a valid empty xml file!


Navigate to the servlet source that has just been created and locate the *processRequest* method.

Update it as follow

```java
 protected void processRequest(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
    response.setContentType("text/html;charset=UTF-8");
    try (PrintWriter out = response.getWriter()) {
        out.println("<!DOCTYPE html><html><body>");
        out.println("<div style=\"font-size:150%;font-weight:100;font-family: sans-serif;text-align: center;color: DimGray;margin-top: 40vh;line-height: 150%\">");
        out.println("Java EE 8 HoL<br/>");
        out.println(request.getAuthType());
        out.println("</body></html>");                     
    }
 }
```

Now right click and select *"Run File"*. This action will compile and deploy our Web Application to GlassFish 5, NetBeans will then launch the browser to test it.

If it works, you will simply see the text produced by the processRequest method "Java EE HoL". 

:bulb: When testing your application, always make sure to connect to the servlet and not to the *index.html* that NetBeans also creates!

## Secure the Web application

We now have a simple CDI enabled Servlet 4.0 basic application that we will secure using the Java EE Security API.

:warning: To demonstrate the concept, we will use Basic Authentication. BasicAuth is a simple mechanism where the browser sends the user name and the password as unencrypted base64 encoded text to the server. This is **clearly not a secure approach and it shouldn't be used for any serious application:bangbang:**


In the servlet class source, you shoudl have the following annotation 

```java
 @WebServlet(name = "NewServlet", urlPatterns = {"/test"})
```
:arrow_right: We are specifying to GlassFish Web container that this class is a servlet and that its url is *"/test"* (under the the context of the Web application).

You can now add the following annotations:
```java
 @DeclareRoles({"foo", "bar"})
```
:arrow_right: This annotation specific that our application will work with 2 types of user roles: foo & bar.

```java
 @ServletSecurity(@HttpConstraint(rolesAllowed = "foo"))
```
:arrow_right: This annotation specify security constraints to be enforced by the Servlet container on HTTP protocol messages, i.e. only user of role *"foo"* will be allowed.

```java
 @BasicAuthenticationMechanismDefinition(realmName="HOL-basic" )
```
:arrow_right: This annotation specify the speicifc container authentication mechanism : "HTTP basic access authentication"and make that it available as an enabled CDI bean.

```java
 @EmbeddedIdentityStoreDefinition({
    @Credentials(callerName = "david", password = "david", groups = {"foo"}),
    @Credentials(callerName = "ed", password = "ed", groups = {"bar",}),
    @Credentials(callerName = "michael", password = "michael", groups = {"foo"})}
 )
```

:arrow_right: This annotation is used to specify which IdentityStore to use, in this case, we will use the   Embedded Identity Store which require to directly specify our users. 

:warning: In the interrest of time, we will take a shortcut and use the most basic IdentityStore to store our users: EmbeddedIdentityStore. This identity store allows you to store users details (username, password and groups) in your code in clear! *This is highly unsecure and not something you should do for any applications:bangbang:*
Since EmbeddedIdentityStore is unsecure it was decided to not include it in the specification. Technically, the EmbeddedIdentityStore is provided by Soteria, the JSR 375 Reference Implementation so it is also available as part of GlassFish 5.

To compile your code, you need to add the Soteria dependency to your pom.xml 

```xml
  <dependency>
     <groupId>org.glassfish.soteria</groupId>
     <artifactId>javax.security.enterprise</artifactId>
     <version>1.0</version>
     <scope>provided</scope>
 </dependency>
```

Now you can test the application, *david* and *michael* should be able to logged in the application while *ed* will be rejected (HTTP Status 403 - Forbidden) as he doesn't have the approrpiate *"foo"* role. This error page is obviously customizable but that is beyond the scope of this exercice.

*PIC*

:bulb: To avoid caching issue, just start Chrome in Incognito mode each time you want to test authentication.

# Secure the application using a custom IdentityStore

In the previous step, you saw that it was possible to easily secure a web application in a portable way! 
To illustrate the concepts, we have used an *EmbeddedIdentityStore* which, by design, is not secure. The Java EE Security API defines *IndendtityStore* that are designed to cope with modern applications, namely the **LdapIdentityStore** and the **DatabaseIdentityStore** which be used to (securely!) store users information respectvely in a LDAP and in a database. These stores can be configured using the *@LdapIdentityStoreDefinition* and the *@DatabaseIdentityStoreDefinition* annotations.

The Java EE Security API can be extended. In this exerice, we will evolve our application to use a custom *IndendtityStore* instead of the *EmbeddedIdentityStore*. In the interreste of time, the *IdentityStore* we will develop will not be secure.

In the servlet class, remove or comment the *@EmbeddedIdentityStoreDefinition* annotation.
Now, create a java class named **TestIdentityStore.java**. As this class will implement an IndentiyStore, it has to iplments the *IdentityStore* interface. Also make sure the class has the *Application* scope by annotating it with *@ApplicationScoped* as below.

```java
 @ApplicationScoped
 public class TestIdentityStore implements IdentityStore {
 ...
```
NetBeans will complain that some imports are missing, fix those.

**PIC4**

```java
 import javax.enterprise.context.ApplicationScoped;
 import javax.security.enterprise.identitystore.IdentityStore;
```



