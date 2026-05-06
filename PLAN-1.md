# 1. OBJECTIVE

Build **Doomnite** - a full production-grade iOS game with a custom Metal-based rendering engine, featuring:
- Open-world RPG exploration with quest systems (Elder Scrolls-inspired)
- Fast-paced FPS combat with diverse weapons, special abilities, and elemental effects (Doom + Destiny-inspired)
- Deep weapon/armor upgrade systems using runes, books, and god-tier ruins (Minecraft + Elder Scrolls-inspired)
- Upgradable animal companions (dragons, pets) with combat abilities
- Social hubs, vendors, equipment customization, and character progression (Animal Jam-inspired)
- Battle royale mode with building mechanics and multiplayer support (Fortnite-inspired)

The goal is to deliver a complete, optimized iOS game with native performance using Swift, Metal, and a custom Entity Component System (ECS) architecture. The project uses XcodeGen for project generation via `project.yml` and will be manually built into an IPA. All systems are fully production-implemented with no stub/demo code, and no references to source games exist in the codebase.

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
- **Special Weapon Abilities**: Destiny-inspired exotic weapons with unique active/passive abilities
  - Staffs with energy blasts and area-of-effect attacks
  - Guns with charge shots, ricochet, and elemental modifications
  - Melee weapons with special combos and ground slams
- **Weapon Upgrade System**: Minecraft-style upgrade mechanics
  - Runes that can be socketed into weapons (fire, ice, lightning, poison, etc.)
  - Books that teach weapon new abilities or increase stats
  - Upgrade stations at villages/blacksmiths
  - Rarity tiers: Common → Uncommon → Rare → Epic → Legendary → God-Tier
- **God Ruins System**: Elder Scrolls-inspired powerful artifacts
  - Ancient ruins scattered throughout the world (dungeons, caves, secret areas)
  - God-tier ruins apply insane damage multipliers (2x, 5x, 10x) or crazy effects
  - Elemental god ruins: "Ruin of Inferno" (fire damage + burn), "Ruin of Tempest" (lightning chain)
  - Set bonuses when combining multiple god ruins
- **Magic System**: Elder Scrolls-style spells and enchantments
  - Spell books that teach new magic abilities
  - Mana system for casting spells
  - Spell types: destruction, restoration, conjuration, illusion
  - Weapon enchantments (fire, frost, shock, soul trap)
- **Armor System**: Multi-layered defensive equipment
  - Light/Medium/Heavy armor sets with set bonuses
  - Armor enchantments and upgrades
  - Shields with block ratings and parry mechanics
  - Helmet/Boots/Gloves with special abilities
- **Animal Companion System**: Upgradable creatures with combat abilities
  - Dragons: fire breath, flight, massive damage, different elements
  - Wolves: pack tactics, speed, bleed attacks
  - Bears: tanking, ground slam, high HP
  - Birds: scouting, dropping bombs, revealing secrets
  - Companion leveling and evolution (baby → adult → alpha)
  - Companion equipment (saddles, collars with stats)
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
**Goal**: Complete weapon and damage system with special abilities
**Method**: Implement:
- `WeaponDefinition` (damage, fire rate, accuracy, recoil)
- Hitscan weapons (instant raycast)
- Projectile weapons (physics-based)
- Melee weapons (swing detection)
- Damage calculation (armor reduction, critical hits)
- Hit feedback (screen shake, sound, particles)
- Death and respawn system
- **Special Weapon Abilities**:
  - `WeaponAbility` component (active/passive abilities)
  - Staff abilities: energy blast (hold charge), area shockwave, homing projectiles
  - Gun abilities: ricochet shot, explosive rounds, time-slow aim, triple shot
  - Melee abilities: ground slam (AOE), whirlwind spin, shield bash
  - Ability cooldown system with UI indicators
  - Mana/energy cost for special abilities
**Reference**: `Sources/Doomnite/Game/Combat/`

### Step 5.4.1: Magic System (Elder Scrolls-style)
**Goal**: Full magic system with spells and enchantments
**Method**: Implement:
- `SpellBook` item that teaches new spells to player
- `ManaComponent` for spell casters
- Spell types:
  - Destruction: Fireball, Lightning Bolt, Ice Spike, Chain Lightning
  - Restoration: Heal, Cure Poison, Restore Mana, Resurrection
  - Conjuration: Summon Wolf, Summon Bear, Summon Dragon, Bound Weapon
  - Illusion: Invisibility, Charm Enemy, Fear AOE, Paralyze
