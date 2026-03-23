# 🌐 Part 6: Web Technologies, Frontend & Other Concepts (Q123–Q142)

---

## Q123. Servlet — Lifecycle & Forwarding

**Servlet Lifecycle:**
```
Class Loading → Instantiation → init() → service(req,res) [repeated] → destroy()
```

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    @Override
    public void init() { System.out.println("Servlet initialized — ONCE"); }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        resp.setContentType("text/html");
        resp.getWriter().println("<h1>Hello, " + req.getParameter("name") + "</h1>");
    }

    @Override
    public void destroy() { System.out.println("Servlet destroyed"); }
}
```

**Forwarding vs Redirect:**
```java
// Forward — server-side, URL doesn't change, same request
req.getRequestDispatcher("/result.jsp").forward(req, resp);

// Redirect — client-side, URL changes, NEW request
resp.sendRedirect("https://example.com/success");
```

| Forward | Redirect |
|---------|----------|
| Server-side | Client-side (302 response) |
| Same request/response | New request |
| URL unchanged | URL changes |
| Faster | Slower (round trip) |

---

## Q124. JSP — Lifecycle, Implicit Objects, Tags

**JSP Lifecycle:** JSP → translated to Servlet → compiled → loaded → executed.

**Implicit Objects:** `request`, `response`, `session`, `application`, `out`, `config`, `pageContext`, `page`, `exception`.

```jsp
<!-- Scriptlet -->
<% String name = request.getParameter("name"); %>

<!-- Expression -->
<p>Hello, <%= name %></p>

<!-- Declaration -->
<%! int count = 0; %>

<!-- Directive -->
<%@ page language="java" contentType="text/html" %>
<%@ include file="header.jsp" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<!-- EL (Expression Language) — preferred over scriptlets -->
<p>Welcome, ${sessionScope.user.name}</p>

<!-- JSTL -->
<c:forEach var="emp" items="${employees}">
    <p>${emp.name} - ${emp.salary}</p>
</c:forEach>

<c:if test="${user.role == 'ADMIN'}">
    <a href="/admin">Admin Panel</a>
</c:if>
```

---

## Q125. JSTL (JSP Standard Tag Library)

**JSTL** provides ready-to-use tags replacing scriptlets for cleaner code.

| Library | Prefix | Purpose |
|---------|--------|---------|
| Core | `c` | Loops, conditions, URL |
| Formatting | `fmt` | Date, number formatting |
| SQL | `sql` | DB queries (avoid in production) |
| XML | `x` | XML processing |
| Functions | `fn` | String functions |

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<c:choose>
    <c:when test="${score >= 90}">Grade: A</c:when>
    <c:when test="${score >= 70}">Grade: B</c:when>
    <c:otherwise>Grade: C</c:otherwise>
</c:choose>
```

---

## Q126. web.xml and struts.xml

```xml
<!-- web.xml — Deployment Descriptor -->
<web-app>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    </filter>

    <session-config>
        <session-timeout>30</session-timeout>
    </session-config>
</web-app>

<!-- struts.xml — Struts Configuration -->
<struts>
    <package name="default" extends="struts-default">
        <action name="login" class="com.app.LoginAction">
            <result name="success">/welcome.jsp</result>
            <result name="error">/login.jsp</result>
        </action>
    </package>
</struts>
```

---

## Q127. Node.js, NPM, package.json

**Node.js** = JavaScript runtime built on Chrome's V8 engine. Event-driven, non-blocking I/O.

**NPM** = Node Package Manager — installs/manages dependencies (`npm install express`).

```json
// package.json
{
  "name": "my-api",
  "version": "1.0.0",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.0.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.0"
  }
}
```

---

## Q128. Angular Basics — Routing, Auth, Lifecycle

```typescript
// Routing
const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'dashboard', component: DashboardComponent, canActivate: [AuthGuard] },
  { path: '**', component: NotFoundComponent }
];

// Auth Guard
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(): boolean {
    return this.authService.isLoggedIn();
  }
}

// Component Lifecycle Hooks (in order):
// ngOnChanges → ngOnInit → ngDoCheck → ngAfterContentInit →
// ngAfterContentChecked → ngAfterViewInit → ngAfterViewChecked → ngOnDestroy
```

**Angular Compiler:** AOT (Ahead-Of-Time — production, faster) vs JIT (Just-In-Time — development).

---

## Q129. HTML Basics, AJAX, JSON, XML

```html
<!-- Table -->
<table border="1">
  <tr><th>Name</th><th>Age</th></tr>
  <tr><td>Alice</td><td>25</td></tr>
</table>

<!-- Dropdown -->
<select id="city">
  <option value="mumbai">Mumbai</option>
  <option value="delhi">Delhi</option>
</select>

<!-- Checkbox -->
<input type="checkbox" name="skill" value="java"> Java
<input type="checkbox" name="skill" value="python"> Python
```

```javascript
// AJAX call (modern fetch API)
fetch('/api/users')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error(error));

// JSON
const user = { "name": "Alice", "age": 25)};
JSON.stringify(user);  // object to string
JSON.parse(jsonString); // string to object
```

