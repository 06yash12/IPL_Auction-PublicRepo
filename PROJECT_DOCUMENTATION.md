# IPL Auction PlayZone - Complete Project Documentation

<div align="center">

![IPL Auction Landing Page](./images/Landing%20Page.png)

**A Real-Time Multiplayer IPL Auction Simulation Platform**

*Experience the thrill of IPL auctions with real-time bidding, team management, and AI-powered analytics*

</div>

---

## Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Core Features](#core-features)
- [Database Schema](#database-schema)
- [WebSocket Events](#websocket-events)
- [AI Integration](#ai-integration)
- [Security](#security)
- [Deployment](#deployment)
- [Technical Challenges & Solutions](#technical-challenges--solutions)
- [Performance Metrics](#performance-metrics)
- [Code Structure](#code-structure)

---

## Overview

IPL Auction PlayZone is a sophisticated real-time multiplayer platform that simulates the IPL player auction experience. Built with Flask and Socket.IO, it supports concurrent auctions with up to 10 teams per room, enforces official IPL rules, and provides AI-powered team analysis.

### Key Highlights

- **Real-Time Bidding**: WebSocket-based instant bid updates across all clients
- **Room-Based Architecture**: Isolated auction rooms with unique codes
- **IPL Rule Enforcement**: Budget limits, squad composition, overseas player caps
- **AI Analysis**: Google Gemini-powered team strength evaluation
- **Scalable Design**: Handles 30+ concurrent users with gevent workers

---

## System Architecture

```
┌────────────────────────────────────────────────────────────┐
│                        Client Layer                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │ Browser 1│  │ Browser 2│  │ Browser 3│  │ Browser N│    │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘    │
└───────┼─────────────┼─────────────┼─────────────┼──────────┘
        │             │             │             │
        └─────────────┴─────────────┴─────────────┘
                      │ WebSocket
        ┌─────────────┴─────────────────────────────┐
        │         Flask-SocketIO Server             │
        │  ┌──────────────────────────────────────┐ │
        │  │   Room Management & Event Handlers   │ │
        │  └──────────────────────────────────────┘ │
        │  ┌──────────┐  ┌──────────┐  ┌─────────┐  │
        │  │ Auth     │  │ Bidding  │  │ AI      │  │
        │  │ Service  │  │ Logic    │  │ Service │  │
        │  └──────────┘  └──────────┘  └─────────┘  │
        └───────────────────┬───────────────────────┘
                            │
        ┌───────────────────┴───────────────────────┐
        │          Data Persistence Layer           │
        │  ┌──────────┐  ┌──────────┐  ┌─────────┐  │
        │  │ MongoDB  │  │ JSON     │  │ CSV     │  │
        │  │(Optional)│  │ Storage  │  │ Players │  │
        │  └──────────┘  └──────────┘  └─────────┘  │
        └───────────────────────────────────────────┘
```

---

## Core Features

### 1. Room Management System

**Implementation Overview:**
```python
# Pseudocode for Room Creation
def create_auction_room(user_id, room_name):
    room_code = generate_unique_code(6)  # e.g., "ABC123"
    
    room_data = {
        'room_id': room_code,
        'host': user_id,
        'teams': [],
        'current_player': None,
        'auction_state': 'waiting',
        'created_at': timestamp()
    }
    
    store_room(room_data)
    return room_code
```

**Key Features:**
- Unique 6-character room codes
- Host controls (start auction, kick users)
- Auto-cleanup after 24 hours of inactivity
- Room persistence across server restarts

![Room Creation or Home Page](./images/Room%20Creation%20or%20home%20page.jpg)
*Create private auction rooms with unique codes and invite friends to join the bidding war*

---

### 2. Real-Time Bidding Engine

**WebSocket Event Flow:**
```
Client                    Server                    Other Clients
  │                         │                            │
  ├─ emit('place_bid') ────>│                            │
  │                         ├─ Validate bid              │
  │                         ├─ Update room state         │
  │                         ├─ broadcast('bid_update')──>│
  │<─ emit('bid_success')───┤                            │
  │                         │                            │
```

**Bid Validation Logic:**
```python
# Pseudocode for Bid Validation
def validate_bid(team, player, bid_amount):
    checks = {
        'budget': team.remaining_budget >= bid_amount,
        'squad_size': len(team.players) < 25,
        'overseas': player.is_overseas and team.overseas_count < 8,
        'minimum': bid_amount >= player.base_price,
        'increment': bid_amount > current_highest_bid
    }
    
    if all(checks.values()):
        return True, "Bid accepted"
    else:
        return False, get_error_message(checks)
```

**Timer System:**
- 30-second countdown per player
- Auto-sold to highest bidder on timeout
- Visual countdown on all clients
- Server-side timer validation

![Start Auction](./images/Start%20Auction.png)
*Host initiates the auction with all teams ready - the countdown begins for the first player*

---

### 3. Team Composition Rules

**IPL Official Rules Enforced:**

| Rule | Limit | Implementation |
|------|-------|----------------|
| Total Players | 25 max | Counter check before bid acceptance |
| Overseas Players | 8 max | Country-based validation from CSV |
| Total Budget | ₹8000L | Real-time budget tracking |
| Minimum Squad | 18 players | Warning system during auction |

**Squad Validation:**
```python
# Pseudocode for Squad Validation
class TeamValidator:
    def validate_purchase(self, team, player, amount):
        # Budget check
        if team.budget - amount < 0:
            return False, "Insufficient budget"
        
        # Squad size check
        if len(team.squad) >= 25:
            return False, "Squad full"
        
        # Overseas check
        if player.country != "India":
            if team.overseas_count >= 8:
                return False, "Overseas limit reached"
        
        return True, "Valid purchase"
```

---

### 4. Rebid System

**Unsold Player Management:**

When a player receives no bids:
1. Player marked as "UNSOLD"
2. Added to rebid queue
3. Re-auctioned after all players
4. Base price reduced by 25%

```python
# Pseudocode for Rebid Logic
def handle_unsold_player(player):
    player.status = "UNSOLD"
    player.base_price = player.base_price * 0.75
    rebid_queue.append(player)
    
    if all_players_auctioned():
        start_rebid_round(rebid_queue)
```

---

### 5. AI-Powered Team Rankings

**Google Gemini Integration:**

```python
# Pseudocode for AI Analysis
def analyze_team_strength(team):
    prompt = f"""
    Analyze this IPL team:
    
    Squad: {team.players}
    Budget Used: {team.spent}/{team.total_budget}
    
    Evaluate:
    1. Batting strength (depth, power hitters, anchors)
    2. Bowling attack (pace, spin, death bowling)
    3. All-rounders balance
    4. Team composition
    5. Overall rating (1-10)
    
    Provide JSON response with scores and analysis.
    """
    
    response = gemini_api.generate(prompt)
    return parse_ai_response(response)
```

**Ranking Criteria:**
- Batting depth and variety
- Bowling balance (pace/spin)
- All-rounder presence
- Budget utilization efficiency
- Squad composition balance

![AI Rankings](./images/AI%20Rankings.png)
*Google Gemini analyzes each team's composition and provides detailed rankings with strengths and weaknesses*

---

## Database Schema

### User Collection
```json
{
  "_id": "user_12345",
  "username": "cricket_fan",
  "email": "user@example.com",
  "password_hash": "bcrypt_hash",
  "google_id": "optional_google_id",
  "created_at": "2026-01-31T10:00:00Z",
  "auctions_participated": 15
}
```

### Room Collection
```json
{
  "_id": "ABC123",
  "host_id": "user_12345",
  "room_name": "Friends IPL Auction",
  "status": "active",
  "teams": [
    {
      "team_id": "team_001",
      "user_id": "user_12345",
      "franchise": "Mumbai Indians",
      "budget": 8000,
      "spent": 3500,
      "players": [...],
      "overseas_count": 4
    }
  ],
  "current_player_index": 45,
  "auction_state": "bidding",
  "created_at": "2026-01-31T10:00:00Z",
  "last_activity": "2026-01-31T11:30:00Z"
}
```

### Player Data (CSV)
```csv
name,role,country,base_price,batting_avg,strike_rate,wickets,economy
Virat Kohli,Batsman,India,200,50.5,138.2,0,0
Jasprit Bumrah,Bowler,India,150,8.5,105.3,150,7.2
```

---

## WebSocket Events

### Client → Server Events

| Event | Payload | Description |
|-------|---------|-------------|
| `join_room` | `{room_code, user_id}` | Join auction room |
| `select_team` | `{franchise_name}` | Select IPL franchise |
| `place_bid` | `{player_id, amount}` | Place bid on player |
| `start_auction` | `{}` | Host starts auction |
| `next_player` | `{}` | Move to next player |

### Server → Client Events

| Event | Payload | Description |
|-------|---------|-------------|
| `room_update` | `{teams, status}` | Room state changed |
| `bid_update` | `{player, highest_bid, bidder}` | New bid placed |
| `player_sold` | `{player, team, amount}` | Player sold |
| `timer_update` | `{seconds_left}` | Countdown update |
| `auction_complete` | `{rankings}` | Auction finished |

### Event Handler Structure
```python
# Pseudocode for WebSocket Handler
@socketio.on('place_bid')
def handle_bid(data):
    room_code = get_user_room(request.sid)
    team = get_team(request.sid)
    
    # Validate bid
    is_valid, message = validate_bid(team, data)
    
    if is_valid:
        # Update room state
        update_highest_bid(room_code, data)
        
        # Broadcast to room
        emit('bid_update', {
            'player': current_player,
            'amount': data['amount'],
            'team': team.name
        }, room=room_code)
    else:
        # Send error to bidder only
        emit('bid_error', {'message': message})
```

---

## AI Integration

### Gemini API Implementation

**Team Analysis Workflow:**
```
1. Auction Completes
2. Collect all team data
3. Generate analysis prompt
4. Call Gemini API
5. Parse JSON response
6. Calculate rankings
7. Broadcast results
```

**Prompt Engineering:**
```python
# Pseudocode for AI Prompt
def generate_analysis_prompt(teams):
    prompt = """
    You are an IPL cricket expert. Analyze these teams:
    
    """
    
    for team in teams:
        prompt += f"""
        Team: {team.franchise}
        Budget: ₹{team.spent}L / ₹8000L
        
        Batsmen: {team.batsmen_list}
        Bowlers: {team.bowlers_list}
        All-rounders: {team.allrounders_list}
        
        """
    
    prompt += """
    Provide:
    1. Batting score (0-10)
    2. Bowling score (0-10)
    3. Balance score (0-10)
    4. Overall rating (0-10)
    5. Strengths and weaknesses
    
    Return as JSON array.
    """
    
    return prompt
```

**Response Parsing:**
```python
# Pseudocode for Response Handling
def parse_gemini_response(response):
    try:
        data = json.loads(response.text)
        rankings = []
        
        for team_analysis in data:
            rankings.append({
                'team': team_analysis['team_name'],
                'overall_score': team_analysis['overall_rating'],
                'batting': team_analysis['batting_score'],
                'bowling': team_analysis['bowling_score'],
                'analysis': team_analysis['summary']
            })
        
        # Sort by overall score
        rankings.sort(key=lambda x: x['overall_score'], reverse=True)
        return rankings
        
    except Exception as e:
        return fallback_ranking(teams)
```

---

## Security

### Authentication System

**Google OAuth Flow:**
```
1. User clicks "Sign in with Google"
2. Redirect to Google OAuth consent
3. Google returns authorization code
4. Exchange code for access token
5. Fetch user profile
6. Create/update user in database
7. Generate session token
8. Set secure HTTP-only cookie
```

**Password Security:**
```python
# Pseudocode for Password Handling
import bcrypt

def register_user(username, password):
    # Hash password with salt
    salt = bcrypt.gensalt(rounds=12)
    password_hash = bcrypt.hashpw(password.encode(), salt)
    
    user = {
        'username': username,
        'password_hash': password_hash,
        'created_at': now()
    }
    
    save_user(user)

def verify_password(username, password):
    user = get_user(username)
    return bcrypt.checkpw(
        password.encode(),
        user['password_hash']
    )
```

### Session Management
- JWT tokens with 24-hour expiry
- Secure HTTP-only cookies
- CSRF protection on forms
- Rate limiting on API endpoints

---

## Deployment

### Production Configuration

**Gunicorn Setup:**
```python
# config/gunicorn.py
import multiprocessing

bind = "0.0.0.0:5000"
workers = multiprocessing.cpu_count() * 2 + 1
worker_class = "geventwebsocket.gunicorn.workers.GeventWebSocketWorker"
timeout = 120
keepalive = 5
max_requests = 1000
max_requests_jitter = 50
```

**Environment Variables:**
```bash
# Production .env
FLASK_ENV=production
SECRET_KEY=<strong-random-key>
GEMINI_API_KEY=<your-api-key>
GOOGLE_CLIENT_ID=<oauth-client-id>
GOOGLE_CLIENT_SECRET=<oauth-secret>
MONGODB_URI=<mongodb-connection-string>
```

### Render Deployment

**Build Command:**
```bash
pip install -r requirements.txt
```

**Start Command:**
```bash
gunicorn --config config/gunicorn.py wsgi:app
```

**Health Check Endpoint:**
```python
@app.route('/health')
def health_check():
    return {'status': 'healthy', 'timestamp': time.time()}
```

---

## 📸 Screenshots

For detailed screenshots and visual walkthrough of the application, see **[PROJECT_IMAGES.md](./PROJECT_IMAGES.md)**

---

## Technical Challenges & Solutions

### Challenge 1: WebSocket Scalability
**Problem:** Socket connections dropping under load  
**Solution:** Implemented gevent workers with connection pooling and heartbeat mechanism

### Challenge 2: Race Conditions in Bidding
**Problem:** Multiple simultaneous bids causing conflicts  
**Solution:** Server-side locking mechanism with timestamp validation

### Challenge 3: Room State Persistence
**Problem:** Rooms lost on server restart  
**Solution:** Periodic JSON snapshots with auto-recovery on startup

### Challenge 4: AI Response Time
**Problem:** Gemini API latency affecting UX  
**Solution:** Async processing with loading states and fallback rankings

---

## Performance Metrics

- **Concurrent Users:** 30+ tested successfully
- **Bid Latency:** < 100ms average
- **Room Capacity:** 10 teams per room
- **Player Database:** 364 players with full stats
- **Uptime:** 99.5% on Render free tier

---

##  Code StEructure

### Main Application (app.py)
```python
# High-level structure
from flask import Flask, render_template
from flask_socketio import SocketIO, emit, join_room

app = Flask(__name__)
socketio = SocketIO(app, cors_allowed_origins="*")

# Routes
@app.route('/')
def index():
    return render_template('index.html')

# WebSocket handlers
@socketio.on('join_room')
def handle_join(data):
    # Room joining logic
    pass

@socketio.on('place_bid')
def handle_bid(data):
    # Bidding logic
    pass

if __name__ == '__main__':
    socketio.run(app, debug=False)
```

### Service Layer Architecture
```
services/
├── auth_service.py          # User authentication
├── room_persistence_service.py  # Room state management
├── rebid_service.py         # Unsold player handling
├── ai_ranking_service.py    # Gemini API integration
├── room_cleanup_service.py  # Automatic cleanup
└── mongodb_service.py       # Database operations
```

---

<div align="center">

## Developer

**Yash Sidram Gajakosh**  
GitHub: [@06yash12](https://github.com/06yash12)

---

**Built with Love for Cricket Fans**

*This project is for educational and entertainment purposes only.*

[Back to Top](#ipl-auction-playzone---complete-project-documentation)

</div>
