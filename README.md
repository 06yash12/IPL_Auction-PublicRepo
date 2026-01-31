# IPL Auction PlayZone

A real-time multiplayer IPL auction simulation platform that recreates the excitement of IPL player auctions. Users can create private auction rooms, invite friends via unique room codes, select their favorite IPL franchises, and compete to build the strongest squad within budget constraints. The platform leverages WebSocket technology for instant bid updates, enforces official IPL team composition rules, and provides AI-powered team analysis to determine the best-balanced squad after the auction concludes.

**Live Demo**: [https://ipl-auction-playzone.onrender.com](https://ipl-auction-playzone.onrender.com)

---

> **Note**: This is a public repository showcasing the project overview and implementation details.  
> **[Complete Documentation](./PROJECT_DOCUMENTATION.md)** | **[Project Screenshots](./PROJECT_IMAGES.md)**

---

## Features

- Real-time multiplayer auctions with WebSocket support
- Up to 10 teams per auction room
- Live bidding with 30-second timers
- Team rules: 25 players max, 8 overseas limit, 8000L budget
- Rebid system for unsold players
- AI-powered team rankings (Google Gemini)
- Google OAuth and traditional authentication
- Spectator mode
- Glassmorphic UI design

## Tech Stack

```
Backend          Flask, Flask-SocketIO, gevent
Real-time        Socket.IO, WebSocket
AI               Google Gemini API
Auth             Google OAuth 2.0, bcrypt
Database         MongoDB (optional), JSON storage
Server           Gunicorn with gevent workers
Frontend         HTML5, CSS3, Vanilla JavaScript
```

## Player Dataset

364+ players with stats extracted from ESPN Cricinfo and IPL records. Includes batting metrics (runs, average, strike rate), bowling metrics (wickets, economy), player roles, base prices, and country info. Data validated and structured in CSV for efficient querying.

## Development Journey

**Ideation** → Designed multiplayer auction system with room-based architecture and IPL rules  
**Backend** → Flask + WebSocket for real-time bidding, room management, auth, rebid logic, AI rankings  
**Frontend** → Responsive templates with vanilla JS for live updates and player stats  
**Design** → Glassmorphic UI with translucent cards, gradients, animations, and IPL branding  
**Testing** → Validated with limited users, then stress-tested with 30+ concurrent users for WebSocket stability

## Quick Start

```bash
# Clone and install
git clone https://github.com/06yash12/IPL_Auction_PlayZone.git
cd ipl-auction-playzone
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env with your API keys

# Run
python app.py
```

Server starts at `http://127.0.0.1:5000`

## License

Educational and entertainment purposes only.
