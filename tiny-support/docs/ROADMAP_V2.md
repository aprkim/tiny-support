# Tiny Support — V2 Roadmap

## Recording & Summary

### Recording
- **Toggle button**: Currently disabled with "Coming in v2" tooltip
- **Implementation plan**:
  1. Use `MediaRecorder` API on the agent's browser to record the composite audio/video
  2. Or: Use a server-side SFU (e.g., mediasoup) to receive and record all streams
  3. Store recordings in cloud storage (S3, GCS)
  4. Add a recording consent banner that all participants must acknowledge
  5. Indicator visible to all participants when recording is active

### Session Summary
- **Room object placeholders** already in the data model:
  - `summaryStatus: 'none' | 'processing' | 'ready'`
  - `summaryText: string | null`
- **Implementation plan**:
  1. On session end, send the recording (or transcript) to an LLM for summarization
  2. Set `summaryStatus` to `'processing'`
  3. When summary is ready, store in `summaryText` and update status to `'ready'`
  4. Agent can view the summary from a session history page
  5. Summary includes: issue description, steps taken, resolution, follow-up actions

## Other V2 Features

### Database Persistence
- Replace in-memory room store with PostgreSQL or SQLite
- Room history, session logs, and analytics
- Agent session metrics (average wait time, call duration, resolution rate)

### Agent Accounts
- Replace password gate with proper auth (OAuth, SSO)
- Per-agent profiles with availability status
- Agent routing: auto-assign waiting rooms to available agents

### Customer Integration
- API for creating support rooms programmatically
- Embeddable widget for customer-facing apps
- Pre-fill PID from authenticated customer sessions

### Enhanced Screen Sharing
- Annotation tools (draw on shared screen)
- Remote control (with explicit customer consent)
- Multi-monitor selection

### Mobile Support
- Responsive in-call layout for mobile browsers
- Picture-in-picture mode
- Native app wrappers (React Native)

### Chat Enhancements
- Rich text / markdown rendering
- Auto-link URLs
- File/image attachments
- Chat export/transcript

### TURN Server
- Deploy a TURN server (coturn) for NAT traversal
- Ensure connectivity behind corporate firewalls and symmetric NATs

### Quality of Service
- Simulcast video encoding
- Bandwidth estimation and quality adaptation
- Network quality indicators in the UI
- Automatic fallback to audio-only on poor connections