- `EnchantmentSystem` for adding magic effects to weapons/armor
- Enchantment types: Fire/Frost/Shock damage, Soul Trap, Absorb Health, Banish Undead
- Spell casting with gesture-based controls (touch swipe patterns)
- Magic effects with particle systems and shaders
**Reference**: `Sources/Doomnite/Game/Magic/`

### Step 5.4.2: Weapon Upgrade & Rune System (Minecraft-style)
**Goal**: Deep weapon customization with runes and books
**Method**: Implement:
- `RuneItem` that can be socketed into weapons (max 3 rune slots)
- Rune types with effects:
  - Fire Rune: +fire damage, burn effect over time
  - Ice Rune: +cold damage, slow effect, freeze on crit
  - Lightning Rune: +shock damage, chain lightning to nearby enemies
  - Poison Rune: Damage over time, reduce enemy defense
  - Vampiric Rune: Heal on hit, life steal percentage
  - Swift Rune: +attack speed, +movement speed
  - Critical Rune: +crit chance, +crit damage
- `UpgradeBook` that teaches permanent weapon stat increases:
  - "Book of Damage" → +10% weapon damage
  - "Book of Stability" → +accuracy, -recoil
  - "Book of Elements" → Unlock elemental mode switching
- `UpgradeStation` entity in villages for applying runes/books
- Visual effects on weapons showing active runes (glowing auras)
- Rune combining system (3 minor runes = 1 major rune)
**Reference**: `Sources/Doomnite/Game/Upgrades/`

### Step 5.4.3: God Ruins System
**Goal**: Epic end-game content with insane power-ups
**Method**: Implement:
- `GodRuin` items found in dangerous dungeons/ruins
- Ruin tiers: Minor (+50% damage), Major (+100% damage), God-Tier (+500% damage, crazy effects)
- Elemental God Ruins:
  - "Ruin of Inferno": +300% fire damage, enemies explode on death
  - "Ruin of Tempest": +300% lightning damage, chain lightning on every hit
  - "Ruin of Permafrost": +300% ice damage, freeze entire screen on kill
  - "Ruin of Void": +300% shadow damage, teleport behind enemy on hit
- Unique God Ruins:
  - "Ruin of the Berserker": +500% damage but -50% defense
  - "Ruin of the Vampire King": +200% damage, 20% lifesteal
  - "Ruin of the Earthquake": Every 10th hit causes screen-wide AOE
- God Ruin dungeons with boss fights to earn them
- Visual transformation when wielding God Ruin weapons (screen effects, aura)
- Set bonuses: "Wearing 3 Fire God Ruins → Immune to fire, enemies burn near you"
**Reference**: `Sources/Doomnite/Game/GodRuins/`

### Step 5.4.4: Armor System Enhancement
**Goal**: Multi-layered armor with set bonuses and enchantments
**Method**: Implement:
- Armor slots: Helmet, Chest, Legs, Boots, Gloves, Shield
- Armor types:
  - Light Armor: +agility, +crit chance, less defense
  - Medium Armor: Balanced defense and mobility
  - Heavy Armor: +defense, +health, -movement speed
- Armor sets with bonuses:
  - "Dragon Set" (5 pieces): +50% fire resistance, flame breath attack
  - "Shadow Set" (5 pieces): Invisibility when standing still, +crit damage
  - "Titan Set" (5 pieces): +100% HP, ground slam on heavy fall
- Armor enchantments: Fortify Health, Resist Elements, Water Breathing, Feather (carry weight)
- Visual armor customization (dyes, cape attachments)
- Durability system with repair at blacksmiths
**Reference**: `Sources/Doomnite/Game/Armor/`

