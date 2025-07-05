# AudioFetch

A FastAPI-based web application for downloading audio from websites with real-time progress tracking via WebSockets

## Features

### Core Features

- **Browser Mode**: Stream downloads directly to browser without server storage
- **Server Mode**: Password-protected downloads saved to server (for admin use)
- **Real-time Progress**: WebSocket-based live updates with detailed status
- **Auto-detection**: Automatically detects audio players on websites
- **Parallel Downloads**: Configurable workers for faster downloads (async)
- **Multiple Formats**: Supports MP3, M4A, AAC, OGG, OPUS, WebM, WAV, FLAC
- **Smart Download Control**: Prevents duplicate downloads across multiple tabs
- **Job Tracking**: Unique job IDs for all downloads with persistent tracking

### Supported Audio Players

- ✅ Direct audio tags
- ✅ Plyr.js audio players
- ❌ Howler.js (planned)
- ❌ MediaElement.js (planned)
- ❌ Video.js (planned)

## Installation

1. Clone the repository:

```bash
git clone <repository-url>
cd download_audio
git checkout containerized
```

2. Install dependencies:

```bash
pip install -r requirements.txt
```

3. Set environment variables (optional):

```bash
export ADMIN_PASSWORD="your-secure-password"  # Default: admin123
export SECRET_KEY="your-secret-key"           # For JWT tokens
```

## Running the Application

### Using Docker

1. Copy the example environment file and fill in your actual values:

   ```bash
   cp .env.example .env
   # Edit .env to set ADMIN_PASSWORD, SECRET_KEY, PORT, etc.
   ```

2. Build and start the containers:
   ```bash
   docker compose up --build -d
   ```

## Usage Guide

### Browser Mode (No Authentication Required)

1. Open `http://localhost:<YOUR_ENV_PORT_NUMBER>` in your browser
2. Enter the URL of the page containing audio
3. Leave "Download Mode" as "Browser"
4. Click "Start Download"
5. Files will stream directly to your browser as a ZIP

### Server Mode (Authentication Required)

1. Double Click AudioFetch title.
2. Click on "Admin Login" and enter the admin password
3. Select "Server" as the download mode
4. Start the download - files will be saved on the server
5. Access saved downloads from the "Server Downloads" section

### CLI Usage (Original Functionality)

The original command-line interface is still available:

```bash
python main.py <url> [name] [--plugin <plugin_name>] [--workers <num>]
```

Example:

```bash
python main.py https://example.com/audiobook my-audiobook --plugin plyr --workers 10
```

## API Endpoints

### Public Endpoints

- `GET /` - Web interface
- `POST /api/download` - Start a new download
- `GET /api/status/{job_id}` - Get job status
- `GET /api/jobs` - List all jobs
- `DELETE /api/jobs/{job_id}` - Delete completed job
- `POST /api/jobs/{job_id}/cancel` - Cancel active job
- `GET /api/stream/{job_id}` - Stream browser mode download
- `WebSocket /ws` - Real-time updates

### Protected Endpoints (Require Authentication)

- `POST /api/auth/login` - Login with admin password
- `GET /api/downloads` - List server downloads
- `DELETE /api/downloads/{name}` - Delete server download
- `GET /api/downloads/{name}/zip` - Download as ZIP

## Architecture

### Backend (Python FastAPI)

- WebSocket support for real-time updates
- JWT-based authentication for server mode
- Fully async/await download operations using aiohttp
- Background tasks for download processing
- True streaming ZIP generation (no buffering)
- Server-side download coordination for multiple tabs

### Frontend

- Vanilla JavaScript with WebSocket client
- Real-time progress bars with detailed status
- localStorage synchronization between tabs
- No framework dependencies

### Download Flow

#### Browser Mode

1. User submits URL
2. Backend analyzes page and detects audio player
3. Extracts audio track information
4. Creates streaming ZIP response
5. Downloads and streams files directly to browser
6. No server storage required
7. Smart tab coordination prevents duplicate downloads

#### Server Mode

1. User authenticates with admin password
2. Backend downloads files asynchronously to server
3. Real-time progress updates via WebSocket
4. Files stored in `downloads/` directory
5. User can download as ZIP later or manage files

## Configuration

### Environment Variables

- `ADMIN_PASSWORD`: Password for server mode (default: "admin123")
- `SECRET_KEY`: JWT secret key (default: auto-generated)
- `ACCESS_TOKEN_EXPIRE_MINUTES`: Token expiry (default: 1440)

### Download Settings

- Workers: 1-20 parallel downloads (default: 5)
- Timeout: 30 seconds per request
- Chunk size: 8KB for streaming

## Logging

Logs are written to:

- Console output (INFO level)
- `audiofetch.log` file (DEBUG level)

Log format includes:

- Timestamp
- Log level
- Job ID (when applicable)
- Progress updates
- Error details

## Security Considerations

1. **Authentication**: Server mode requires password
2. **CORS**: Configure for production deployment
3. **File paths**: Sanitized to prevent directory traversal
4. **Rate limiting**: Built-in rate limits protect against abuse
   - Login endpoint: 3 attempts per minute
   - Download endpoint: 5 requests per minute
5. **HTTPS**: Use reverse proxy with SSL in production

## Troubleshooting

### WebSocket Connection Issues

- Check firewall allows WebSocket connections
- Ensure reverse proxy forwards WebSocket headers
- Check browser console for errors

### Download Failures

- Check `audiofetch.log` for details
- Verify URL is accessible
- Check audio player is supported
- Ensure sufficient disk space (server mode)

## Potential Future Enhancements

- [ ] Support more audio players (Howler.js, Video.js)
- [ ] Browser extension
- [ ] Playlist support
- [ ] Metadata extraction and tagging
