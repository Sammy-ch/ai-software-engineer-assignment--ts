# Explanation

## What was the bug?

When `oauth2Token` was a plain object instead of a proper `OAuth2Token` instance, the client skipped the token refresh and sent API requests without an Authorization header.

## Why did it happen?

The refresh condition relied on `!this.oauth2Token` to catch missing tokens, but plain objects aren't null or undefined so they slip past that. The expiry check only ran for `instanceof OAuth2Token`, which plain objects also fail. A plain object token ended up skipping both paths entirely.

## Why does the fix solve it?

I changed the condition to `!(this.oauth2Token instanceof OAuth2Token) || this.oauth2Token.expired`, so we refresh unless we have an actual non-expired `OAuth2Token` instance. Plain objects and null both get caught by the `instanceof` check now.

## One edge case the tests don't cover

The fix would break if `OAuth2Token` ends up loaded twice, like from a duplicate in `node_modules` or separate bundle chunks. The token would be valid but `instanceof` returns false, so you'd get a refresh on every request without any obvious reason why.
