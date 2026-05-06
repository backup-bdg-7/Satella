# 1. OBJECTIVE

Build **Doomnite** - a full production-grade iOS game with a custom Metal-based rendering engine, featuring:
- Open-world RPG exploration with quest systems (Elder Scrolls-inspired)
- Fast-paced FPS combat with diverse weapons and enemies (Doom-inspired)
- Social hubs, vendors, equipment customization, and character progression (Animal Jam-inspired)
- Battle royale mode with building mechanics and multiplayer support (Fortnite-inspired)

The goal is to deliver a complete, optimized iOS game with native performance using Swift, Metal, and a custom Entity Component System (ECS) architecture. The project uses XcodeGen for project generation via `project.yml` and will be manually built into an IPA.

# 2. CONTEXT SUMMARY

## Repository State
- Empty repository with only a placeholder file (`J`)
- No existing code, dependencies, or project structure
- Will use XcodeGen to generate the Xcode project from `project.yml`

## Technical Stack
- **Language**: Swift 5.9+ (leveraging latest Swift concurrency and optimizations)
- **Rendering**: Apple Metal 3 with low-level GPU access
- **Architecture**: Custom ECS (Entity Component System) for game logic
- **Build System**: XcodeGen (project.yml configuration)
- **Dependencies** (via Swift Package Manager):
  - `simd` (built-in) - Vector/matrix math for 3D
  - `MetalKit` (built-in) - Metal utilities
  - `ModelIO` (built-in) - 3D model loading
  - `GameController` (built-in) - Controller support
  - `AVFoundation` (built-in) - Audio system
  - `Network` (built-in) - Low-level networking

## Target Platform
- **iOS 17.0+** (minimum for latest Metal features)
- **Devices**: iPhone 12 and newer (A14 Bionic+ for optimal performance)
- **Orientation**: Landscape (primary), Portrait (UI only)

## Performance Constraints
- Target 60 FPS locked (with 120Hz ProMotion support)
- Thermal state monitoring to prevent overheating
- Dynamic resolution scaling based on device thermal state
- Memory budgeting: <2GB RAM usage on target devices

# 3. APPROACH OVERVIEW

## Engine Architecture: Custom Metal ECS Engine

The engine will be built from scratch with these core systems:

### 1. Core Engine Systems
- **ECS Framework**: Type-safe Swift ECS with archetype-based storage for cache-friendly iteration
- **Metal Renderer**: PBR (Physically Based Rendering) pipeline with:
  - Deferred rendering path for complex scenes
  - Forward rendering for transparent objects
  - Shadow mapping (cascaded for directional, omnidirectional for point lights)
  - Post-processing stack (bloom, tone mapping, color grading, vignette)
  - Particle system with GPU compute
- **Asset Pipeline**: Asynchronous asset loading with reference counting
- **Scene Graph**: Spatial partitioning (octree) for culling and queries

### 2. Game Systems
- **Combat System**: Hitscan and projectile weapons, melee combat, damage types, armor penetration
- **AI System**: Behavior trees, navigation mesh, squad tactics, enemy spawning
- **Quest System**: Objective tracking, quest chains, rewards, NPC dialog
- **Inventory System**: Equipment slots, item stats, rarity tiers, crafting
- **Vendor System**: Dynamic pricing, faction reputation, shop inventories
- **Progression System**: XP, leveling, skill trees, stat allocation
- **World System**: Zone management, weather, day/night cycle, POI (points of interest)

### 3. Networking (Battle Royale Mode)
- Client-server architecture with authoritative server
- State synchronization with client-side prediction
- Lag compensation (rewind-based hit detection)
- Matchmaking and lobby system

### 4. Performance Strategy
- **Thermal Management**: Register for `NSProcessInfo.thermalState` notifications, dynamically scale:
  - Render resolution (100% → 75% → 50%)
  - Shadow quality
  - Particle counts
  - AI update frequencies
- **Memory Management**: Custom memory pools for frequent allocations, texture streaming
- **GPU Optimization**: Batch draw calls, texture atlasing, LOD system

## Why This Approach
- **Custom ECS**: Maximum flexibility and performance for a game with diverse systems
- **Metal (not Unity/Unreal)**: Native performance, full control over rendering, no licensing fees
- **Swift**: Memory safety, modern concurrency, great Metal integration
- **XcodeGen**: Reproducible builds, no `.xcodeproj` merge conflicts

# 4. IMPLEMENTATION STEPS

