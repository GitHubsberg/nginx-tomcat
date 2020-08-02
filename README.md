# nginx-tomcat
Here is the configuration required to run a simple VM running nginx in the fronend and tomcat in the backend

At a very high level we let nginx listen on port 80 and send all requests to tomcat on port 8080. We then allow
tomcat do the virtual hosting thereby handling the processing of requests to cbills.com, mycardrooms.com and nyteout.com

Here is how this works.
within the nginx.conf we have this
```
         location / {
            proxy_set_header X-Forwarded-Host $host;
            proxy_set_header X-Forwarded-Server $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header  Host $http_host;
            proxy_pass http://127.0.0.1:8080/;
         }
```
here we are telling nginx to proxy_pass all requests to "/" for all requests to Tomcat.

Now on the tomcat side we have in server.xml a definition for every virtual host we will handle. If you look
at `Server/Service/Engine` you will see definitions for four different hosts. Lets look at one of those:

```
      <Host name="cbills.com"  
      	    appBase="webapps/cbills.com"
            unpackWARs="true" 
            autoDeploy="true">
        <Valve className="org.apache.catalina.valves.RemoteIpValve" />    
        <Valve className="org.apache.catalina.valves.AccessLogValve" 
               directory="logs"
               prefix="cbills_access_log" 
               suffix=".txt"
               buffered="false"
               pattern="%I %{X-Forwarded-For}i %S %u [%{begin:dd/MM/yyyy:HH:mm:ss.SSS Z}t] &quot;%r&quot; %s %b %D" 
        />
      </Host>
```

notice how we specify the Host name to be `cbills.com` and set its appBase to `"webapps/cbills.com"`. The web app (war file) is deployed
under the directory `${TOMCAT_HOME}/webapps/cbills.com/ROOT/` where `TOMCAT_HOME` is the directory where your tomcat is deployed. You will 
need to create a `cbilss.com/ROOT/` directory under webapps and place the contents of the web app war file there. If you want to use the
`.war` format, just rename it to ROOT.war and place it under `${TOMCAT_HOME}/webapps/cbills.com`