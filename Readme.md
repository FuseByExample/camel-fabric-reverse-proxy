## Camel reverse proxy with Fuse Fabric

### Setup
Setup
- Install Apache Maven 3.2.3 or later
    - Download distribution from http://maven.apache.org. 
    - Unzip the downloaded Maven distribution to a location on your hard disk that you find suitable.
    - configure this location as the environment variable MAVEN_HOME
    - add MAVEN_HOME/bin to your PATH environment variable

- Install JBoss Fuse  6.3.0
    - Download from https://access.redhat.com/jbossnetwork/ (registration required) and extract
    
### Configuring additional users
Before we can start the Fuse ESB, we have to make sure we configure a user we can use later on to connect to the embedded
message broker and send messages to a queue.  Edit the '$ESB_HOME/etc/users.properties file and add a line that says:

    admin=admin,Administrator

The syntax for this line is &lt;userid&gt;=&lt;password&gt;,&lt;role&gt;, so we're creating a user called `admin` with a password `admin`
who has the `Administrator` role.

### Start JBoss Fuse
Start JBoss Fuse with these commands

* on Linux/Unix/MacOS: `bin/fuse`
* on Windows: `bin\fuse.bat`

Create a fabric in the fuse console

JBossFuse:karaf@root> fabric:create --wait-for-provisioning 

### Install the example profile

This uses the fabric8 plugin to generate a profile to install into a fabric.

See more here: http://fabric8.io/#/site/book/doc/index.md?chapter=mavenPlugin_md

Note, to use the fabric8-maven-plugin, youll need this in your ~/.m2/settings.xml (see the docs):

    <server>
        <id>fabric8.upload.repo</id>
        <username>admin</username>
        <password>admin</password>
    </server>

cd to the home directory for this project and run

    mvn clean install

Then move to the maven module *http-gateway-proxy* and run the maven fabric8 deploy plugin :

    mvn fabric8:deploy
    
to run against a local fabric. if you have a remote fabric do this:

    mvn fabric8:deploy -Dfabric8.jolokiaUrl=http://fuse-xpaas.xpaas.com/jolokia
    
This should build the jar, build the profile, upload both to the fabric, and from there you
can deploy it to a container by executing the following command at the fuse console:

     JBossFuse:karaf@root> container-add-profile root com.redhat.httpgateway-http-gateway-proxy 

The routing is limited, although the pieces are in place to expand on it.

The assumption is that each group of RESTFull services to load balance against has one and only one
url context path (for e.g., /context/path/foo)

What this does is take a request, determine how to map the path to the fabric cluster name and then
call it using a recipient list.

For example, a call to _http://localhost:9000/test_ will use a recipient list to call fabric:gatewaytest

Note that the context was used to concatenate with a constant "gateway" ... so /test got replaced to gatewaytest.
See the Router class for the implementation of the recipient list. Also note where you can plug in your custom logic.

There are three groups: test, beer, cheese.

You can hit each one with the following commands:

    curl http://localhost:9000/beer
    curl http://localhost:9000/test
    curl http://localhost:9000/cheese
    