## Phase 1: Project Foundation & Build System

### Step 1.1: Create project.yml for XcodeGen
**Goal**: Define the complete Xcode project structure
**Method**: Create `project.yml` with:
- All targets (Doomnite game, DoomniteEngine framework, DoomniteShared)
- Build settings optimized for release (fast math, optimization)
- Code signing configuration
- Swift package dependencies
**Reference**: `project.yml` (root)

### Step 1.2: Directory Structure Setup
**Goal**: Create the source code directory hierarchy
**Method**: Create directories:
```
Sources/
  Doomnite/           # Main game target
    App/              # App delegate, entry point
    Game/             # Game-specific systems
    UI/               # SwiftUI/UIKit overlays
  DoomniteEngine/     # Engine framework
    Core/             # ECS, Math, Memory
    Rendering/        # Metal renderer
    Audio/            # Audio system
    Physics/          # Physics simulation
    Networking/       # Network layer
    Assets/           # Asset loading
  DoomniteShared/     # Shared types and utilities
Resources/
  Assets.xcassets/    # Textures, colors, icons
  Models/             # 3D models (USDZ/OBJ)
  Shaders/            # Metal shader files (.metal)
  Audio/              # Sound effects and music
  Data/               # JSON game data files
```
**Reference**: Entire `Sources/` and `Resources/` directories

### Step 1.3: Generate Xcode Project
**Goal**: Verify project.yml generates correctly
**Method**: Run `xcodegen generate` and verify the project structure
**Reference**: Generated `.xcodeproj`

## Phase 2: Core Engine - Math & ECS

### Step 2.1: SIMD Math Library
**Goal**: High-performance vector/matrix math using SIMD
**Method**: Create extensions on `SIMD3`, `SIMD4`, `float4x4` for:
- Transform operations (translate, rotate, scale)
- Quaternion math for rotations
- Ray casting, frustum culling math
- Interpolation functions (lerp, slerp)
**Reference**: `Sources/DoomniteEngine/Core/Math/`

### Step 2.2: ECS Framework Implementation
**Goal**: Type-safe, performant ECS with archetype storage
**Method**: Implement:
- `Entity` (u64 ID with generation counter)
- `Component` protocol and component storage
- `Archetype` - groups entities with same component set
- `System` protocol with update methods
- `World` - container for all entities/systems
- Query API for efficient component iteration
- Command buffer for deferred entity mutations
**Reference**: `Sources/DoomniteEngine/Core/ECS/`

### Step 2.3: Memory Management System
**Goal**: Custom allocators to reduce ARC overhead in hot paths
**Method**: Implement:
- `ArenaAllocator` for frame-temporary allocations
- `PoolAllocator` for frequently allocated types
- `RingBuffer` for lock-free producer-consumer
- Memory tracking and leak detection in debug
**Reference**: `Sources/DoomniteEngine/Core/Memory/`

## Phase 3: Metal Renderer

### Step 3.1: Metal Device & Command Setup
**Goal**: Initialize Metal and create command queues
**Method**: Implement:
- `MetalContext` - device, queue, command buffer management
- `RenderTarget` - encapsulating textures for render passes
- `PipelineState` - cached render/compute pipeline states
- `BufferPool` - recycled GPU buffer allocations
**Reference**: `Sources/DoomniteEngine/Rendering/Core/`

### Step 3.2: PBR Shader Implementation
**Goal**: Physically based rendering shaders
**Method**: Write `.metal` shaders:
- Vertex shader with skeletal animation support
- PBR fragment shader (Albedo, Metallic, Roughness, Normal, AO)
- IBL (Image-Based Lighting) with environment maps
- BRDF lookup table generation
**Reference**: `Resources/Shaders/PBR.metal`

### Step 3.3: Deferred Rendering Pipeline
**Goal**: G-Buffer based deferred renderer for complex lighting
**Method**: Implement:
- G-Buffer pass (albedo, normal, metallic/roughness, depth)
- Light culling compute shader (tile-based)
- Lighting pass (directional, point, spot lights)
- Skybox/atmosphere rendering
- Tone mapping and gamma correction
**Reference**: `Sources/DoomniteEngine/Rendering/DeferredRenderer/`

### Step 3.4: Shadow System
**Goal**: Dynamic shadow mapping
**Method**: Implement:
- Cascaded shadow maps for directional light
- Omnidirectional shadows for point lights (cubemap)
- PCF (Percentage Closer Filtering) for soft shadows
- Shadow bias calculations to prevent acne
**Reference**: `Sources/DoomniteEngine/Rendering/Shadows/`