### Step 5.4.5: Animal Companion System
**Goal**: Upgradable animal companions with combat abilities
**Method**: Implement:
- `AnimalCompanion` component with stats and abilities
- Companion types:
  - **Dragons**: 
    - Fire/ice/lightning variants
    - Abilities: Fire Breath (cone AOE), Wing Buffet (knockback), Tail Whip (spin attack)
    - Evolution: Dragon Egg → Baby Dragon → Adult Dragon → Elder Dragon → Ancient Dragon
    - Dragon armor slots (scale reinforcement, saddle)
  - **Wolves**: 
    - Pack tactics (summon 2 additional wolves at max level)
    - Abilities: Bleed Bite (DOT), Howl (fear enemies), Pin (hold enemy down)
    - Evolution: Wolf Pup → Timber Wolf → Dire Wolf → Alpha Wolf
  - **Bears**:
    - Tank role with high HP and defense
    - Abilities: Ground Slam (AOE stun), Maul (high damage), Hibernation (heal over time)
    - Evolution: Bear Cub → Black Bear → Grizzly Bear → Cave Bear
  - **Birds** (Hawks/Owls):
    - Scouting role, reveal secrets on map
    - Abilities: Dive Bomb (instant damage), Screech (stun), Carry Item (fetch loot)
    - Evolution: Chick → Adult Bird → Hunting Bird → Storm Bird
- Companion leveling system (gains XP from combat)
- Companion equipment: Collars, Saddles, Armor with stat bonuses
- Companion AI: Follow, Attack, Stay, Patrol commands
- Companion inventory (can carry items for player)
- Emotional bond system (feed, pet → bonuses)
**Reference**: `Sources/Doomnite/Game/Companions/`

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
- NPC placements (vendors, quest givers, spell merchants, blacksmiths)
- Enemy spawn points
- Loot container placements
- POI definitions (dungeons, ruins, upgrade stations)
- God Ruin dungeon entrances (high-level areas)
**Reference**: `Resources/Data/Zones/` and `Sources/Doomnite/Game/Content/`

### Step 10.2: Weapon Arsenal with Special Abilities
**Goal**: Create diverse weapon roster with unique abilities
**Method**: Define:
- **Pistols**: 
  - "Shadow Pistol" (Destiny-style exotic): Shots phase through enemies, hit 3 targets
  - "Flame Burster": +fire damage, burn effect
  - "Void Amplifier": +shadow damage, +crit chance
- **Rifles**:
  - "Stormcaller" (Exotic): Lightning chains to 5 enemies on headshot
  - "Ice Piercer": +ice damage, freeze on 3 consecutive hits
  - "Sniper of the Ancients": +200% damage, +accuracy, scope zoom
- **Shotguns**:
  - "Dragon's Breath" (Exotic): Cone of fire that burns for 5 seconds
  - "Thunderclap": Knockback + lightning damage
- **SMGs**:
  - "Vampiric Hive" (Exotic): Life steal on every hit, +fire rate
  - "Toxic Sprayer": Poison DOT, -enemy defense
- **Melee Weapons**:
  - "Godslayer Sword" (Exotic): +500% damage vs bosses, ground slam AOE
  - "Frost Blade": +ice damage, freeze aura around player
  - "Staff of Elements": Switch between fire/ice/lightning modes
- **Unique Abilities per Weapon**:
  - Active: Hold to charge super shot, instant area blast, time-slow aim
  - Passive: +damage after kill for 5s, reveal enemies through walls, +speed on hit
- Weapon models, textures, sounds, and ability VFX
**Reference**: `Resources/Data/Weapons/` and `Resources/Models/Weapons/`

### Step 10.3: Enemy Bestiary
**Goal**: Create diverse enemy types
**Method**: Define:
- Humanoid enemies (soldiers, bandits, mages with spell attacks)
- Monstrous enemies (demons, beasts, dragons)
- Boss enemies with unique mechanics (dragon bosses, ruin guardians)
- Enemy models, animations, AI parameters
- Loot tables per enemy type (including rune drops, spell book drops)
- Elemental variants (fire/ice/lightning versions of each enemy)
**Reference**: `Resources/Data/Enemies/` and `Resources/Models/Enemies/`

### Step 10.4: Item Catalog (Armor, Runes, Books, God Ruins)
**Goal**: Create equipment, upgrade items, and god-tier artifacts
**Method**: Define:

**Armor Sets**:
- "Shadow Assassin Set" (Light): +crit, +speed, invisibility on sneak
- "Flame Knight Set" (Heavy): +fire resist, flame aura, ground slam
- "Storm Mage Set" (Medium): +mana, +spell damage, chain lightning
- "Titan's Bulwark Set" (Heavy): +HP, +defense, earthquake on heavy fall

