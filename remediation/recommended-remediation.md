# Recommended Remediation — IDOR/BOLA

## 1. Overview

The identified vulnerability is caused by insufficient server-side object-level authorization within the transaction-history functionality.

The application accepts a client-controlled account identifier but does not adequately verify whether the authenticated user is authorized to access the corresponding account.

The primary remediation is to enforce authorization checks on the server before returning transaction information.

---

## 2. Server-Side Authorization

The server should determine the identity of the authenticated user from the active session or authorization mechanism.

It should then verify that the requested account belongs to, or is otherwise accessible by, that user.

The authorization decision must be performed server-side.

A client-supplied account identifier must never be treated as proof that the user is authorized to access the account.

---

## 3. Recommended Authorization Flow

A secure implementation should follow this general process:

```text
Authenticated Request
        ↓
Identify Authenticated User
        ↓
Identify Requested Account
        ↓
Check User's Access Rights
        ↓
 ┌─────────────────────┐
 │ Authorized to access?│
 └─────────────────────┘
        ↓          ↓
       YES         NO
        ↓           ↓
 Return Data    Deny Request
                (403 Forbidden)
```

---

## 4. Ownership Validation

Before returning transaction information, the application should verify the relationship between the authenticated user and the requested account.

Conceptually:

```text
authenticated_user = current_session.user

requested_account = requested_object.account

if user_is_authorized(authenticated_user, requested_account):
    return transaction_data
else:
    deny_access()
```

The exact implementation will depend on the application's architecture and authorization model.

---

## 5. Avoid Client-Side Authorization

Authorization decisions should not be based solely on:

* Account IDs supplied by the client.
* Hidden form fields.
* URL parameters.
* Client-side JavaScript.
* Values stored only in browser-controlled data.

These values can be modified by the user and therefore cannot independently establish authorization.

---

## 6. Consistent Authorization Enforcement

Authorization checks should be applied consistently across all endpoints that access sensitive objects.

This should include, where applicable:

* Transaction history.
* Account details.
* Account statements.
* Payment information.
* Beneficiary information.
* User profiles.
* Other sensitive financial resources.

Fixing only one endpoint may leave the same authorization weakness exposed elsewhere in the application.

---

## 7. Error Handling

When an authenticated user attempts to access an object they are not authorized to access, the server should reject the request.

For example:

```http
HTTP/1.1 403 Forbidden
```

The response should not disclose the requested object's sensitive information.

---

## 8. Logging and Monitoring

The application should consider logging authorization failures and monitoring repeated attempts to access unauthorized objects.

Repeated requests involving different account identifiers may indicate attempted object enumeration or authorization abuse.

Logging should avoid storing unnecessary sensitive financial information or authentication secrets.

---

## 9. Testing Requirements

Authorization controls should be tested using multiple user accounts.

Security testing should verify that:

* User A can access User A's resources.
* User A cannot access User B's resources.
* User B cannot access User A's resources.
* Unauthorized object identifiers are rejected.
* Authorization remains enforced regardless of how the object identifier is supplied.

---

## 10. Retesting

After implementing the remediation, repeat the original proof-of-concept test.

The same account identifier manipulation that previously returned another user's transaction information should now result in an authorization failure.

### Expected Result

```text
User A
  ↓
Requests User B's Account
  ↓
Server Authorization Check
  ↓
Access Denied
  ↓
403 Forbidden
```

User B's transaction information must not be returned.

---

## 11. Conclusion

The IDOR/BOLA vulnerability can be addressed by implementing robust server-side object-level authorization.

The key security requirement is that access to an account or transaction object must be determined by the authenticated user's permissions rather than by the account identifier supplied by the client.

Authorization checks should be applied consistently across all sensitive application functionality and verified through security testing after implementation.