### Step 3.5: Post-Processing Stack
**Goal**: Visual effects for polish
**Method**: Implement:
- Bloom (bright pass + Gaussian blur chain)
- Tone mapping (ACES filmic)
- Color grading (3D LUT support)
- Vignette and chromatic aberration
- Motion blur (velocity buffer based)
**Reference**: `Sources/DoomniteEngine/Rendering/PostProcessing/`

### Step 3.6: Particle System
**Goal**: GPU-accelerated particle effects
**Method**: Implement:
- Particle spawner with emitters
- Compute shader for particle simulation
- Billboard and mesh particles
- Integration with renderer
**Reference**: `Sources/DoomniteEngine/Rendering/Particles/`

### Step 3.7: Model Loading & Mesh System
**Goal**: Load 3D models and manage mesh data
**Method**: Implement:
- `Mesh` class with vertex/index buffers
- ModelIO integration for USDZ/OBJ loading
- Material system linking shaders + textures
- Skeletal mesh with bone hierarchies
- Animation system (keyframe interpolation)
**Reference**: `Sources/DoomniteEngine/Rendering/Mesh/`

### Step 3.8: Texture System
**Goal**: Texture loading, compression, and streaming
**Method**: Implement:
- `Texture` wrapper with mipmap generation
- ASTC texture compression support
- Texture atlas packing
- Streaming manager for large worlds
**Reference**: `Sources/DoomniteEngine/Rendering/Textures/`

## Phase 4: Asset Pipeline

### Step 4.1: Asset Loading System
**Goal**: Asynchronous asset loading with reference counting
**Method**: Implement:
- `AssetManager` with async loading
- `AssetHandle` for reference-counted access
- Priority-based loading queue
- Loading progress tracking
**Reference**: `Sources/DoomniteEngine/Assets/`

### Step 4.2: Game Data Definitions
**Goal**: JSON-based game data for items, enemies, quests
**Method**: Create:
- `ItemDefinition` (weapons, armor, consumables)
- `EnemyDefinition` (stats, loot tables, AI parameters)
- `QuestDefinition` (objectives, rewards, dialog)
- `WorldDefinition` (zones, POIs, spawn areas)
- Codable structs with validation
**Reference**: `Sources/DoomniteShared/DataDefinitions/`

### Step 4.3: Default Assets
**Goal**: Procedural fallback assets when files are missing
**Method**: Generate:
- Default textures (checkerboard, normal map)
- Primitive meshes (cube, sphere, plane)
- Default shaders for error states
**Reference**: `Sources/DoomniteEngine/Assets/Defaults/`

## Phase 5: Game Systems

### Step 5.1: Input System
**Goal**: Unified input handling for touch, gamepad, keyboard
**Method**: Implement:
- `InputManager` with action mapping
- Touch gesture recognition (swipe, pinch, tap)
- GameController integration
- Input buffering for responsive controls
**Reference**: `Sources/DoomniteEngine/Core/Input/`

### Step 5.2: Camera System
**Goal**: Multiple camera modes (FPS, third-person, free)
**Method**: Implement:
- `Camera` component with projection/view matrices
- FPS camera with mouse/touch look
- Third-person with orbit controls
- Camera collision and smoothing
- Shake effects for impact feedback
**Reference**: `Sources/DoomniteEngine/Core/Camera/`

### Step 5.3: Player Controller
**Goal**: Player movement and action handling
**Method**: Implement:
- Character controller (WASD movement, jumping, sprint)
- FPS view model rendering
- Weapon switching and firing
- Interaction system (pickup, talk, use)
- Player stats (health, armor, stamina)
**Reference**: `Sources/Doomnite/Game/Player/`

### Step 5.4: Combat System
**Goal**: Complete weapon and damage system
**Method**: Implement:
- `WeaponDefinition` (damage, fire rate, accuracy, recoil)
- Hitscan weapons (instant raycast)
- Projectile weapons (physics-based)
- Melee weapons (swing detection)
- Damage calculation (armor reduction, critical hits)
- Hit feedback (screen shake, sound, particles)
- Death and respawn system
**Reference**: `Sources/Doomnite/Game/Combat/`

