# Vulnerability Report: Cross-Site Request Forgery (CSRF)

## Vulnerability Type
**Cross-Site Request Forgery (CSRF)**

## Explanation
Cross-Site Request Forgery (CSRF) is a vulnerability that occurs when a web application trusts malicious requests sent from a authenticated user's browser without their knowledge or consent. 

Web browsers automatically include session cookies, authentication headers, and standard identity markers with every request sent to a destination domain, regardless of where the request originated. If an application performs sensitive state-changing actions (such as transferring funds, changing passwords, or modifying account settings) solely based on the presence of these automated session cookies, an attacker can host a malicious page that forces the victim's browser to execute those actions on the target application.

In this specific application, the `/transfer` endpoint accepts state-changing configurations via POST requests without validating the origin of the request or requiring a unique, unpredictable token specific to the user's session.

