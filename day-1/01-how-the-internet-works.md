# Chapter 1: How the Internet Actually Works

> ⏱ Estimated time: 45 minutes

## What You'll Learn

- What "client" and "server" mean and how they talk to each other
- What IP addresses and ports are (and why they matter)
- What DNS does (the internet's phone book)
- Exactly what happens when you type a URL in your browser

---

## Concepts

### The Big Picture: Client and Server

Every time you use the internet — loading a website, checking email, watching a video — two computers are talking to each other.

- **Client**: The computer asking for something (your laptop, your phone, your browser)
- **Server**: The computer that has the thing and sends it back

That's it. The entire internet is built on this one pattern: **ask and answer**.

```
┌──────────┐                        ┌──────────┐
│          │  "Give me this page"   │          │
│  CLIENT  │ ────────────────────►  │  SERVER  │
│ (browser)│                        │(computer)│
│          │  ◄──────────────────── │          │
│          │  "Here's the page"     │          │
└──────────┘                        └──────────┘
```

**Analogy**: Think of a restaurant. You (the client) sit at a table and ask for food. The kitchen (the server) prepares it and sends it out. You don't walk into the kitchen. The kitchen doesn't come to your table unprompted. There's a clear ask-and-answer pattern.

> **Key Insight**: The server doesn't do anything until someone asks. It sits there, waiting. When a request comes in, it processes it and sends a response. Then it goes back to waiting.

### What Is a Server, Really?

A "server" is just a regular computer running a program that listens for requests. There's nothing magical about it. Your laptop could be a server right now — all it needs is a program that says "I'm listening, send me requests."

When you write a Spring Boot application later in this guide, your laptop will literally become a server.

### IP Addresses: Finding the Right Computer

The internet has billions of devices. How does your request find the right server?

Every device connected to the internet has an **IP address** — a unique number that identifies it, like a street address for a house.

```
Your laptop:     192.168.1.5
Google's server:  142.250.80.46
GitHub's server:  140.82.121.4
```

**IPv4** addresses look like `192.168.1.5` — four numbers separated by dots (each 0–255).

**IPv6** addresses look like `2001:0db8:85a3:0000:0000:8a2e:0370:7334` — we're running out of IPv4 addresses, so this longer format exists. You don't need to memorize this.

**Analogy**: IP addresses are like street addresses. If you want to mail a letter to someone, you need their address. If you want your computer to talk to a server, you need its IP address.

### Ports: Finding the Right Program

One server computer can run many programs. A port number tells the request *which program* on that computer should handle it.

```
Server at 142.250.80.46
  ├── Port 80:   Web server (HTTP)
  ├── Port 443:  Web server (HTTPS, secure)
  ├── Port 25:   Email server
  └── Port 5432: Database server
```

**Analogy**: If the IP address is a building's street address, the port is the apartment number. The mail carrier needs both to deliver to the right place.

A full network address looks like: `142.250.80.46:443`
- `142.250.80.46` = which computer
- `443` = which program on that computer

Common ports to know:
| Port | Used For |
|------|----------|
| 80   | HTTP (web traffic, unencrypted) |
| 443  | HTTPS (web traffic, encrypted) |
| 8080 | Common for development servers (including Spring Boot!) |
| 3306 | MySQL database |
| 5432 | PostgreSQL database |

> **Why 8080?** Port 80 often requires administrator privileges. Port 8080 is the convention for "I'm a web server running in development mode." When you run your Spring Boot app, it will default to port 8080.

### DNS: The Internet's Phone Book

Nobody types `142.250.80.46` into their browser. We type `google.com`. But computers need IP addresses. Something has to translate.

**DNS (Domain Name System)** is that translator. It's like a phone book that maps human-readable names to IP addresses.

```
You type:  google.com
DNS says:  "That's 142.250.80.46"
Your browser: connects to 142.250.80.46
```

The DNS lookup happens automatically — you never see it. But it happens every time you visit a website.

**How DNS resolution works (simplified):**

```
1. You type "google.com" in your browser
2. Your computer asks your router: "What's the IP for google.com?"
3. Router doesn't know → asks your ISP's DNS server
4. ISP's DNS doesn't know → asks a root DNS server
5. Root server says: "Ask the .com DNS server"
6. .com server says: "Ask Google's DNS server"
7. Google's DNS says: "142.250.80.46"
8. Answer flows back to your browser
9. Your computer caches it so it doesn't ask again for a while
```

This entire process takes about 20–100 milliseconds.

### What Happens When You Type a URL

Let's trace the complete journey when you type `https://www.example.com/books` in your browser:

```
Step 1: DNS Resolution
  Browser → DNS → "www.example.com = 93.184.216.34"

Step 2: TCP Connection
  Your computer establishes a connection to 93.184.216.34:443
  (443 because it's HTTPS)

Step 3: TLS Handshake (the "S" in HTTPS)
  Your browser and the server agree on encryption
  (So nobody can read the data in transit)

Step 4: HTTP Request
  Your browser sends:
  "GET /books HTTP/1.1
   Host: www.example.com"
  (We'll learn exactly what this means in Chapter 2)

Step 5: Server Processing
  The server receives the request
  Runs some code to figure out what to respond with
  Maybe queries a database for book data

Step 6: HTTP Response
  The server sends back:
  "HTTP/1.1 200 OK
   Content-Type: text/html
   
   <html>...the page content...</html>"

Step 7: Rendering
  Your browser receives the HTML and draws the page on screen
```

```
┌────────┐     ┌─────┐     ┌────────────┐     ┌──────────┐
│Browser │────►│ DNS │────►│   Server   │────►│ Database │
│        │     │     │     │ (your code │     │          │
│        │◄────│     │◄────│  runs here)│◄────│          │
└────────┘     └─────┘     └────────────┘     └──────────┘
   Step 1        Step 1      Steps 4-6          Step 5
```

**This is the journey every single web request takes.** Every Google search, every Instagram post, every Netflix stream. The same pattern, billions of times per second, all over the world.

> 🧠 **Think Like a Backend Engineer**: Your job, as a backend developer, is Step 5. You write the code that receives the request, figures out what to do with it, and sends back the response. Everything else (DNS, TCP, TLS, browser rendering) is handled for you.

---

## Code Examples

You don't need to write code for this chapter, but here's a quick demonstration that servers are just programs. You've used Java — here's what a barebones server looks like in Java (don't worry about understanding every line):

```java
// DON'T TYPE THIS — it's just to show that a server is "just a Java program"
import java.net.ServerSocket;
import java.net.Socket;
import java.io.*;

public class TinyServer {
    public static void main(String[] args) throws Exception {
        // Listen on port 8080
        ServerSocket server = new ServerSocket(8080);
        System.out.println("Server is waiting on port 8080...");
        
        while (true) {
            // Wait for a client to connect (blocks here until someone connects)
            Socket client = server.accept();
            
            // Read what the client sent
            BufferedReader in = new BufferedReader(
                new InputStreamReader(client.getInputStream())
            );
            String request = in.readLine();
            System.out.println("Received: " + request);
            
            // Send a response
            PrintWriter out = new PrintWriter(client.getOutputStream(), true);
            out.println("HTTP/1.1 200 OK");
            out.println("Content-Type: text/plain");
            out.println();
            out.println("Hello from my tiny server!");
            
            client.close();
        }
    }
}
```

Notice the pattern:
1. Listen on a port (`new ServerSocket(8080)`)
2. Wait for a request (`server.accept()`)
3. Read the request (`in.readLine()`)
4. Send a response (`out.println(...)`)
5. Go back to waiting (`while(true)`)

This is what every web server does. Spring Boot does exactly this — but with a lot more power and a lot less boilerplate.

---

## Exercise: Draw the Request Flow

**Goal**: Internalize the client-server model by tracing a real scenario.

### Task

On paper (or a drawing app), draw the complete journey for this scenario:

> You open your phone's weather app. It shows the weather for your city.

Draw and label:
1. Who is the **client**? (Be specific — is it the phone? The app?)
2. Who is the **server**?
3. What **request** does the client send? (What is it asking for?)
4. What does the server need to do to **process** this request?
5. What **response** does the server send back?
6. Where does **DNS** fit in?
7. What **port** might the server be listening on?

### Bonus Questions

- What happens if the server is down? What does the client see?
- What if your phone has no internet connection? Where does the process break?
- Could the weather server also be a *client* to some other server? (Hint: where does it get the actual weather data?)

---

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| "A server is a special kind of computer" | A server is a regular computer running server software. Your laptop is a server right now if you run server code on it. |
| "The internet is one big thing" | The internet is millions of independent computers agreeing to talk using the same rules (protocols). |
| "Only browsers are clients" | Mobile apps, desktop apps, command-line tools, even other servers can be clients. Anything that sends a request is a client. |
| "DNS happens once" | DNS results are *cached*, but they expire. Your computer periodically re-checks. |

---

## Key Takeaways

- [ ] I can explain the client-server model in my own words
- [ ] I know that an IP address identifies a computer and a port identifies a program on that computer
- [ ] I understand that DNS translates domain names (google.com) to IP addresses
- [ ] I can trace the full journey of a web request from browser to server and back
- [ ] I understand that a server is just a program that listens for requests — and I'll be writing one

---

## Quick Quiz

1. Your friend says "I need to set up a server." What do they actually need to do?
2. Why can't two programs on the same computer both listen on port 8080?
3. What would happen if DNS didn't exist? Could the internet still work?
4. In the URL `http://localhost:8080/books`, what is `localhost`? What is `8080`? What is `/books`?
5. When your Spring Boot app runs on your laptop, is your laptop a client or a server? (Trick question — think carefully.)

---

*Next: `02-speaking-http.md` — We'll learn the exact language that clients and servers use to talk to each other →*
