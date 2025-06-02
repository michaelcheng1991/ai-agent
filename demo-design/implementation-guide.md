# ChatGPT Voice Wrapper - Implementation Guide

build a chat gpt wrapper that supports voice conversation

## existing API: 
- 11ElevnLabs Conversational AI 

## Architecture Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend UI   │◄──►│  Backend API    │◄──►│   OpenAI API    │
│                 │    │                 │    │                 │
│ • Voice Input   │    │ • Chat Logic    │    │ • GPT-4/3.5     │
│ • Voice Output  │    │ • STT/TTS       │    │ • Completions   │
│ • Chat Display  │    │ • Session Mgmt  │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
        │                       │
        ▼                       ▼
┌─────────────────┐    ┌─────────────────┐
│   Speech APIs   │    │    Database     │
│                 │    │                 │
│ • Web Speech    │    │ • Chat History  │
│ • Azure/Google  │    │ • User Sessions │
│ • OpenAI Whisper│    │ • Voice Prefs   │
└─────────────────┘    └─────────────────┘
```

## Technology Stack

### Frontend
- **React/Next.js** - Modern web framework
- **TypeScript** - Type safety
- **Tailwind CSS** - Styling
- **Web Speech API** - Browser native STT/TTS
- **Socket.io Client** - Real-time communication

### Backend
- **Node.js/Express** - API server
- **TypeScript** - Type safety
- **Socket.io** - WebSocket support
- **OpenAI SDK** - ChatGPT integration
- **Multer** - File upload handling
- **Redis** - Session storage
- **MongoDB** - Persistent storage

### External APIs
- **OpenAI API** - ChatGPT responses
- **OpenAI Whisper** - Speech-to-text (fallback)
- **Azure Speech** - Enterprise STT/TTS (optional)
- **Google Cloud Speech** - Alternative STT (optional)

## Implementation Steps

### 1. Project Setup

```bash
# Create project structure
mkdir chatgpt-voice-wrapper
cd chatgpt-voice-wrapper

# Initialize packages
npm init -y
npm install express socket.io openai multer cors dotenv
npm install redis ioredis mongoose
npm install -D typescript @types/node @types/express ts-node nodemon

