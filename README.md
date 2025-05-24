# Datastar Chess Game

A real-time chess game built with **Datastar**, demonstrating advanced reactive web development patterns including real-time multiplayer functionality, server-sent events, and dynamic UI updates.

## Overview

This chess application showcases Datastar's capabilities for building interactive, real-time web applications. Players can start a new game and play against an AI opponent with live board updates, move validation, and game state synchronization.

## Project Structure

```
datastar-chess/
├── app.py                # Main Flask/Quart application
├── templates/
│   └── index.html        # Landing page template
├── static/
│   ├── css/
│   │   ├── index.css     # Main stylesheet
│   │   └── gold.css
│   └── img/
│       ├── favicon.ico
│       └── chess/
│           └── pieces/   # Chess piece SVG assets
├── datastar_py/          # Local Datastar Python integration
└── README.md
```

## Technology Stack

- **Backend**: Python with Quart (async Flask)
- **Frontend**: Datastar for reactive UI
- **Chess Engine**: `python-chess` library
- **State Management**: Redis for persistence and pub/sub
- **Real-time Communication**: Server-Sent Events (SSE)

## Datastar Integration & Features

### 1. Client-Side Setup

The frontend loads Datastar from CDN and establishes the reactive foundation:

```html
<script
  type="module"
  src="https://cdn.jsdelivr.net/gh/starfederation/datastar@1.0.0-beta.9/bundles/datastar.js"
></script>
```

### 2. Event Handling & Navigation

**Landing Page Interaction**

```html
<button class="card gp gc gt l" data-on-click="@get('/new_game')">Chess</button>
```

- `data-on-click="@get('/new_game')"`: Datastar directive that makes an HTTP GET request when clicked
- Seamless navigation without page refreshes

### 3. Real-Time Game State Management

**Server-Sent Events Stream**

```python
@stream_with_context
async def event(generator):
    try:
        while True:
            message = await pubsub.get_message()
            if message:
                # Process game state changes
                html = await view(board_name, fen, session['user_id'], "AI", opponent_turn, game_over_message)
                yield generator.merge_fragments(fragments=[html])
```

Key Datastar concepts demonstrated:

- **Fragment Merging**: `generator.merge_fragments()` updates specific parts of the UI
- **Server-Sent Events**: Continuous connection for real-time updates
- **State Synchronization**: Game state changes trigger immediate UI updates

### 4. Interactive Chess Board

**Dynamic Board Generation**

```python
async def h_chessboard(fen):
    board = chess.Board(fen)

    for square in chess.SQUARES[::-1]:
        square_attr = {
            'clicked': f'$clicked=={square}',
            'possible': f'{possible.get(square, [])}.includes($clicked)',
            'size': '$clicked'
        }
        square_html = f'<div class="square {square_color} gc gs" data-attr="{square_attr}" data-on-click="$clicked={square}">'
```

**Reactive Square States**:

- `data-signals-clicked__ifmissing=-1`: Initialize clicked signal to -1 if missing
- `data-on-click__outside="$clicked=-1"`: Reset clicked state when clicking outside
- `data-attr="{clicked: '$clicked==square', possible: '...', size: '$clicked'}"`: Dynamic CSS classes based on state
- `data-on-click="$clicked=square"`: Update clicked signal when square is selected

### 5. Move Processing & Validation

**Client-Side Move Submission** (generated in `view()` function):

```html
<main id="main" data-on-signals-change__viewTransition="@post('/chess')"></main>
```

- `data-on-signals-change__viewTransition`: Automatically POST to `/chess` when signals change
- Enables smooth animations during state transitions

**Server-Side Move Processing**

```python
@app.post("/chess")
async def chess_route():
    data = await request.json
    clicked = data.get('clicked')

    # Process move logic
    if await try_move(board_name, last_two_clicks):
        # Trigger AI response
        await redis_client.publish(board_name, "Someone played")
        asyncio.create_task(ai_move(board_name))
```

### 6. State Signals & Reactivity

The chess board uses several reactive signals:

- **`$clicked`**: Tracks which square is currently selected
- **Square States**: Each square reactively updates its appearance based on:
  - Whether it's the currently clicked square
  - Whether it's a valid move destination
  - Current game state

### 7. Game Flow Architecture

1. **Game Initialization**:

   - Player clicks "Chess" → GET `/new_game`
   - Server creates Redis game state and establishes SSE connection

2. **Move Interaction**:

   - Player clicks square → Updates `$clicked` signal
   - Signal change triggers POST to `/chess` with move data

3. **Move Processing**:

   - Server validates move using chess engine
   - Updates Redis game state
   - Publishes game state change

4. **Real-Time Updates**:
   - All connected clients receive state changes via SSE
   - UI automatically updates with new board position
   - AI makes counter-move if game continues

## Key Datastar Patterns Demonstrated

### Reactive Data Binding

- Signals (`$clicked`) automatically trigger UI updates
- Attribute binding with `data-attr` for dynamic styling

### Server Communication

- `@get()` and `@post()` for HTTP requests
- `@sse` patterns for real-time communication

### Fragment-Based Updates

- Selective DOM updates without full page reloads
- Merge fragments for efficient rendering

### Event Handling

- `data-on-click` for user interactions
- `data-on-signals-change` for automatic form submission

## Running the Application

### Prerequisites

- Python 3.12+
- Redis server
- uv

### Setup with uv (Recommended)

1. **Start Redis server**:

   ```bash
   redis-server
   ```

2. **Run the application**:

   ```bash
   uv run ./app.py
   ```

3. **Access**: Open `http://localhost:5000` in your browser

### Note on Datastar Integration

This project uses a local copy of the `datastar_py` module (included in the `datastar_py/` directory) rather than the PyPI package. This ensures compatibility with the specific Datastar version and features used in this chess implementation.

## Notable Implementation Details

- **Performance Optimization**: Execution time logging for performance monitoring
- **Rate Limiting**: 10 requests per second protection
- **Session Management**: Automatic user ID generation with Faker
- **Game State Persistence**: Redis-backed game history and move tracking
- **AI Integration**: Asynchronous AI move generation with 2-second thinking delay
- **Error Handling**: Graceful connection management and cleanup

This chess game demonstrates Datastar's power for building complex, real-time applications with minimal client-side JavaScript while maintaining excellent user experience and performance.