### Step 5.5: Enemy AI System
**Goal**: Intelligent enemy behavior
**Method**: Implement:
- Behavior tree system
- Navigation mesh pathfinding
- Enemy states (idle, patrol, alert, attack, flee)
- Squad coordination (flanking, covering fire)
- Loot drops on death
- Enemy spawner system with waves
**Reference**: `Sources/Doomnite/Game/AI/`

### Step 5.6: Inventory & Equipment System
**Goal**: Full inventory management
**Method**: Implement:
- `Inventory` component (grid or list-based)
- Item stacking and sorting
- Equipment slots (head, chest, legs, weapon, etc.)
- Item stats and modifiers
- Rarity system (common → legendary)
- Item comparison UI data
**Reference**: `Sources/Doomnite/Game/Inventory/`

### Step 5.7: Vendor System
**Goal**: NPCs that buy and sell items
**Method**: Implement:
- `Vendor` component with shop inventory
- Dynamic pricing (supply/demand)
- Faction reputation discounts
- Buy/sell transaction logic
- Restock timers
- Vendor UI data provider
**Reference**: `Sources/Doomnite/Game/Vendors/`

### Step 5.8: Quest System
**Goal**: Objective tracking and quest progression
**Method**: Implement:
- `Quest` with multiple objectives
- Objective types (kill, collect, reach, interact)
- Quest state machine (available, active, complete, turned in)
- Quest giver NPCs with dialog trees
- Reward distribution (XP, items, currency)
- Quest log UI data
**Reference**: `Sources/Doomnite/Game/Quests/`

### Step 5.9: World/Zone System
**Goal**: Open world with zones and points of interest
**Method**: Implement:
- `Zone` definition with boundaries
- Weather system (rain, snow, fog)
- Day/night cycle with sun/moon positioning
- POI (shops, dungeons, quest locations)
- Zone transitions and loading
- Ambient audio zones
**Reference**: `Sources/Doomnite/Game/World/`

### Step 5.10: Progression System
**Goal**: Player advancement and skill trees
**Method**: Implement:
- XP and leveling system
- Skill tree with nodes and dependencies
- Stat allocation (strength, agility, intelligence)
- Unlockable abilities
- Save/load progression data
**Reference**: `Sources/Doomnite/Game/Progression/`

## Phase 6: UI System

### Step 6.1: UI Framework
**Goal**: Game UI using SwiftUI + Metal overlay
**Method**: Implement:
- SwiftUI views for menus and HUD
- `MetalView` integration with UIKit/SwiftUI
- UI scaling for different device sizes
- Localization support
**Reference**: `Sources/Doomnite/UI/`

### Step 6.2: HUD (Heads-Up Display)
**Goal**: In-game HUD showing player status
**Method**: Create:
- Health/armor bars
- Ammo counter
- Minimap with player position
- Quest objective tracker
- Crosshair with hit feedback
- Notification toasts
**Reference**: `Sources/Doomnite/UI/HUD/`

### Step 6.3: Menu System
**Goal**: Full menu navigation
**Method**: Create:
- Main menu (play, settings, about)
- Pause menu (resume, settings, quit)
- Settings (graphics, audio, controls)
- Inventory screen
- Character sheet
- Map screen
**Reference**: `Sources/Doomnite/UI/Menus/`

### Step 6.4: Vendor & Quest UI
**Goal**: Interfaces for vendors and quests
**Method**: Create:
- Vendor shop interface with buy/sell tabs
- Item tooltip with stats
- Quest log with active/completed
- Quest tracker overlay
- Dialog box for NPC conversations
**Reference**: `Sources/Doomnite/UI/GameUI/`

## Phase 7: Audio System

### Step 7.1: Audio Engine
**Goal**: 3D positional audio with environmental effects
**Method**: Implement:
- `AudioManager` using AVAudioEngine
- 3D positional audio with distance attenuation
- Audio source component for entities
- Music playback with crossfading
- Sound effect triggering from game events
**Reference**: `Sources/DoomniteEngine/Audio/`

### Step 7.2: Audio Assets
**Goal**: Sound effects and music integration
**Method**: Create:
- Weapon sound definitions (fire, reload, empty)
- Footstep sounds per surface type
- Ambient zone audio
- UI click sounds
- Music track definitions with looping
**Reference**: `Resources/Audio/` and `Sources/DoomniteShared/AudioData/`

## Phase 8: Networking (Battle Royale Mode)

### Step 8.1: Network Transport Layer
**Goal**: Low-level networking using Network framework
**Method**: Implement:
- `NetworkManager` with connection handling
- UDP-based reliable/unreliable messaging
- Message serialization/deserialization
- Connection quality estimation (ping, packet loss)
**Reference**: `Sources/DoomniteEngine/Networking/Transport/`

