# Folio 2025 - Project Documentation

## Overview

Folio 2025 is an elaborate **3D interactive portfolio experience** built with Three.js WebGPU and JavaScript. It combines a creative portfolio presentation with an explorable game world featuring physics-based vehicle movement, dynamic weather systems, and multiple interactive content areas.

---

## Table of Contents

1. [Project Structure](#1-project-structure)
2. [Technologies & Dependencies](#2-technologies--dependencies)
3. [Architecture](#3-architecture)
4. [Core Systems](#4-core-systems)
5. [World & Areas](#5-world--areas)
6. [Input System](#6-input-system)
7. [Rendering & Materials](#7-rendering--materials)
8. [Physics Engine](#8-physics-engine)
9. [Environmental Systems](#9-environmental-systems)
10. [Audio System](#10-audio-system)
11. [Build & Development](#11-build--development)
12. [Debug Mode](#12-debug-mode)
13. [Special Features](#13-special-features)

---

## 1. Project Structure

### Root Directory Layout

```
/folio-2025
├── package.json           # Dependencies and scripts
├── vite.config.js         # Vite bundler configuration
├── readme.md              # Game loop documentation
├── sources/               # Main source code (Vite root)
├── static/                # Public assets (models, textures, sounds)
├── resources/             # Blender source files and design assets
├── scripts/               # Utility scripts (compression)
└── dist/                  # Build output (generated)
```

### Sources Directory Structure

```
sources/
├── index.js               # Application entry point
├── index.html             # HTML template
├── threejs-override.js    # Three.js customizations
├── Game/                  # Core game engine (117 JS files)
│   ├── Game.js            # Main singleton orchestrator
│   ├── Ticker.js          # Game loop and timing
│   ├── Rendering.js       # WebGPU renderer setup
│   ├── Physics/           # Rapier3D physics engine
│   ├── Inputs/            # Multi-input system
│   ├── World/             # Game world and areas
│   ├── Materials/         # Custom shaders and materials
│   ├── Geometries/        # Custom geometry definitions
│   ├── Cycles/            # Time systems (day/year)
│   ├── Passes/            # Post-processing effects
│   └── utilities/         # Helper functions
├── style/                 # Stylus CSS files (20 files)
└── data/                  # Static data (achievements, projects, etc.)
```

### Key Directories

| Directory | Purpose |
|-----------|---------|
| `Game/` | Core engine with 117 JavaScript files |
| `Game/World/` | World rendering and interactive areas |
| `Game/World/Areas/` | 13 distinct portfolio content areas |
| `Game/Physics/` | Rapier3D physics integration |
| `Game/Inputs/` | Multi-device input handling |
| `Game/Materials/` | Custom TSL shader materials |
| `Game/Cycles/` | Day/night and seasonal cycles |
| `style/` | 20 Stylus preprocessor files |
| `data/` | Static content data |

---

## 2. Technologies & Dependencies

### Core Graphics & 3D

| Technology | Purpose |
|------------|---------|
| **Three.js WebGPU** | Custom build for cutting-edge GPU compute |
| **TSL (Three.js Shading Language)** | Node-based shader programming |
| **WebGPU** | Modern GPU API for high-performance rendering |
| **GLTFLoader, DRACOLoader, KTX2Loader** | 3D model and texture loading |

### Physics

| Technology | Purpose |
|------------|---------|
| **Rapier3D** | 3D rigid body physics engine (WebAssembly) |

### Input & Interaction

| Technology | Purpose |
|------------|---------|
| **Keyboard API** | Traditional keyboard input |
| **Gamepad API** | Game controller support |
| **Pointer Events** | Mouse/touch input |
| **Nipple.js** | Touch joystick for mobile |

### Build Tools

| Technology | Version | Purpose |
|------------|---------|---------|
| **Vite** | 7.3.1 | Modern bundler with HMR |
| **vite-plugin-wasm** | - | WebAssembly module loading |
| **vite-plugin-top-level-await** | - | Top-level await support |
| **Bun** | - | Package manager |

### UI & Animation

| Technology | Version | Purpose |
|------------|---------|---------|
| **GSAP** | 3.14.2 | Professional animation |
| **Tweakpane** | 4.0.5 | Debug UI panels |
| **Howler.js** | 2.2.4 | Audio playback |

### Styling

| Technology | Purpose |
|------------|---------|
| **Stylus** | CSS preprocessor |

### Utilities

| Package | Purpose |
|---------|---------|
| **camera-controls** | Advanced camera controller |
| **seedrandom** | Deterministic random generation |
| **uuid** | Unique ID generation |
| **msgpack-lite** | Binary serialization |
| **sharp** | Image compression |
| **@gltf-transform** | glTF model optimization |

---

## 3. Architecture

### Design Patterns

#### Singleton Pattern
- **Game.js** serves as the central singleton hub
- All systems communicate through `Game.getInstance()`
- Single instance ensures coordinated system access

#### Observer/Event Pattern
- **Events class** provides pub/sub with priority ordering
- `on(name, callback, order)` enables ordered execution
- Used for ticker updates, input events, and inter-system communication

#### Component/System Architecture
- Each major system is a separate class
- Loose coupling through events and the Game instance
- Systems register tick handlers for specific update phases

#### Game Loop (Ticker-based)

The game uses an **ordered event-driven update cycle**:

```
Tick Order:
0:   Time + Inputs
1:   Player:pre-physics (reads inputs)
2:   PhysicalVehicle:pre-physics
3:   Physics simulation
4:   PhysicsWireframe + Objects
5:   PhysicalVehicle:post-physics
6:   Player:post-physics
7:   View (camera)
8:   Intro, DayCycles, YearCycles, Weather, Zones, VisualVehicle
9:   Wind, Lighting, Tornado, InteractivePoints, Tracks
10:  Areas, Foliage, Fog, Terrain, Trails, Grass, Leaves
13:  InstancedGroup rendering
14:  Audio + Notifications + Title
998: Rendering
999: Monitoring
```

### System Relationships

```
┌─────────────────────────────────────────────────────────┐
│                        Game.js                          │
│                    (Singleton Hub)                       │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐ │
│  │ Ticker   │  │ Inputs   │  │ Physics  │  │ Audio   │ │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬────┘ │
│       │             │             │              │       │
│       └─────────────┴──────┬──────┴──────────────┘       │
│                            │                             │
│  ┌──────────┐  ┌──────────┴─────────┐  ┌────────────┐  │
│  │Rendering │  │      Player        │  │   World    │  │
│  │          │  │  (PhysicsVehicle)  │  │  (Areas)   │  │
│  └──────────┘  └────────────────────┘  └────────────┘  │
│                                                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │  View    │  │  Cycles  │  │ Weather  │              │
│  │ (Camera) │  │(Day/Year)│  │          │              │
│  └──────────┘  └──────────┘  └──────────┘              │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 4. Core Systems

### Game.js (Main Orchestrator)

The central singleton that initializes and coordinates all systems:

```javascript
// Access pattern
const game = Game.getInstance()
```

**Responsibilities:**
- Initializes all subsystems in correct order
- Provides access to shared resources
- Manages application lifecycle

### Ticker.js (Game Loop)

Manages frame timing and ordered updates:

- **Elapsed time tracking**
- **Delta time calculation**
- **Frame rate limiting**
- **Ordered event dispatching**

### Time.js (Time Control)

Advanced time management:

- **Time scaling** - Speed up/slow down game time
- **Bullet-time effect** - Dramatic slow motion
- **Pause functionality**

### Events.js (Event System)

Custom event dispatcher with priority ordering:

```javascript
// Register ordered callback
events.on('tick', callback, order)

// Order determines execution sequence
// Lower numbers execute first
```

### Viewport.js

Screen and window management:

- Resize handling
- Resolution scaling
- Aspect ratio calculations

---

## 5. World & Areas

### World.js

The main world container managing:

- Scene graph organization
- Area loading and management
- Environmental systems coordination

### Areas System

**13 distinct interactive portfolio areas:**

| Area | Purpose |
|------|---------|
| **LandingArea** | Initial spawn and introduction |
| **ProjectsArea** | Portfolio project showcase |
| **LabArea** | Experimental/lab content |
| **CircuitArea** | Racing circuit with obstacles |
| **CareerArea** | Career/work history |
| **SocialArea** | Social links and contact |
| **AchievementsArea** | Player achievements display |
| **BowlingArea** | Bowling mini-game |
| **AltarArea** | Special interactive altar |
| **CookieArea** | Cookie-themed area |
| **BehindTheSceneArea** | Behind-the-scenes content |
| **ToiletArea** | Easter egg area |
| **TimeMachine** | Time-related interactions |

### Visual Components

| Component | Description |
|-----------|-------------|
| **Floor.js** | Ground plane rendering |
| **Grass.js** | Animated grass system |
| **Trees.js** | Birch, Oak, and Cherry trees |
| **Foliage.js** | General vegetation |
| **WaterSurface.js** | Water rendering |
| **Leaves.js** | Wind-affected leaf particles |
| **Confetti.js** | Celebration particles |
| **Lanterns.js, PoleLights.js** | Light sources |
| **Fences.js, Bricks.js** | Structural elements |
| **ExplosiveCrates.js** | Interactive explosives |
| **Snow.js, RainLines.js** | Weather particles |

---

## 6. Input System

### Multi-Device Support

The input system abstracts multiple input methods into a unified interface:

```
┌─────────────────────────────────────────┐
│              Inputs.js                   │
│          (Input Aggregator)              │
├─────────────────────────────────────────┤
│                                          │
│  ┌──────────┐  ┌──────────┐            │
│  │ Keyboard │  │ Gamepad  │            │
│  └──────────┘  └──────────┘            │
│                                          │
│  ┌──────────┐  ┌──────────┐            │
│  │ Pointer  │  │  Wheel   │            │
│  └──────────┘  └──────────┘            │
│                                          │
│  ┌──────────┐  ┌──────────────────────┐ │
│  │ Nipple   │  │ InteractiveButtons  │ │
│  │ (Touch)  │  │                      │ │
│  └──────────┘  └──────────────────────┘ │
│                                          │
└─────────────────────────────────────────┘
```

### Input Methods

| Method | File | Purpose |
|--------|------|---------|
| **Keyboard** | Keyboard.js | Key press/release handling |
| **Gamepad** | Gamepad.js | Controller support |
| **Pointer** | Pointer.js | Mouse/touch coordinates |
| **Wheel** | Wheel.js | Scroll wheel input |
| **Nipple** | Nipple.js | Touch joystick (mobile) |
| **Interactive Buttons** | InteractiveButtons.js | On-screen buttons |

### Features

- **Mode detection** - Automatically detects keyboard/mouse, gamepad, or touch
- **Context filtering** - Areas can enable/disable specific inputs
- **Observable state** - CSS classes reflect current input mode

---

## 7. Rendering & Materials

### Rendering.js

WebGPU renderer configuration:

- **WebGPURenderer** setup
- **Post-processing pipeline**
- **Resolution management**
- **Quality settings**

### Post-Processing

| Pass | File | Effect |
|------|------|--------|
| **Depth of Field** | cheapDOF.js | Focus blur effect |

### Custom Materials

#### MeshDefaultMaterial.js

Extended Lambert material using TSL (Three.js Shading Language):

**Features:**
- Core shadows
- Drop shadows
- Light bounce
- Fog integration
- Water effects
- Reveal animation

```javascript
// TSL node-based shader composition
// Modular shader nodes for flexibility
```

#### MeshGridMaterial.js

Grid visualization material for debug and special effects.

### Custom Geometries

| Geometry | Purpose |
|----------|---------|
| **LineGeometry** | Custom line rendering |
| **PortalSlabsGeometry** | Portal visual effects |
| **WindLineGeometry** | Wind visualization |

---

## 8. Physics Engine

### Rapier3D Integration

The physics system uses **Rapier3D** (WebAssembly) for rigid body simulation.

### Physics.js

Core physics world management:

- **World creation and stepping**
- **Collision detection**
- **Rigid body management**

### PhysicsVehicle.js

Vehicle-specific physics controller:

**Configuration:**
- Suspension settings (low/mid/high heights)
- Wheel tuning (radius, friction)
- Steering limits
- Acceleration curves

**Collision Groups:**
- `all` - General collision
- `object` - Static objects
- `bumper` - Vehicle bumpers

**Friction Rules:**
- Average
- Min
- Max
- Multiply

### PhysicsWireframe.js

Debug visualization for physics bodies.

---

## 9. Environmental Systems

### Cycles (Time Systems)

#### DayCycles.js

Day/night cycle management:

- **Sun position** calculation
- **Ambient light** adjustment
- **Sky color** transitions
- **Shadow intensity** changes

#### YearCycles.js

Seasonal progression:

- **Flora changes** (leaf colors, snow coverage)
- **Weather patterns**
- **Ambient sounds**

### Weather.js

Dynamic weather system:

| Weather | Components |
|---------|------------|
| **Rain** | RainLines.js |
| **Snow** | Snow.js |
| **Lightning** | Lightnings.js |
| **Wind** | Wind.js, WindLines.js |

### Lighting.js

Dynamic lighting management:

- **Directional light** (sun)
- **Ambient light**
- **Point lights** (lanterns, poles)
- **Light bounce** simulation

### Fog.js

Atmospheric fog effects:

- Distance-based fog
- Color matching with sky
- Density adjustments

### Wind.js

Wind simulation affecting:

- Grass movement
- Leaf particles
- Tree swaying
- Foliage animation

### Tornado.js & VisualTornado.js

Special tornado weather event:

- Physics-based object interaction
- Visual spiral effect
- Sound effects

---

## 10. Audio System

### Audio.js (Howler.js Integration)

Centralized audio management:

**Sound Groups:**
- **Playlists** - Background music
- **Ambients** - Environmental sounds
- **One-offs** - Sound effects

**Features:**
- Master volume control
- Group-level volume
- Positional audio
- Playlist management

### Sound Categories

| Category | Examples |
|----------|----------|
| **Music** | Background tracks |
| **Ambient** | Wind, birds, rain |
| **Effects** | Engine, collisions, UI |
| **Voice** | Notifications |

---

## 11. Build & Development

### Prerequisites

- **Bun** (recommended) or **npm** package manager
- **Node.js** 18+

### Development Commands

```bash
# Install dependencies
npm install --force

# Start development server with HMR
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Compress assets (models, textures)
npm run compress
```

### Environment Variables

Create a `.env` file in the root directory:

```env
# Backend API endpoint
VITE_SERVER_URL=

# Google Analytics tag
VITE_ANALYTICS_TAG=

# Expose game to window.game for debugging
VITE_GAME_PUBLIC=

# Override day cycle progress (0-1)
VITE_DAY_CYCLE_PROGRESS=

# Override year cycle progress (0-1)
VITE_YEAR_CYCLE_PROGRESS=

# Number of whisper messages
VITE_WHISPERS_COUNT=30

# Enable/disable music
VITE_MUSIC=1

# Enable console logging
VITE_LOG=1

# Override player spawn location
VITE_PLAYER_SPAWN=
```

### Asset Compression

The compression script optimizes:

- **GLB models** - ETC1S compression
- **Textures** - GPU-friendly formats (KTX2)
- **UI images** - WebP conversion

---

## 12. Debug Mode

### Activation

Add `#debug` to the URL hash:

```
http://localhost:5173/#debug
```

### Tweakpane Panels

| Panel | Controls |
|-------|----------|
| **Rendering** | Resolution, quality settings |
| **View** | Camera mode, focus distance |
| **Physics** | Vehicle tuning parameters |
| **Time** | Time scale, bullet-time |
| **Zones** | Zone visualization |
| **Weather** | Weather controls |
| **Lighting** | Light settings |

### Inspector Mode

Add `#inspector` for Three.js scene inspection:

```
http://localhost:5173/#inspector
```

### Debug.js Features

- **Live variable tweaking**
- **Performance graphs**
- **Physics wireframe visualization**
- **Camera controls**
- **Zone debugging**

---

## 13. Special Features

### Easter Eggs

#### KonamiCode.js

Classic Konami code activation for hidden features.

#### Easter.js

Seasonal Easter event with special content.

### Holiday Events

#### BlackFriday.js

Black Friday themed content and visuals:

- Special visual effects
- Themed areas
- Fragment objects

### Player Progression

#### Achievements.js

Achievement tracking system:

- **Distance traveled**
- **Time played**
- **Flip counter**
- **Area discoveries**
- **Special actions**

### Interactive Elements

| Feature | Description |
|---------|-------------|
| **InteractivePoints** | Clickable hotspots |
| **Zones** | Spherical interaction areas |
| **Explosions** | Destructible objects |
| **Respawns** | Player spawn points |

### Performance Optimization

| Technique | Implementation |
|-----------|----------------|
| **Instanced rendering** | InstancedGroup.js |
| **LOD** | Quality settings |
| **Texture compression** | KTX2, ETC1S |
| **Model compression** | Draco |
| **Frustum culling** | Built-in Three.js |
| **Zone-based loading** | Areas system |

### Camera Modes

| Mode | Description |
|------|-------------|
| **Default** | Following vehicle |
| **Free** | Debug free camera |
| **Cinematic** | Smooth transitions |

---

## File Statistics

| Category | Count |
|----------|-------|
| JavaScript files (Game) | 117 |
| JavaScript files (total) | 125 |
| Stylus files | 20 |
| Interactive areas | 13 |
| Static asset categories | 42 |
| Custom materials | 2 |
| Custom geometries | 3 |
| Input methods | 6 |
| Time systems | 3 |

---

## Development Patterns

### Code Organization

- **Class-based** - Each system is a class instance
- **Composition** - Game aggregates all systems
- **Event-driven** - Ticker events coordinate updates
- **Singleton** - Game instance is the central hub

### Naming Conventions

- Debug panels prefixed with emoji
- State constants as static properties
- Private methods with underscore prefix
- Hierarchical naming (PhysicalVehicle vs VisualVehicle)

### Configuration Approach

- Hardcoded defaults in constructors
- Debug panel bindings for live tweaking
- Environment variables for external config
- Observable properties for reactive updates

---

## Recent Development Focus

Based on recent commits:

- CircuitArea bug fixes (collision handling, respawn logic)
- Confetti system enhancements
- Achievements refinements
- Performance optimization (preload assets)
- Visual polish (foliage animation)
- Vehicle enhancements (back lights)
- Dependency updates