**Runes** (Socketable):
- Minor Runes: +10% elemental damage, common drops
- Major Runes: +25% elemental damage, rare drops
- Epic Runes: +50% elemental damage + special effect, boss drops
- "Rune of the Phoenix": Revive on death (1 use), legendary drop

**Upgrade Books**:
- "Tome of Destruction": +20% weapon damage (permanent)
- "Codex of Stability": +30% accuracy, -50% recoil
- "Scroll of Elements": Unlock elemental mode switching
- "Manuscript of Mastery": +1 weapon ability slot

**God Ruins** (Epic end-game items):
- "Inferno Ruin" (God-Tier): +500% fire damage, enemies explode, fire immunity
- "Tempest Ruin" (God-Tier): +500% lightning damage, chain lightning on every hit
- "Void Ruin" (God-Tier): +300% shadow damage, teleport behind enemy on kill
- "Earthquake Ruin" (God-Tier): Every 10th hit = screen-wide AOE
- "Vampire King Ruin" (God-Tier): +300% damage, 30% lifesteal
- "Berserker Ruin" (God-Tier): +1000% damage, -90% defense (hardcore mode)

- Item icons, descriptions, rarity colors, inventory models
**Reference**: `Resources/Data/Items/` and `Resources/Data/Ruins/`

### Step 10.5: Animal Companions Content
**Goal**: Create companion types with evolution paths
**Method**: Design:
**Dragons** (Find dragon eggs in fire/ice/lightning caves):
- Fire Dragon: Fire breath (cone), +fire damage to player, flame armor
- Ice Dragon: Frost breath (slow), +ice damage to player, freeze aura
- Lightning Dragon: Lightning breath (chain), +speed to player, shock shield
- Evolution stages with model/stat changes
- Dragon equipment: Scale Mail (+defense), War Saddle (+damage), Amulet of Health

**Wolves** (Tame wolf pups in forest zones):
- Timber Wolf: Bleed bite, pack tactics (summon 2 more at max level)
- Dire Wolf: Massive damage, fear howl, pin enemies down
- Alpha Wolf: Lead the pack, +damage to all nearby companions

**Bears** (Rescue bear cubs from traps):
- Black Bear: Ground slam (AOE stun), high HP
- Grizzly Bear: Maul (massive damage), take reduced damage from frontal attacks
- Cave Bear: Hibernation (heal over time), +carrying capacity for player

**Birds** (Hatch bird eggs found in nests):
- Hawk: Dive bomb (instant damage), reveal loot on mini-map
- Owl: Screech (stun enemies), night vision for player
- Storm Bird: Call lightning, drop bombs from sky

- Companion leveling curves, evolution requirements, equipment stats
- Companion models, animations, ability VFX, UI icons
**Reference**: `Resources/Data/Companions/` and `Resources/Models/Companions/`

### Step 10.6: Spell Books & Magic Content
**Goal**: Create magic system content
**Method**: Design:
**Destruction Spells** (found in mage towers/dungeons):
- "Fireball": Basic fire projectile, AOE on impact
- "Lightning Bolt": Instant lightning damage, chains to nearby enemies
- "Ice Spike": Projectile that slows enemy
- "Chain Lightning": Hits 5 enemies in sequence
- "Meteor Storm" (Master spell): Rain meteors for 10 seconds

**Restoration Spells** (learned from priest NPCs):
- "Heal": Restore 50% HP over 3 seconds
- "Cure Poison": Remove all poison effects
- "Restore Mana": Instant mana refill
- "Resurrection": Revive fallen companion (30 min cooldown)

**Conjuration Spells** (found in dark ruins):
- "Summon Wolf": Temporary wolf companion for 60 seconds
- "Summon Bear": Tank companion with high HP
- "Summon Dragon": Ultimate spell, dragon ally for 30 seconds
- "Bound Sword": Conjure magical sword with +100% damage

**Illusion Spells** (found in thief guilds):
- "Invisibility": Become invisible for 30 seconds (breaks on attack)
- "Charm Enemy": Enemy fights for you for 60 seconds
- "Fear AOE": All nearby enemies flee for 10 seconds
- "Paralyze": Single enemy frozen for 5 seconds

