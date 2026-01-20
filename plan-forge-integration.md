# Plan: Forge Integration for tuicr (GitHub Example)

## Overview
Extend tuicr to fetch and push code reviews from/to forge platforms, starting with GitHub as the reference implementation.

## GitHub Review Structure

### Review Object
- **Unique ID**: Each review has an `id` and `node_id`
- **State**: `APPROVED`, `CHANGES_REQUESTED`, `COMMENT`, `DISMISSED`, `PENDING`
- **Metadata**: `user`, `submitted_at`, `commit_id`, `body` (overall summary)
- **Comments**: Multiple inline comments associated via `pull_request_review_id`

### Comment Object
- **Location**: `path`, `line` (or `position` in diff)
- **Content**: `body`, `user`, `created_at`
- **Association**: `pull_request_review_id` (null for standalone comments)
- **Threading**: `in_reply_to_id` for replies

## Fetching Reviews from GitHub

### Approach
Use GitHub REST API via `gh api` command:

```bash
# 1. Get all reviews for a PR
gh api repos/{owner}/{repo}/pulls/{pr}/reviews

# 2. Get all inline comments (across all reviews)
gh api repos/{owner}/{repo}/pulls/{pr}/comments

# 3. Optional: Get comments for specific review
gh api repos/{owner}/{repo}/pulls/{pr}/reviews/{review_id}/comments
```

### Data Mapping
```
GitHub Review → tuicr Review Session
├── review.id → session identifier
├── review.state → review status
├── review.body → overall summary/notes
└── comments[] → inline annotations
    ├── path + line → file location
    ├── body → comment text
    └── user.login → reviewer
```

### Implementation Notes
- Comments can be grouped by `pull_request_review_id` to reconstruct reviews
- Standalone comments (null `pull_request_review_id`) are individual annotations

## Pushing Reviews to GitHub

### Recommended API: Reviews Endpoint
**Endpoint**: `POST /repos/{owner}/{repo}/pulls/{pr}/reviews`

**Advantages**:
- ✅ Batch submission (all comments in one request)
- ✅ Set review state (APPROVE/REQUEST_CHANGES/COMMENT)
- ✅ Add overall summary
- ✅ Atomic operation

### Payload Format
```json
{
  "body": "Overall review summary from tuicr",
  "event": "COMMENT|APPROVE|REQUEST_CHANGES",
  "commit_id": "abc123...",
  "comments": [
    {
      "path": "src/file.rs",
      "position": 10,
      "body": "Comment from tuicr review"
    }
  ]
}
```

### Position Calculation
- **`position`**: Lines down from first `@@` hunk header in diff
- Requires parsing the PR diff to calculate positions from line numbers
- Alternative: Use Comments API with absolute `line` numbers (but no batch support)

### Alternative: Comments API (Individual)
**Endpoint**: `POST /repos/{owner}/{repo}/pulls/{pr}/comments`

**Use when**:
- Adding single comments incrementally
- Replying to existing comments

**Payload**:
```json
{
  "body": "Comment text",
  "commit_id": "abc123...",
  "path": "src/file.rs",
  "line": 42,
  "side": "RIGHT"
}
```

## Implementation Strategy

### Phase 1: Fetch Support
1. Add forge abstraction layer in tuicr
2. Implement GitHub fetcher using REST API
3. Parse review/comment JSON → tuicr data structures
4. Handle authentication (requires `repo` scope token)

### Phase 2: Push Support
1. Generate review payload from tuicr session
2. Calculate diff positions OR use line-based comments
3. Submit via Reviews API (batch) or Comments API (individual)
4. Handle API errors and rate limits

### Phase 3: Generalization
1. Define forge trait/interface
2. Extract GitHub-specific logic
3. Prepare for other forges (GitLab, Gitea, etc.)

## Authentication Requirements
- GitHub: Personal access token with `repo` scope or fine-grained token with PR write permissions
- Available via `gh auth` (GitHub CLI handles this)

## Open Questions
1. How should tuicr represent review states internally?
2. Should tuicr calculate diff positions or prefer line-based comments?
3. How to handle comment threading/replies?
4. Should reviews be imported as read-only or editable sessions?
5. Conflict handling: what if remote review changes while editing locally?

## Related Resources
- GitHub Docs: [REST API for Pull Request Reviews](https://docs.github.com/en/rest/pulls/reviews)
- GitHub Docs: [REST API for Pull Request Review Comments](https://docs.github.com/en/rest/pulls/comments)
