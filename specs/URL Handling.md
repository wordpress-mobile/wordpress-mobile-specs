# URL Handling

This document specifies how WordPress mobile apps (Android and iOS) should handle URLs.

## Overview

The apps support multiple URL handling mechanisms:

1. **Custom URL Schemes** - `wordpress://` for app-specific deep linking (ie – browsers will never try to handle it)
2. **HTTPS Universal/App Links** - `https://wordpress.com/*` and `https://jetpack.com/*` web URLs that open in the app
3. **QR Code Authentication** - Special handling for QR-based login flows

### User Experience

### Don't stack identical content
When the app opens a universal link, it should check whether the content is already being displayed. If so, it should not present the content again.

### Show progress
When the app opens a universal link, it should display a progress indicator that it's working. If the app later determines that the content cannot be displayed because the user doesn't have access, it should show the error messsage provided by the server.

## Supported URL Schemes

### Custom Scheme: `wordpress://`

Both platforms support the `wordpress://` URL scheme for direct deep linking into CMS features. The Jetpack app supports all `wordpress://` deep links, as well as the `jetpack://` URL scheme for deep linking into Jetpack-only features.

## Universal Links / App Links

Both platforms support opening HTTPS links directly in the app instead of the browser.

### Supported Domains

- `wordpress.com` - Primary domain for both platforms
- `jetpack.com` - Supported on iOS
- `apps.wordpress.com` - Special handling for marketing campaigns and QR codes
- `public-api.wordpress.com` - Email tracking links (Android)
- `*.wordpress.com` - Custom WordPress.com subdomain blogs (for Reader posts)

## Deep Linking

#### Site Creation

- `https://wordpress.com/start` - Site creation flow

#### Posts and Pages

- `https://wordpress.com/post` - New post
- `https://wordpress.com/post/{siteHost}` - New post for site
- `https://wordpress.com/post/{siteHost}/{postId}` - Edit existing post
- `https://wordpress.com/page` - New page
- `https://wordpress.com/page/{domain}` - New page for site

#### Stats

- `https://wordpress.com/stats` - Stats dashboard
- `https://wordpress.com/stats/{timeframe}` - Stats with timeframe (`day`, `week`, `month`, `year`)
- `https://wordpress.com/stats/day/{siteHost}` - Stats for site with day timeframe
- `https://wordpress.com/stats/activity/{domain}` - Activity log
- `https://wordpress.com/stats/subscribers/{domain}` - Subscriber stats

#### Reader

- `https://wordpress.com/read` - Reader main feed
- `https://wordpress.com/discover` - Discover stream
- `https://wordpress.com/read/search` - Reader search
- `https://wordpress.com/tag/{tag_name}` - Tag stream
- `https://wordpress.com/read/feeds/{feedId}/posts/{postId}` - Specific feed post
- `https://wordpress.com/read/blogs/{blogId}/posts/{postId}` - Specific blog post

#### My Site / Site Management

- `https://wordpress.com/pages/{domain}` - Pages list
- `https://wordpress.com/media/{domain}` - Media library
- `https://wordpress.com/comments/{domain}` - Comments
- `https://wordpress.com/sharing/{domain}` - Sharing settings
- `https://wordpress.com/people/{domain}` - People management
- `https://wordpress.com/plugins/{domain}` - Plugins
- `https://wordpress.com/domains/manage` - Domain management
- `https://wordpress.com/me/domains` - Domain management
- `https://wordpress.com/site-monitoring/{domain}` - Site monitoring

#### Notifications

- `https://wordpress.com/notifications` - Notifications list

#### Me / Account

- `https://wordpress.com/me` - Account settings
- `https://wordpress.com/me/account` - Account details
- `https://wordpress.com/me/notifications` - Notification settings

#### Special Routes

**App Banners:**
- `https://apps.wordpress.com/get#<fragment>` - App banner campaigns

**QR Code Authentication:**
- `https://apps.wordpress.com/get?campaign=login-qr-code`
- iOS uses fragment-based parsing: `#qr-code-login?token=...&data=...`

**QR Code Media:**
- `https://apps.wordpress.com/get?campaign=qr-code-media` - QR media library access

**Email Tracking / Marketing Redirects:**
- `public-api.wordpress.com/mbar/*` - Mobile email tracking links (Android supports it)
- `public-api.wordpress.com/bar/*` - Email tracking links (Android supports it)



## Error Handling

### Invalid or Unsupported URLs

Unsupported URLs should be forwarded back to the browser.

#### Site Not Found

If the app recieves a deep link for a site it's not authorized to access (for instance, a private WP.com subdomain), it should display the error message provided by the server.

#### Post Not Found

The the app receives a deep link for a post that doesn't exist (for instance, it's a draft or was deleted), it should display the error message provided by the server.

### Login Required

Both platforms intercept deep links that require authentication:

1. Check if user is logged in
2. Present the WP.com login flow
3. On Success, redirect the user to their intended destination.

## Analytics and Tracking

Every deep link that's received by the app (whether it's handled or passed back to a browser) should track the following:

**Event Name**
`(wp|jp)(android|ios)_deep_linked`

**Event Parameters**
- `url`: The format of the handled link (ex: `/:post_year/:post_month/:post_day/:post_name` or `/me/account`)
- `source_info`: The source of the deep link, if known (ex: `calypso-banner`, `link`, `email`)
- `hostname`: The `host` portion of the URL: (ex: `wordpress.com`, `public-api.wordpress.com`, `mysite.wordpress.com`)
- `scheme`: The `scheme` portion of the URL: (ex: `http`, `https`, `wordpress`, `jetpack`)
- `handled`: Was that app able to handle the link?

## Testing Recommendations

### Test Cases

1. **Custom Scheme URLs**
   - Test all `wordpress://` action types
   - Test with and without parameters
   - Test with invalid site/post IDs
   - Test with site IDs that the current user does not have access to

2. **Universal/App Links**
   - Test all `wordpress.com` path patterns
   - Test with valid and invalid domains
   - Test custom domain blog URLs

3. **Authentication**
   - Test deep links while logged out
   - Verify post-login navigation completes
   - Test OAuth callback handling

4. **Error Handling**
   - Test with non-existent sites
   - Test with non-existent posts
   - Test with malformed URLs
   - Verify appropriate error messages

5. **Analytics**
  - When running any of the tests above, validate that the event shows up in Tracks.

### Platform-Specific Tests

**Android:**
- Test intent filter matching via `adb`
- Verify App Links verification with `adb shell pm get-app-links`
- Test activity aliases

**iOS:**
- Test Universal Links using `xcrun simctl openurl booted ${URL}`