```xml
<!-- XML -->
<employees>
  <employee id="1">
    <name>Alice</name>
    <salary>75000</salary>
  </employee>
</employees>
```

---

## Q130-Q131. What Happens When You Type google.com in Browser

```
1. DNS Resolution
   Browser cache → OS cache → Router → ISP DNS → Root DNS → TLD (.com) → Google's NS
   Returns IP: 142.250.77.46

2. TCP Connection (Three-Way Handshake)
   Client → SYN → Server
   Client ← SYN-ACK ← Server
   Client → ACK → Server

3. TLS/SSL Handshake (for HTTPS)
   Exchange certificates, agree on encryption

4. HTTP Request
   GET / HTTP/1.1
   Host: www.google.com

5. Server Processing
   Load balancer → Web server → Application server → Database (if needed)

6. HTTP Response
   HTTP/1.1 200 OK
   Content-Type: text/html
   [HTML body]

7. Browser Rendering
   Parse HTML → Build DOM tree → Parse CSS → CSSOM
   → Render tree → Layout → Paint → Composite
   Execute JavaScript

8. Connection Close (or keep-alive)
```

---

## Q132-Q133. HTTP Request Lifecycle, Methods

**HTTP Methods:**
| Method | Purpose | Body | Idempotent | Safe |
|--------|---------|------|-----------|------|
| GET | Read | No | Yes | Yes |
| POST | Create | Yes | No | No |
| PUT | Full update | Yes | Yes | No |
| PATCH | Partial update | Yes | No | No |
| DELETE | Delete | Optional | Yes | No |
| HEAD | Headers only | No | Yes | Yes |
| OPTIONS | Allowed methods | No | Yes | Yes |

---

## Q134. HTTP Status Codes

| Code | Meaning | Example |
|------|---------|---------|
| **2xx** | Success | |
| 200 | OK | GET succeeded |
| 201 | Created | POST created resource |
| 204 | No Content | DELETE succeeded |
| **3xx** | Redirection | |
| 301 | Moved Permanently | URL changed |
| 304 | Not Modified | Cache valid |
| **4xx** | Client Error | |
| 400 | Bad Request | Invalid JSON |
| 401 | Unauthorized | Not logged in |
| 403 | Forbidden | No permission |
| 404 | Not Found | Wrong URL |
| 409 | Conflict | Duplicate entry |
| **5xx** | Server Error | |
| 500 | Internal Server Error | Unhandled exception |
| 502 | Bad Gateway | Upstream server down |
| 503 | Service Unavailable | Server overloaded |

---

## Q135. GET vs POST

| GET | POST |
|-----|------|
| Data in URL (`?key=val`) | Data in request body |
| Bookmarkable | Not bookmarkable |
| Cached by browser | Not cached |
| Length limited (~2KB URL) | No size limit |
| Idempotent (safe to repeat) | Not idempotent |
| Use: fetching data | Use: submitting forms, creating resources |

---

## Q136. Session vs Cookie

| Session | Cookie |
|---------|--------|
| Stored on **server** | Stored on **client** (browser) |
| More secure | Less secure (visible in browser) |
| Expires when browser closes (default) | Can have expiry date |
| Can store any object | Only strings |
| Uses session ID (in cookie/URL) | Sent with every request |

```java
// Session (server-side)
HttpSession session = request.getSession();
session.setAttribute("user", userObj);

// Cookie (client-side)
Cookie cookie = new Cookie("theme", "dark");
cookie.setMaxAge(60 * 60 * 24 * 7);  // 7 days
response.addCookie(cookie);
```

---

## Q137. Stateless Protocol

**HTTP is stateless** — each request is independent; the server doesn't remember previous requests.

**Why stateless?** Simplicity, scalability (any server can handle any request), reliability.

**How to maintain state?**
- **Cookies** — store session ID on client
- **Sessions** — store data on server (linked via session ID in cookie)
- **Tokens (JWT)** — self-contained token with user info, no server-side storage
- **URL Rewriting** — append session ID to URLs

---

## Q138-Q142. Client-Server Architecture, API Internals, Request-Response Cycle, REST

**Client-Server Architecture:**
```
Client (Browser/App) ←── HTTP ──→ Server (Backend)
                                       ↕
                                   Database
```

**How API Works Internally:**
```
Client → HTTP Request → DNS → Load Balancer → Web Server
  → Middleware (auth, logging) → Router → Controller
  → Service Layer → Repository → Database
  → Response flows back up → JSON response to Client
```

**Request-Response Cycle:**
1. Client sends HTTP request (method, URL, headers, body)
2. Server receives, parses, routes to handler
3. Handler processes business logic, accesses DB
4. Server sends HTTP response (status code, headers, body)

**REST API in Simple Terms:**
> "REST API is like a restaurant waiter. The client (customer) sends a request (order), the waiter (API) takes it to the kitchen (server/DB), and brings back the response (food). The menu is the API documentation."
