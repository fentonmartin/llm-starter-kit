# Authentication Specification — v1.2

*Drafted 2026-04-15. Owner: platform team.*

## 1. Scope

Covers session handling for the web application. Does not cover API tokens or service-to-service
authentication.

## 2. Session creation

On successful password verification the server issues an opaque 256-bit session identifier. The
identifier is random; it carries no user data and is not a JWT.

## 3. Session storage

Sessions are held in Redis, keyed by session identifier. The value is a serialized map of user
id, issue time, and the originating IP.

## 4. Session expiry

### 4.1 Idle timeout

A session is destroyed after 15 minutes without a request.

### 4.2 Absolute TTL

**A session expires 30 minutes after issue regardless of activity.** The client must
re-authenticate; there is no refresh.

## 5. Rejected alternatives

JWTs were considered and rejected. Revocation would have required a denylist, which reintroduces
the shared-state dependency that stateless tokens are meant to remove.
