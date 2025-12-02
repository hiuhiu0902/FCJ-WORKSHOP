---
title: "Week 11 Worklog"
date: "2025-11-17T09:00:00+07:00"
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 Objectives:
* Implement Application Security (Spring Security).
* Optimize delivery with CloudFront and WAF.

### Tasks:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | **Route 53:**<br>- Alias record to ALB. | 17/11/2025 | 17/11/2025 | |
| 2 | **CloudFront:**<br>- Distribution setup (CDN). | 18/11/2025 | 18/11/2025 | |
| 3 | **WAF:**<br>- Attach to CloudFront to block attacks. | 19/11/2025 | 19/11/2025 | |
| 4 | **Spring Security:**<br>- Implement JWT Filter and AuthenticationManager. | 20/11/2025 | 20/11/2025 | |
| 5 | **Final Config:**<br>- HTTPS Redirect. | 21/11/2025 | 21/11/2025 | |

### 🧠 Extra Knowledge: Stateful vs Stateless Auth
Unlike traditional Session-based authentication (Stateful), I implemented **Stateless Authentication** using JWT (JSON Web Tokens).
* **How it works:** When a user logs in via `AuthenticationService`, the server validates credentials and issues a signed Token.
* **Benefit:** The server doesn't need to store session data in RAM. This allows the Auto Scaling Group to scale the application horizontally without worrying about "Sticky Sessions".

### 💻 Backend Code: Secure Authentication Service
Here is the `login` logic in `AuthenticationService.java`. It relies on Spring Security's `AuthenticationManager` to handle credential validation securely.

**File:** `AuthenticationService.java`
```java
public AccountResponse login(LoginRequest loginRequest) {
    try {
        // 1. Delegate authentication to Spring Security Manager
        // This checks the username/password against the DB (encrypted)
        authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(
                loginRequest.getUsername(),
                loginRequest.getPassword()
        ));
    } catch (BadRequestException e) {
        throw new BadRequestException("Invalid username or password");
    }

    // 2. If valid, retrieve user details
    User user = authenticationRepository.findUserByUsername(loginRequest.getUsername());
    Member profile = memberRepository.findMemberByUser(user);
    
    // 3. Generate JWT Token
    String token = tokenService.generateToken(user);
    
    AccountResponse response = new AccountResponse();
    response.setUsername(user.getUsername());
    response.setRole(user.getRole());
    response.setAddress(profile.getAddress());
    response.setToken(token); // Send token back to client
    return response;
}