# Frontend setup (if separate)
npx create-next-app@latest frontend --typescript --tailwind --app
```

### 2. Environment Configuration

Create `.env`:
```env
OPENAI_API_KEY=your_openai_api_key
PORT=3001
REDIS_URL=redis://localhost:6379
MONGODB_URI=mongodb://localhost:27017/chatgpt_wrapper
NODE_ENV=development
```

### 3. Database Models

**Create MongoDB schemas for:**

1. **User Model** (`src/models/User.ts`):
   - Email, name, voice preferences (voice, speed, language)
   - Subscription tier, API usage tracking
   - Embedded API usage array with tokens, cost, request type

2. **ChatSession Model** (`src/models/ChatSession.ts`):
   - Reference to User, title, timestamps
   - Embedded messages array (role, content, isVoice, audioUrl, timestamp)
   - Voice settings (voice, speed, language)
   - Pre-save hook to auto-generate title from first user message

3. **Database Connection** (`src/config/database.ts`):
   - MongoDB connection with error handling
   - Connection event listeners
   - Graceful disconnect function

### 4. Core Backend Services

**Session Service** (`src/services/sessionService.ts`):
- **Hybrid storage**: Try Redis first, fallback to MongoDB
- **Session management**: Get, save, delete sessions with TTL
- **Message handling**: Save individual messages to MongoDB immediately
- **User operations**: Create users, track API usage, get session history
- **Cache strategy**: Background MongoDB sync, immediate Redis updates

**Chat Handler** (`src/handlers/chatHandler.ts`):
- **Process text messages**: Validate session ID, create if not exists
- **OpenAI integration**: Send conversation history, get response
- **Persistence**: Save user and AI messages separately
- **Usage tracking**: Calculate tokens and cost for logged-in users
- **Session management**: Handle ObjectId validation and generation

**Voice Handler** (`src/handlers/voiceHandler.ts`):
- **Audio processing**: Save uploaded audio to temp files
- **Speech-to-text**: Use OpenAI Whisper for transcription
- **Text-to-speech**: Generate audio responses with OpenAI TTS
- **Integration**: Process transcribed text through chat handler
- **Cleanup**: Remove temporary audio files after processing

### 5. Server Setup

**Main Server** (`src/server.ts`):

Essential structure:
- Initialize Express + Socket.io with CORS
- Connect to MongoDB using database config
- Set up middleware (cors, express.json)
- Initialize OpenAI client with API key
- Create handler instances (chat, voice)

Socket events to implement:
- `'text-message'`: Process text chat through chat handler
- `'voice-message'`: Process audio input through voice handler
- `'get-session-history'`: Retrieve chat history from MongoDB
- Error handling for all events with appropriate error messages

### 6. Frontend Implementation

**Main Chat Component** (`components/VoiceChat.tsx`):

**State Management:**
- Messages array with id, content, role, timestamp, isVoice
- Recording state, connection status, input text
- Session ID generation (timestamp-based)
- Socket and MediaRecorder refs

**Socket Integration:**
- Connect to backend on component mount
- Listen for AI responses (text and voice)
- Handle connection status updates
- Error handling and reconnection logic

**Voice Recording:**
- Request microphone permissions with getUserMedia
- Use MediaRecorder API for audio capture
- Collect audio chunks in ondataavailable
- Convert audio blob to buffer for transmission
- Visual feedback for recording state (animate button)

**Audio Playback:**
- Receive audio buffer from backend response
- Create blob URL and play with HTML5 Audio API
- Handle audio loading and playback errors
- Clean up blob URLs after playback

**UI Components:**
- Message display with role-based styling (user vs assistant)
- Input area with textarea and send button
- Voice recording button with visual state
- Connection status indicator
- Voice/text message type indicators

### 7. Advanced Features

**Session Management:**
- **Redis Caching**: Store active sessions with 1-hour TTL
- **MongoDB Persistence**: Permanent storage of complete chat history
- **Background Sync**: Non-blocking database writes for performance
- **Session Recovery**: Load from database if Redis cache miss

**Error Handling:**
- **Retry Logic**: Implement exponential backoff for API failures
- **Graceful Degradation**: Fallback to text if voice fails
- **Network Recovery**: Reconnect sockets and resume sessions
- **Input Validation**: Sanitize all user inputs and file uploads

**Performance Optimizations:**
- **Audio Streaming**: Process audio chunks in real-time
- **Message Pagination**: Load chat history incrementally
- **Connection Pooling**: Efficient database connections
- **Rate Limiting**: Prevent API abuse per user/IP

### 8. Security Considerations

**Input Validation:**
- Validate message length (max 2000 chars), session IDs
- Check audio file types and size limits (25MB OpenAI limit)
- Sanitize user content before storage

**Rate Limiting:**
- Implement per-user voice request limits (50 per 15 minutes)
- IP-based rate limiting for anonymous users
- Track and limit API usage per subscription tier

**Authentication (Future):**
- JWT token validation for protected routes
- User session management with secure cookies
- Permission-based access to premium features

### 9. Testing Strategy

**Unit Tests:**
- Test chat handler with mocked OpenAI responses
- Test session service Redis/MongoDB operations
- Test voice handler audio processing pipeline
- Test database models and validation

**Integration Tests:**
- End-to-end socket communication
- Database operations and data consistency
- OpenAI API integration with real responses
- File upload and audio processing

**Frontend Tests:**
- Component rendering and user interactions
- Socket event handling and state updates
- Audio recording and playback functionality
- Error boundary and fallback behavior

### 10. Deployment Configuration

**Docker Setup:**
- Multi-stage build for Node.js backend
- Separate containers for frontend, backend, Redis, MongoDB
- Environment variable configuration
- Volume mounts for temporary audio files

**Production Considerations:**
- Load balancing for multiple backend instances
- Redis cluster for session storage scalability
- MongoDB replica set for high availability
- CDN for frontend assets and audio files

## Storage Architecture

**Hot Data (Redis)**:
- Active chat sessions (1-hour TTL)
- User authentication tokens
- Rate limiting counters
- Real-time typing indicators

**Persistent Data (MongoDB)**:
- User profiles and voice preferences
- Complete chat history with messages
- API usage tracking and billing
- Session metadata

## Key Implementation Notes

1. **Hybrid Storage**: Redis for speed, MongoDB for persistence
2. **Background Sync**: Non-critical writes happen asynchronously  
3. **ObjectId Handling**: Validate and generate proper MongoDB IDs
4. **Audio Quality**: Use 16kHz, 16-bit encoding for better transcription
5. **Cost Tracking**: Monitor OpenAI API usage per user
6. **Error Recovery**: Handle network failures gracefully
7. **Flexible Schema**: MongoDB handles varying message metadata

## Next Steps

1. **Basic Setup**: Server, database, and OpenAI integration
2. **Frontend Implementation**: React components with Socket.io
3. **Voice Features**: Recording, transcription, and playback
4. **Session Persistence**: Redis caching with MongoDB fallback
5. **Error Handling**: Retry logic and graceful degradation
6. **Testing**: Unit and integration test coverage
7. **Deployment**: Docker containers and production setup
8. **Monitoring**: Logging, metrics, and alerting

This streamlined guide provides the essential architecture and clear implementation steps without overwhelming code examples. 