### Step 8.2: Game State Synchronization
**Goal**: Sync game state between client and server
**Method**: Implement:
- Snapshot interpolation
- Client-side prediction for movement
- Server reconciliation
- Lag compensation (rewind for hit detection)
- State delta compression
**Reference**: `Sources/DoomniteEngine/Networking/Sync/`

### Step 8.3: Matchmaking & Lobby
**Goal**: Player matchmaking for battle royale
**Method**: Implement:
- Lobby system with player list
- Matchmaking queue
- Server browser (for hosted games)
- Player ready checks
- Game start countdown
**Reference**: `Sources/Doomnite/Game/Networking/`

### Step 8.4: Building System (Fortnite-style)
**Goal**: Place and edit structures in battle royale
**Method**: Implement:
- Buildable structures (wall, floor, stairs, roof)
- Material costs (wood, stone, metal)
- Placement validation and collision
- Edit mode for modifying structures
- Structure health and destruction
**Reference**: `Sources/Doomnite/Game/Building/`

## Phase 9: Performance Optimization

### Step 9.1: Thermal State Management
**Goal**: Prevent device overheating
**Method**: Implement:
- `ThermalManager` monitoring `NSProcessInfo.thermalState`
- Dynamic resolution scaling
- Quality preset adjustments
- User notification of thermal throttling
**Reference**: `Sources/DoomniteEngine/Core/Performance/`

### Step 9.2: GPU Performance Optimization
**Goal**: Optimize Metal rendering performance
**Method**: Implement:
- Draw call batching
- Texture atlasing
- LOD (Level of Detail) system
- Occlusion culling
- GPU profiling instrumentation
**Reference**: `Sources/DoomniteEngine/Rendering/Optimization/`

### Step 9.3: Memory Optimization
**Goal**: Stay within memory budget
**Method**: Implement:
- Texture streaming based on visibility
- Asset unloading for unused content
- Memory warnings handling
- Leak detection in debug builds
**Reference**: `Sources/DoomniteEngine/Core/Memory/`

## Phase 10: Game Content Creation

### Step 10.1: Starting Zone/World
**Goal**: Create the first playable zone
**Method**: Design and implement:
- Zone geometry (using procedural generation or loaded models)
- NPC placements (vendors, quest givers)
- Enemy spawn points
- Loot container placements
- POI definitions
**Reference**: `Resources/Data/Zones/` and `Sources/Doomnite/Game/Content/`

### Step 10.2: Weapon Arsenal
**Goal**: Create diverse weapon roster
**Method**: Define:
- Pistols, rifles, shotguns, SMGs, snipers (hitscan)
- Rocket launchers, grenade launchers (projectile)
- Swords, axes, hammers (melee)
- Unique/legendary weapons with special properties
- Weapon models, textures, sounds
**Reference**: `Resources/Data/Weapons/` and `Resources/Models/Weapons/`

### Step 10.3: Enemy Bestiary
**Goal**: Create diverse enemy types
**Method**: Define:
- Humanoid enemies (soldiers, bandits)
- Monstrous enemies (demons, beasts)
- Boss enemies with unique mechanics
- Enemy models, animations, AI parameters
- Loot tables per enemy type
**Reference**: `Resources/Data/Enemies/` and `Resources/Models/Enemies/`

### Step 10.4: Item Catalog
**Goal**: Createequipment and consumables
**Method**: Define:
- Armor sets (light, medium, heavy)
- Accessories (rings, amulets)
- Consumables (health potions, buffs)
- Crafting materials
- Item icons and descriptions
**Reference**: `Resources/Data/Items/`

### Step 10.5: Quest Lines
**Goal**: Create engaging quest chains
**Method**: Design:
- Tutorial quests introducing game systems
- Main story quest chain
- Side quests with unique rewards
- Daily/weekly repeatable quests
- Quest dialog and NPC interactions
**Reference**: `Resources/Data/Quests/`

## Phase 11: App Integration & Polish

### Step 11.1: App Delegate & Entry Point
**Goal**: iOS app lifecycle integration
**Method**: Implement:
- `AppDelegate` with Metal view setup
- Background/foreground handling
- Save game on exit
- Crash reporting integration
**Reference**: `Sources/Doomnite/App/`