- Mana cost per spell, cooldowns, VFX, sound effects
- Spell book models, merchant NPCs, skill tree unlocks
**Reference**: `Resources/Data/Spells/` and `Resources/Models/Books/`

### Step 10.7: Quest Lines
**Goal**: Create engaging quest chains
**Method**: Design:
- Tutorial quests introducing all systems (combat, magic, companions, upgrades)
- Main story: "The God Ruin Prophecy" (15 quests, 5 chapters)
  - Chapter 1: Discover first ruin, learn about ancient power
  - Chapter 2: Tame first companion, learn bonding
  - Chapter 3: Defeat ruin guardians, collect 3 minor ruins
  - Chapter 4: Upgrade weapons with runes, face epic dragon boss
  - Chapter 5: Claim God-Tier ruin, final battle
- Side quests:
  - "The Blacksmith's Request": Collect 10 dragon scales, unlock upgrade station
  - "Spell Merchant's Apprentice": Learn 5 spells, unlock enchanting
  - "Dragon Tamer": Find and tame all 3 dragon types
  - "Rune Hunter": Collect all 7 rune types, unlock set bonus
- Daily/weekly repeatable quests (for grinding XP and materials)
- Quest dialog, NPC interactions, cutscenes for key moments
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
5. **Special Abilities**: All weapon/staff abilities trigger correctly with proper cooldowns
6. **Magic System**: Spells cast correctly, mana consumed, VFX/sound play
7. **Weapon Upgrades**: Runes socket into weapons, books teach new abilities, stats increase
8. **God Ruins**: God-tier ruins found in dungeons, insane damage/effects apply correctly
9. **Armor System**: All armor slots equip correctly, set bonuses activate
10. **Animal Companions**: Dragons/wolves/bears/birds follow, attack, level up, evolve
11. **AI**: Enemies patrol, detect player, attack, and use cover
12. **Inventory**: Items can be picked up, equipped, and used
13. **Vendors**: Buy/sell transactions work with proper pricing
14. **Quests**: Objectives track, complete, and reward properly (including spell books, runes)
15. **World**: Zones load, weather changes, day/night cycle functions
16. **Multiplayer** (if implemented): Network play works without desync

### Performance Requirements
1. **Frame Rate**: Consistent 60 FPS (or 120 FPS on ProMotion) with <5ms frame time
2. **Thermal**: Device remains below thermal throttling during 30min gameplay (even with God Ruins + dragon companion active)
3. **Memory**: RAM usage stays below 2GB on target devices
4. **Load Times**: Zone loads in <3 seconds on A14+ devices
5. **Battery**: <15% battery drain per hour of gameplay

### Quality Requirements
1. **No Crashes**: Zero crashes during normal gameplay (tested 1hr session with all systems)
2. **No Stub Code**: All systems fully implemented (no TODOs or placeholders)
3. **Asset Completeness**: All referenced assets exist and load correctly (weapons, armor, companions, spells, runes, god ruins)
4. **Code Quality**: No compiler warnings, SwiftLint clean
5. **Documentation**: Critical systems documented in-code
6. **Game Balance**: God Ruins feel powerful but not game-breaking; companion evolution rewarding

## Validation Methods

### Automated Validation
- Swift compiler warnings treated as errors (`-Werror`)
- Unit tests for ECS, math, and data structures (if time permits)
- Metal validation layer enabled in debug
- Memory leak detection in debug builds


## Final Deliverable
A complete, production-ready iOS game (`Doomnite`) that:
- Can be built into an IPA using the provided build script
- Runs natively on iOS with Metal-optimized rendering
- Contains full game systems with no demo/stub code, including:
  - Special weapon abilities (Destiny-style exotics)
  - Weapon upgrade system (Minecraft-style runes + books)
  - God Ruins with insane damage/effects (Elder Scrolls-style artifacts)
  - Magic system with 4 schools of magic and spell books
  - Armor sets with bonuses and enchantments
  - Animal companions (dragons, wolves, bears, birds) with evolution
- Is performant and thermally safe for extended play sessions (even with maxed God Ruins + dragon companion)
- Is ready for App Store submission (pending Apple review guidelines compliance)
