# Bug Explanation

## What was the bug?

The `HttpClient.request()` method failed to refresh the OAuth2 token when `oauth2Token` was a plain object (not `null` and not an `OAuth2Token` instance). This caused API requests to proceed without proper authentication headers when the token state was a plain object.

## Why did it happen?

The refresh condition in `httpClient.ts` (lines 21-24) checked:
1. `!this.oauth2Token` - false for plain objects (they're truthy)
2. `this.oauth2Token instanceof OAuth2Token && this.oauth2Token.expired` - false for plain objects (not instances)

When `oauth2Token` was a plain object like `{ accessToken: "stale", expiresAt: 0 }`, neither condition was true, so the token wasn't refreshed. The code only handled `null` tokens and expired `OAuth2Token` instances, missing the plain object case.

## Why does my fix solve it?

The fix adds `!(this.oauth2Token instanceof OAuth2Token)` to the condition. Now the token refreshes when:
- It's `null`
- It's not an `OAuth2Token` instance (plain object case)
- It's an expired `OAuth2Token` instance

This ensures all invalid token states trigger a refresh, including plain objects that shouldn't be used directly.

## One realistic case / edge case my tests still don't cover

The tests don't cover the scenario where `refreshOAuth2()` fails or throws an error. If the token refresh API call fails, the code would still attempt to use the invalid token state, potentially causing authentication errors in subsequent requests. A production system should handle refresh failures gracefully, possibly with retry logic or error propagation.

