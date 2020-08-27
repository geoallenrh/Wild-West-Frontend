
# Openshift Deployment Instructions

**prerequisites**

* an openshift cluster available.

* a terminal with the openshift command line client and active login
  session to the openshift cluster available.

* Tested on OCP 4.5.4

* odo v1.2.1 (b7b7c2fac) (odo experimental mode did not work).

```
$ odo preference set experimental false
```


**oc cli based deployment**
This will deploy the components quickly, but there isn't the ability to peform iterative deployment like with odo.

**Create a new project**
   ```
   oc new-project wildwest
   ```
**Deploy backend service**

```
oc new-app fabric8/s2i-java~https://github.com/geoallenrh/Wild-West-Backend --name=backend-app
```

**Apply policy to view System Resources (Pods, BuildConfigs, etc)**
```
oc policy add-role-to-user view system:serviceaccount:<project-name>:default
```
```
oc policy add-role-to-user view system:serviceaccount:wildwest:default
```
**Optional - Apply policy to actually delete resources (like Pods)**
```
oc policy add-role-to-user edit system:serviceaccount:<project-name>:default
```
**Create frontend app**

```
oc new-app https://github.com/geoallenrh/Wild-West-Frontend --name=frontend-app -e BACKEND_SERVICE=backend-app:8080
```
**Expose Service as Route**
```
oc expose service/frontend-app
```
**Get Public URL**
```
oc get route frontend-app
```

**Using odo**

```
$ mkdir wildwest
$ cd wildwest
```

**Clone Backend**

```
$ git clone https://github.com/geoallenrh/Wild-West-Backend backend
Cloning into 'backend'...
remote: Enumerating objects: 391, done.
remote: Total 391 (delta 0), reused 0 (delta 0), pack-reused 391
Receiving objects: 100% (391/391), 89.15 KiB | 742.00 KiB/s, done.
Resolving deltas: 100% (107/107), done.
```
**Valdiate directory exists**
```
$ ls
backend
```

**Clone Front End**
```
$ git clone https://github.com/geoallenrh/Wild-West-Frontend frontend
Cloning into 'frontend'...
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 294 (delta 0), reused 0 (delta 0), pack-reused 290
Receiving objects: 100% (294/294), 5.57 MiB | 698.00 KiB/s, done.
Resolving deltas: 100% (140/140), done.
```
**Validate directory exists**
```
$ ls
backend		frontend
```

**Create Project**

```
$ odo project create wildwest-demo
 ✓  Project 'wildwest-demo' is ready for use
 ✓  New project created and now using project: wildwest-demo
 ```

**List Available Projects**
```
$ odo catalog list components
Odo OpenShift Components:
NAME       PROJECT       TAGS                      SUPPORTED
java       openshift     11,8,latest               YES
nodejs     openshift     10,12,latest              YES
dotnet     openshift     2.1,3.1,latest            NO
golang     openshift     1.11.5,latest             NO
httpd      openshift     2.4,latest                NO
nginx      openshift     1.10,1.14,1.16,latest     NO
perl       openshift     5.26,latest               NO
php        openshift     7.2,7.3,latest            NO
python     openshift     2.7,3.6,latest            NO
ruby       openshift     2.4,2.5,2.6,latest        NO
```

**Change to Backend Directory**
``
$ cd backend/
$ ls
debug.sh	pom.xml		src
``
**Build Project**
```
$ mvn package
.....
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.803 s
[INFO] Finished at: 2020-08-20T22:00:04-05:00
[INFO] ------------------------------------------------------------------------
```

**Create Backend Component**
```
$ odo create java backend --binary target/wildwest-1.0.jar
Validation
 ✓  Validating component [333ms]

Please use `odo push` command to create the component with source deployed
```

**View current config**
```
$ odo config view
COMPONENT SETTINGS
------------------------------------------------
PARAMETER         CURRENT_VALUE
Type              java
Application       app
Project           wildwest-demo
SourceType        binary
Ref               
SourceLocation    target/wildwest-1.0.jar
Ports             8080/TCP,8443/TCP,8778/TCP
Name              backend
MinMemory         
MaxMemory         
DebugPort         
Ignore            
MinCPU            
MaxCPU
```

**Push the Backend component**

```
$ odo push
Validation
 ✓  Checking component [208ms]

Configuration changes
 ✓  Initializing component
 ✓  Creating component [1s]

Applying URL changes
 ✓  URLs are synced with the cluster, no changes are required.

Pushing to component backend of type binary
 ✓  Checking files for pushing [5ms]
 ✓  Waiting for component to start [31s]
 ✓  Syncing files to the component [1m]
 ✓  Building component [2s]
```

**View the log**

```
$ odo log -f
2020-08-21 03:06:14.570  INFO 125 --- [           main] c.o.wildwest.WildWestApplication         : Started WildWestApplication in 17.512 seconds (JVM running for 20.503)
```

**View the component status**

```
$ odo list
APP     NAME        PROJECT           TYPE     SOURCETYPE     STATE
app     backend     wildwest-demo     java     binary         Pushed
```

**Move to Front end Directory**
```
$ cd ../frontend/
$ ls
README.md		bin			index.html		package-lock.json	playfield.png
assets			favicon.ico		kwww-frontend.iml	package.json		server.js
```
**Create Frontend Component**
```
$ odo create nodejs frontend
Validation
 ✓  Validating component [130ms]
```

**Push Component**
```
$ odo push
Validation
 ✓  Checking component [202ms]

Configuration changes
 ✓  Initializing component
 ✓  Creating component [577ms]

Applying URL changes
 ✓  URLs are synced with the cluster, no changes are required.

Pushing to component frontend of type local
 ✓  Checking files for pushing [3ms]
 ✓  Waiting for component to start [22s]
 ✓  Syncing files to the component [13s]
 ✓  Building component [17s]
 ✓  Changes successfully pushed to component
```

**List Components**
```
$ odo list
APP     NAME         PROJECT           TYPE       SOURCETYPE     STATE
app     backend      wildwest-demo     java       binary         Pushed
app     frontend     wildwest-demo     nodejs     local          Pushed
```

**Link Components**
```
$ odo link backend --port 8080
 ✓  Component backend has been successfully linked to the component frontend

The below secret environment variables were added to the 'frontend' component:

· COMPONENT_BACKEND_HOST
· COMPONENT_BACKEND_PORT

You can now access the environment variables from within the component pod, for example:
$COMPONENT_BACKEND_HOST is now available as a variable within component frontend
$ odo url create frontend --port 8080
 ✓  URL frontend created for component: frontend

To apply the URL configuration changes, please use `odo push`
```

**Push Link**
```
$ odo push
Validation
 ✓  Checking component [283ms]

Configuration changes
 ✓  Retrieving component data [327ms]
 ✓  Applying configuration [686ms]

Applying URL changes
 ✓  URL frontend: http://frontend-app-wildwest-demo.apps.cluster-af98.af98.example.opentlc.com created

Pushing to component frontend of type local
 ✓  Checking file changes for pushing [1ms]
 ✓  No file changes detected, skipping build. Use the '-f' flag to force the build.
```
