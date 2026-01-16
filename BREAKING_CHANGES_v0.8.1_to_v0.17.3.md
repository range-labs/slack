# Breaking Changes: v0.8.1 → v0.17.3-range.2

This document outlines breaking changes when upgrading from v0.8.1 (2020) to v0.17.3-range.2 (2025).

## Summary

**Total Upstream Commits**: 823
**Major Breaking Changes**: 6 categories
**Deprecations**: Multiple APIs deprecated/removed

---

## 🔴 CRITICAL: Removed Deprecated APIs

### channels.*, groups.*, im.* Methods (Removed in v0.9.0, 2020)

Slack deprecated these APIs for new apps (June 2020) and removed support for existing apps (February 2021). The library removed these methods in favor of the `conversations.*` API.

**Removed Files:**
- `channels.go` - All channel-specific methods
- `groups.go` - All group-specific methods
- `im.go` - All instant message-specific methods

**Migration Path:**
Use the `conversations.*` API methods instead:

```go
// ❌ OLD (REMOVED)
api.GetChannels(false)
api.ArchiveChannel(channelID)
api.GetChannelInfo(channelID)

// ✅ NEW
api.GetConversations(&GetConversationsParameters{})
api.ArchiveConversation(channelID)
api.GetConversationInfo(&GetConversationInfoInput{ChannelID: channelID})
```

**Reference:** https://api.slack.com/changelog/2020-01-deprecating-antecedents-to-the-conversations-api

---

## ⚠️ Breaking Signature Changes

### 1. NewInputBlock - Added `hint` Parameter (v0.10.0, 2020)

**Change:** Added required `hint` parameter to `NewInputBlock()` function.

```go
// ❌ OLD (v0.8.1)
NewInputBlock(blockID string, label *TextBlockObject, element BlockElement)

// ✅ NEW (v0.10.0+)
NewInputBlock(blockID string, label, hint *TextBlockObject, element BlockElement)
```

**Migration:**
```go
// If you don't need a hint, pass nil
inputBlock := NewInputBlock(
    blockID,
    label,
    nil, // hint
    element,
)
```

**Commit:** `369f07a`

---

### 2. AppHomeOpenedEvent.View - Changed to Pointer (v0.14.0, 2024)

**Change:** `View` field changed from value to pointer type.

```go
// ❌ OLD
type AppHomeOpenedEvent struct {
    View slack.View
}

// ✅ NEW
type AppHomeOpenedEvent struct {
    View *slack.View  // Now a pointer, can be nil
}
```

**Migration:**
```go
// Check for nil before accessing
if event.View != nil {
    // Use event.View
}
```

**Commit:** `f39c68c` (May 2024)
**PR:** #1424

---

### 3. UploadToURL - Changed from Private to Public (v0.14.0, 2024)

**Change:** Function renamed from `uploadToURL` (private) to `UploadToURL` (public).

**Impact:** If you were using reflection or workarounds to access the private function, update to use the public API.

```go
// ✅ NEW - Now officially supported
api.UploadToURL(uploadURL, reader, filename)
```

**Commit:** `edb966b` (May 2024)
**PR:** #1422

---

### 4. GetConversationsParameters.ExcludeArchived - Type Change

**Change:** Field type changed from `string` to `bool`.

```go
// ❌ OLD
params := GetConversationsParameters{
    ExcludeArchived: "true",
}

// ✅ NEW
params := GetConversationsParameters{
    ExcludeArchived: true,
}
```

**Commit:** `e626a83`

---

## 🗑️ Deprecated Features

### 1. Legacy Workflows API (Deprecated in v0.13.0, 2024)

**Status:** Fully removed from the library (March 2024)

**Removed:**
- `workflow_step.go`
- `workflow_step_execute.go`
- All workflow step events from `slackevents`
- Example: `examples/workflow_step/`

**Migration:** Use the new Workflow Triggers API instead:
- `workflows_triggers.go` - New API methods added
- See: https://api.slack.com/automation/triggers

**Commit:** `a454f77`
**PR:** #1350

---

### 2. files.upload API (Deprecated, 2023)

**Status:** Marked as deprecated, still functional but should be replaced.

**Migration:** Use `files.getUploadURLExternal` + `files.completeUploadExternal` workflow:

```go
// ❌ OLD (Deprecated)
api.UploadFile(params)

// ✅ NEW
// 1. Get upload URL
uploadURL, fileID, err := api.GetUploadURLExternal(params)
// 2. Upload file to URL
err = api.UploadToURL(uploadURL, reader, filename)
// 3. Complete the upload
err = api.CompleteUploadExternal(fileID, uploads)
```

**Commit:** `5094cdf`
**PR:** #1300

---

### 3. RTM (Real-Time Messaging) API

**Status:** Deprecated by Slack in favor of Socket Mode. Still supported but not recommended for new applications.

**Migration:** Use Socket Mode for real-time events:

```go
// ✅ Recommended: Socket Mode
client := socketmode.New(api)
```

**Reference:** https://api.slack.com/rtm

---

## 📝 Other Notable Changes

### 1. Go Version Requirement

**Change:** Minimum Go version increased from 1.16 → 1.24

**Impact:** Ensure your build environment supports Go 1.24+ (or 1.20+ which should be compatible).

---

### 2. Dependency Changes

**Removed:**
- `github.com/pkg/errors` - Replaced with standard library `errors`

**Updated:**
- `github.com/gorilla/websocket` - 1.4.2 → 1.5.3
- `github.com/stretchr/testify` - 1.2.2 → 1.11.1
- `github.com/go-test/deep` - 1.0.4 → 1.1.1

**Added:**
- `github.com/google/go-cmp` - v0.7.0 (for testing)

---

### 3. Removed Vendor Directory

**Change:** The `vendor/` directory has been removed. Use Go modules for dependency management.

**Migration:** Ensure your project uses `go.mod`:
```bash
go mod download
```

---

### 4. Event Type Changes

**EventTimestamp Field Type:**
Changed from `json.Number` to `string` in multiple event types for consistency.

**Affected Events:**
- `AppMentionEvent`
- `AppHomeOpenedEvent`
- Multiple others in `slackevents/`

---

### 5. New Required Fields

**TimePickerBlockElement:**
- Added optional `Timezone` parameter (v0.16.0)

**SetAssistantThreadsStatus:**
- Added optional `loading_messages` parameter (v0.17.0)

---

## 🔧 Testing & Validation

After upgrading, ensure you:

1. **Run all tests:**
   ```bash
   go test -race ./...
   ```

2. **Check for compilation errors:**
   ```bash
   go build ./...
   ```

3. **Review deprecation warnings:**
   ```bash
   golangci-lint run
   ```

4. **Verify API calls:**
   - Test with Slack API in development environment
   - Monitor for `warning` responses (use `OptionOnWarning` callback)

---

## 📚 Resources

- **Slack API Changelog:** https://api.slack.com/changelog
- **Conversations API Migration:** https://api.slack.com/changelog/2020-01-deprecating-antecedents-to-the-conversations-api
- **Workflow Triggers:** https://api.slack.com/automation/triggers
- **Socket Mode:** https://api.slack.com/apis/connections/socket

---

## ✅ Custom Range Features (Preserved)

The following **Range-specific features** have been preserved and are fully compatible:

- ✅ `OptionOnWarning` callback system
- ✅ `Warning` field in `SlackResponse`
- ✅ `warner` interface and `Warning` type
- ✅ Warning callbacks in `postMethod` and `getMethod`

**No breaking changes** to Range's custom functionality.

---

*Last Updated: January 2025*
*Range Fork Version: v0.17.3-range.2*
