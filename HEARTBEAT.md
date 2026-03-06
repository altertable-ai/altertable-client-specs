# Maintainer Heartbeat

# Schedule: Every 2 minutes

# Priority: High

## Routine

1. **Check notifications**: `gh api notifications` to ensure nothing is missed
2. **Run maintainer-routine**: Use the `maintainer-routine` skill to identify actionable work
3. **React using skills**:
   - `triage-issues` - for issues
   - `review-pr` - for PR reviews
   - `sync-repos` - for cross-repo consistency
   - `build-lakehouse-sdk` - for Lakehouse SDKs
   - `build-product-analytics-sdk` - for Product Analytics SDKs
   - `build-readme` - for READMEs
   - `build-http-sdk` - HTTP client reference

If new actionable items are found (bug reports, review comments, CI failures), address them immediately.
Reply HEARTBEAT_OK if nothing actionable.
