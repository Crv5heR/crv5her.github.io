+++
title = "Mastering Asp.Net Core Web Api Security"
date = "2024-06-18T13:03:41+03:00"
description = "Protecting ASP.NET Core Web Api Security against attacks"
tags = ["Web API Security", "ASP.NET Core", "C#", "CSRF Protection", "XSS Prevention", "Rate Limiting", "Application Security"]
+++


![C# logo](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d2/C_Sharp_Logo_2023.svg/1200px-C_Sharp_Logo_2023.svg.png)

## Introduction

In today's interconnected world, securing web APIs is paramount for protecting sensitive data and ensuring the integrity of applications. As developers, understanding and implementing robust security measures such as CSRF (Cross-Site Request Forgery) protection, XSS (Cross-Site Scripting) prevention, and rate limiting can safeguard our APIs from malicious attacks. This blog explores how to master these aspects of web API security using C# ASP.NET Core, empowering you to build resilient and secure applications.

## 1. Cross-Site Request Forgery (CSRF) Protection

CSRF attacks exploit the trust a web application has in a user's browser by executing unauthorized commands. To defend against CSRF attacks in ASP.NET Core Web API:

- Use Anti-CSRF Tokens: Implement Anti-CSRF tokens to validate incoming requests. ASP.NET Core provides middleware to generate and validate these tokens automatically.

- Require SameSite Cookies: Configure cookies to be SameSite strict to prevent them from being sent by the browser in cross-origin requests, thus mitigating CSRF risk.

Inside our API we create a middleware for validating the CSRF Token:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAntiforgery(options =>
    {
        options.HeaderName = "X-CSRF-TOKEN";
    });
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseMiddleware<AutoValidateAntiforgeryTokenMiddleware>();
}
```

and in our front-end application we validate the authenticity of every request:

```javascript
const retrieveCSRFToken = () => {
    return axios('/api/v1/csrf')
        .then(data => data.Token)
        .catch(error => console.error('Error retrieving csrf token:', error));
};

retrieveCSRFToken().then(token => {
    axios.post('/api/v1/blogs', requestData, {
        headers: {
            'X-CSRF-Token': token,
        }
    });
});
```

## 2. Cross-Site Scripting (XSS) Prevention

XSS attacks inject malicious scripts into web pages viewed by other users. To prevent XSS vulnerabilities:

- Encode Output: Always encode user input and dynamically generated content before rendering it to prevent script injection.

- Content Security Policy (CSP): Implement CSP headers to restrict the sources from which content can be loaded, reducing the risk of XSS attacks.

- Purify and sanitize the user generated content in React to ensure XSS Protection.

In the backend:
```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.Use(async (context, next) =>
    {
        context.Response.Headers.Add("Content-Security-Policy", "default-src 'self'");
        await next();
    });
}
```

In the frontend:
```javascript
import DOMPurify from 'dompurify';
import React from 'react';

const UnsafeHtmlComponent = ({ htmlContent }) => {
  // Sanitize the HTML content using DOMPurify
  const sanitizedHtml = DOMPurify.sanitize(htmlContent);

  return <div dangerouslySetInnerHTML={{ __html: sanitizedHtml }} />;
};

export default UnsafeHtmlComponent;
```

Rendering the component:
```javascript
import React from 'react';
import UnsafeHtmlComponent from './UnsafeHtmlComponent';

const App = () => {
  const htmlContent = '<p>Hello, <b>world</b>!</p>';

  return (
    <div>
      <h1>Safely Rendered HTML Content</h1>
      <UnsafeHtmlComponent htmlContent={htmlContent} />
    </div>
  );
};

export default App;
```

Additional Considerations
- Contextual Escaping: Whenever possible, prefer using React’s built-in methods like dangerouslySetInnerHTML along with DOMPurify for sanitization.

- Content Sources: Only use dangerouslySetInnerHTML with content that you trust, and always validate and sanitize user-generated content on the server side before rendering it in your React application.

By following these practices, you can effectively sanitize and render HTML content safely in your React applications, protecting against XSS vulnerabilities and ensuring a secure user experience.

## 3. Rate Limiting

Implementing rate limiting helps protect your API from abuse by limiting the number of requests a client can make within a specified time frame. This can prevent denial-of-service (DoS) attacks and ensure fair usage of your API:

- Use Middleware: Implement rate limiting middleware using libraries like AspNetCoreRateLimit to control access rates based on IP address, client ID, or other criteria.

Example configuration for rate limiting middleware in `Startup.cs`:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMemoryCache();

    services.Configure<IpRateLimitOptions>(Configuration.GetSection("IpRateLimiting"));
    services.AddSingleton<IIpPolicyStore, MemoryCacheIpPolicyStore>();
    services.AddSingleton<IRateLimitCounterStore, MemoryCacheRateLimitCounterStore>();
    services.AddSingleton<IRateLimitConfiguration, RateLimitConfiguration>();
    services.AddSingleton<IHttpContextAccessor, HttpContextAccessor>();
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseIpRateLimiting();
}
```

Example configuration for rate limiting in `appsettings.json`
```json
"IpRateLimiting": {
  "EnableEndpointRateLimiting": true,
  "StackBlockedRequests": false,
  "RealIpHeader": "X-Real-IP",
  "ClientIdHeader": "X-ClientId",
  "HttpStatusCode": 429,
  "GeneralRules": [
    {
      "Endpoint": "*",
      "Period": "1m",
      "Limit": 30
    }
  ]
}
```

This configuration should limit the incoming requests to 30 request per minute

## Conclusion

Mastering web API security against CSRF, XSS, and implementing rate limiting in C# ASP.NET Core involves a combination of understanding the attack vectors, leveraging built-in security features, and implementing best practices. By applying these techniques, you can significantly reduce the risk of security breaches and ensure the reliability and trustworthiness of your applications.

Remember, security is an ongoing process. Stay informed about emerging threats and keep your security measures up to date to protect your APIs and users effectively.