### Step 11.2: Save/Load System
**Goal**: Persistent game state
**Method**: Implement:
- `SaveManager` with JSON or binary serialization
- Player progression saving
- World state persistence
- Cloud save support (iCloud)
- Multiple save slots
**Reference**: `Sources/Doomnite/Game/SaveLoad/`

### Step 11.3: Settings System
**Goal**: User-configurable settings
**Method**: Implement:
- Graphics settings (quality presets, FPS cap)
- Audio settings (master, music, SFX volumes)
- Control settings (sensitivity, inversion)
- Settings persistence
**Reference**: `Sources/Doomnite/Game/Settings/`

### Step 11.4: Final Polish
**Goal**: Production-ready polish
**Method**: Add:
- Loading screens with tips
- Achievement system
- Tutorial popups
- Error handling and recovery
- Analytics integration (privacy-compliant)
**Reference**: `Sources/Doomnite/Game/Polish/`

## Phase 12: Build & Distribution

### Step 12.1: IPA Build Script
**Goal**: Manual IPA generation script
**Method**: Create:
- `build_ipa.sh` script
- Archive the app using `xcodebuild`
- Export IPA with proper provisioning
- Code signing verification
**Reference**: `build_ipa.sh` (root)

### Step 12.2: Release Configuration
**Goal**: Optimized release build settings
**Method**: Update `project.yml`:
- Dead code stripping
- Link-time optimization (LTO)
- Swift optimization: `-O -whole-module-optimization`
- Strip debug symbols for release
**Reference**: `project.yml`

### Step 12.3: Final Testing & Validation
**Goal**: Ensure production readiness
**Method**: Perform:
- Playthrough of all game modes
- Performance profiling with Instruments
- Memory leak detection
- Thermal behavior validation
- Device compatibility testing
**Reference**: Manual testing

# 5. TESTING AND VALIDATION

## Success Criteria

### Functional Requirements
1. **Game Launches**: App opens without crashes on iOS 17+ devices
2. **Rendering**: Metal renderer displays 3D world at target FPS (60 FPS)
3. **Controls**: Touch and gamepad input responsive and configurable
4. **Combat**: Weapons fire, hit detection works, damage applied correctly
5. **AI**: Enemies patrol, detect player, attack, and use cover
6. **Inventory**: Items can be picked up, equipped, and used
7. **Vendors**: Buy/sell transactions work with proper pricing
8. **Quests**: Objectives track, complete, and reward properly
9. **World**: Zones load, weather changes, day/night cycle functions
10. **Multiplayer** (if implemented): Network play works without desync

### Performance Requirements
1. **Frame Rate**: Consistent 60 FPS (or 120 FPS on ProMotion) with <5ms frame time
2. **Thermal**: Device remains below thermal throttling during 30min gameplay
3. **Memory**: RAM usage stays below 2GB on target devices
4. **Load Times**: Zone loads in <3 seconds on A14+ devices
5. **Battery**: <15% battery drain per hour of gameplay

### Quality Requirements
1. **No Crashes**: Zero crashes during normal gameplay (tested 1hr session)
2. **No Stub Code**: All systems fully implemented (no TODOs or placeholders)
3. **Asset Completeness**: All referenced assets exist and load correctly
4. **Code Quality**: No compiler warnings, SwiftLint clean
5. **Documentation**: Critical systems documented in-code

## Validation Methods

### Automated Validation
- Swift compiler warnings treated as errors (`-Werror`)
- Unit tests for ECS, math, and data structures (if time permits)
- Metal validation layer enabled in debug

### Manual Testing Checklist
1. **New Game Flow**: Create character → tutorial → open world
2. **Combat Testing**: Use every weapon type against each enemy type
3. **Vendor Testing**: Buy/sell all item categories
4. **Quest Testing**: Complete each quest type
5. **Performance Testing**: 30-minute session monitoring FPS and thermal
6. **Edge Cases**: Inventory full, no ammo, low health scenarios

### Profiling Tools
- **Metal System Trace** (Instruments): GPU bound analysis
- **Time Profiler** (Instruments): CPU hot spot identification
- **Allocations** (Instruments): Memory leak detection
- **Energy Log** (Instruments): Battery/thermal impact

## Final Deliverable
A complete, production-ready iOS game (`Doomnite`) that:
- Can be built into an IPA using the provided build script
- Runs natively on iOS with Metal-optimized rendering
- Contains full game systems with no demo/stub code
- Is performant and thermally safe for extended play sessions
- Is ready for App Store submission (pending Apple review guidelines compliance)
