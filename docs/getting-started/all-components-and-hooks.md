---
title: All Components and Hooks
description: Complete reference for all available components and hooks in @react-three/viverse.
nav: 1
---

## Components

## `<Viverse>`

The main provider component that sets up VIVERSE authentication and physics context. Must wrap your entire application or the parts that use VIVERSE features.

**Props:**

- `children?: ReactNode` - Child components
- `loginRequired?: boolean` - Forces user to login before playing (default: `false`)
- `checkAuth?: checkAuthOptions` - Authentication check options
- `clientId?: string` - VIVERSE app client ID (can also be set via `VITE_VIVERSE_APP_ID` environment variable).
- `domain?: string` - Authentication domain (default: `'account.htcvive.com'`)
- `authorizationParams?: object` - Additional authorization parameters
- `cookieDomain?: string` - Cookie domain for authentication
- `httpTimeoutInMS?: number` - HTTP request timeout in milliseconds

> [!WARNING]
> Don't set the `clientId` during local development!

**Example:**

```tsx
<Viverse loginRequired={true} clientId="your-app-id">
  <YourGame />
</Viverse>
```

### `<SimpleCharacter>`

Creates a simple character controller with physics based on three-mesh-bvh, walking, running, jumping actions, and camera controls. Automatically uses the active VIVERSE avatar if authenticated.

**Props:** See [SimpleCharacter Options](#simplecharacter-options) section below for complete details.

**Example:**

```tsx
<SimpleCharacter walk={{ speed: 3 }} run={{ speed: 6 }} jump={{ speed: 10 }}>
  {/* Optional child components */}
</SimpleCharacter>
```

### `<BvhPhysicsWorld>`

Provides physics context for collision detection. Usually wrapped automatically by `<Viverse>`, but can be used standalone.

**Props:**

- `children?: ReactNode` - Child components

### `<BvhPhysicsBody>`

Adds visible children as static (non-moving) or kinematic (moving) objects as obstacles to the physics world.

> [!WARNING]
> Content inside the object can not structurally change.

**Props:**

- `children?: ReactNode` - Static mesh objects for collision
- `kinematic?: boolean` - whether the objects world transformation can change - default: false

**Example:**

```tsx
<BvhPhysicsBody>
  <mesh>
    <boxGeometry />
    <meshStandardMaterial />
  </mesh>
</BvhPhysicsBody>
```

### `<BvhPhysicsSensor>`

Adds visible children as sensors to the physics world. Sensors detect when the player character intersects with them without blocking movement.

> [!WARNING]
> Content inside the sensor cannot structurally change. Hiding the sensor's content requires wrapping it in `<group visible={false}>...</group>`.

**Props:**

- `children?: ReactNode` - Mesh objects that define the sensor area
- `isStatic?: boolean` - Whether the sensor's world transformation is static (default: `true`)
- `onIntersectedChanged?: (intersected: boolean) => void` - Callback invoked when the player starts or stops intersecting with the sensor

**Use Cases:**
- Trigger zones (door openers, checkpoint areas)
- Collectible item detection
- Environmental effects (entering water, lava, etc.)
- Quest area boundaries

**Example:**

```tsx
<BvhPhysicsSensor 
  isStatic={true}
  onIntersectedChanged={(intersected) => {
    console.log("Player in zone:", intersected)
  }}
>
  <group visible={false}>
    <mesh>
      <boxGeometry args={[5, 3, 5]} />
    </mesh>
  </group>
</BvhPhysicsSensor>
```

**Moving Sensor Example:**

```tsx
<BvhPhysicsSensor 
  isStatic={false}
  onIntersectedChanged={(intersected) => {
    if (intersected) {
      console.log("Player touched moving platform!")
    }
  }}
>
  <mesh position={[0, Math.sin(Date.now() / 1000) * 2, 0]}>
    <boxGeometry />
  </mesh>
</BvhPhysicsSensor>
```

### `<PrototypeBox>`

A quick prototyping component that renders a textured box with the prototype material.

**Props:**

- `color?: ColorRepresentation` - Box color tint
- All standard Three.js Group props (position, rotation, scale, etc.)

**Example:**

```tsx
<PrototypeBox position={[0, 1, 0]} scale={[2, 1, 3]} color="red" />
```

### `<CharacterModelBone>`

Component for placing content inside character models (works with both VRM and GLTF models).

**Props:**

- `bone: VRMHumanBoneName` - The bone name to access (uses VRM bone naming convention)
- `children?: ReactNode` - Child components to attach to the bone

**Example:**

```tsx
<SimpleCharacter>
  <CharacterModelBone bone="rightHand">
    <SwordModel />
  </CharacterModelBone>
</SimpleCharacter>
```

> [!NOTE]
> This component works with both VRM models and GLTF models that follow the VRM bone naming convention.

## Hooks

| Hook                             | Description                                                   | Returns                                                          |
| -------------------------------- | ------------------------------------------------------------- | ---------------------------------------------------------------- |
| `useViverseClient()`             | Returns the VIVERSE client instance for making API calls      | `Client`                                                         |
| `useViverseAuth()`               | Returns the current authentication state                      | Auth object with access tokens, or `undefined`                   |
| `useViverseAvatarClient()`       | Returns the avatar client for avatar-related operations       | `AvatarClient \| undefined`                                      |
| `useViverseLogin()`              | Returns a function to initiate the VIVERSE login flow         | Login function                                                   |
| `useViverseLogout()`             | Returns a function to initiate the VIVERSE logout flow        | Logout function                                                  |
| `useViverseProfile()`            | Fetches the user's profile (name, avatar info) using Suspense | Profile object with `name`, `activeAvatar`, etc., or `undefined` |
| `useViverseActiveAvatar()`       | Fetches the user's currently selected avatar using Suspense   | Avatar object with `vrmUrl`, `headIconUrl`, etc., or `undefined` |
| `useViverseAvatarList()`         | Fetches the user's personal avatar collection using Suspense  | Array of avatar objects, or `undefined`                          |
| `useViversePublicAvatarList()`   | Fetches publicly available avatars using Suspense             | Array of public avatar objects, or `undefined`                   |
| `useViversePublicAvatarByID(id)` | Fetches a specific public avatar by ID using Suspense         | Avatar object, or `undefined`                                    |
| `useIsMobile()`                  | Returns `true` on touch-centric/mobile devices (media query)  | `boolean`                                                        |

> [!NOTE]
> `useViverseClient()` returns `undefined` if not within a `<Viverse>` provider or if no `clientId` is provided. Also all avatar-related hooks return `undefined` when the user is not authenticated.

### useIsMobile

Lightweight media-query based mobile detection. It subscribes to `@media (hover: none) and (pointer: coarse)`.

```tsx
import { useIsMobile } from '@pmndrs/viverse/react'

function MobileOnlyUI() {
  const isMobile = useIsMobile()
  return isMobile ? <div>Shown on mobile</div> : null
}
```

## SimpleCharacter Options

The `SimpleCharacter` component can be configured with a variety of props but also supports all the default group props, such as position, rotation, and scale.

### `useViverseAvatar` flag

Allows to configure whether the users vrm avatar should be displayed as the character model.

- **Default:** `true`

### `movement` Options

- **walk:** `object | boolean` - Enable walking (default: `true`)
  - **speed:** Movement speed in units per second (default: `2.5`)
  - Set to `false` to disable walking

- **run:** `object | boolean` - Enable running (default: `true`)
  - **speed:** Running speed in units per second (default: `4.5`)
  - Set to `false` to disable running

- **run:** `object | boolean` - Enable jumping (default: `true`)
  - **delay:** Time before jump starts in seconds (default: `0.15`)
  - **bufferTime:** Jump input buffer time in seconds (default: `0.1`)
  - **speed:** Jump velocity in units per second (default: `8`)
  - Set to `false` to disable jumping

### `input` Options

Either a array of `Input` objects or a custom `InputSystem`

- **Default:** `[ScreenJoystickInput, ScreenJumpButtonInput, PointerCaptureInput, LocomotionKeyboardInput]`
- Configure input handling with custom input classes

**Available Input Classes provided by @pmndrs/viverse:**

- `LocomotionKeyboardInput` - WASD movement, Space for jump, Shift for run
  - **Options:**
    - `keyboardMoveForwardKeys?: string[]` - Keys for forward movement (default: `['KeyW']`)
    - `keyboardMoveBackwardKeys?: string[]` - Keys for backward movement (default: `['KeyS']`)
    - `keyboardMoveLeftKeys?: string[]` - Keys for left movement (default: `['KeyA']`)
    - `keyboardMoveRightKeys?: string[]` - Keys for right movement (default: `['KeyD']`)
    - `keyboardRunKeys?: string[]` - Keys for running (default: `['ShiftRight', 'ShiftLeft']`)
    - `keyboardJumpKeys?: string[]` - Keys for jumping (default: `['Space']`)
  - **Example:**
    ```tsx
    <SimpleCharacter
      inputOptions={{
        keyboardMoveForwardKeys: ['KeyW', 'ArrowUp'],
        keyboardJumpKeys: ['Space', 'KeyJ']
      }}
    />
    ```

- `PointerCaptureInput` - Mouse look with pointer capture (requires manual `setPointerCapture`)
  - **Options:**
    - `pointerCaptureRotationSpeed?: number` - Camera rotation sensitivity (default: `0.4`)
    - `pointerCaptureZoomSpeed?: number` - Zoom sensitivity for mouse wheel (default: `0.0001`)
  - **Features:**
    - Single-finger/mouse drag for camera rotation
    - Two-finger pinch for zoom on touch devices
    - Mouse wheel zoom support
  - **Example:**
    ```tsx
    <SimpleCharacter
      inputOptions={{
        pointerCaptureRotationSpeed: 0.6,
        pointerCaptureZoomSpeed: 0.0002
      }}
    />
    ```

- `PointerLockInput` - Mouse look with pointer lock (requires manual `requestPointerLock`)
  - **Options:**
    - `pointerLockRotationSpeed?: number` - Camera rotation sensitivity (default: `0.4`)
    - `pointerLockZoomSpeed?: number` - Zoom sensitivity for mouse wheel (default: `0.0001`)
  - **Features:**
    - Unlimited mouse movement (cursor is hidden and locked)
    - Mouse wheel zoom support
  - **Usage:** Call `domElement.requestPointerLock()` on user interaction
  - **Example:**
    ```tsx
    <Canvas onClick={(e) => (e.target as HTMLElement).requestPointerLock()}>
      <SimpleCharacter input={[PointerLockInput, LocomotionKeyboardInput]} />
    </Canvas>
    ```

- `ScreenJoystickInput` - On-screen joystick for movement and run (mobile)
  - **Options:**
    - `screenJoystickDeadZonePx?: number` - Dead zone radius in pixels (default: `24`)
    - `screenJoystickRunDistancePx?: number` - Distance threshold for running in pixels (default: `46`)
  - **Features:**
    - Automatically hidden on non-mobile devices (CSS `.mobile-only` class)
    - Positioned bottom-left by default
    - Customizable via CSS targeting `.viverse-joystick` class
  - **Example:**
    ```tsx
    <SimpleCharacter
      inputOptions={{
        screenJoystickDeadZonePx: 30,
        screenJoystickRunDistancePx: 50
      }}
    />
    ```

- `ScreenJumpButtonInput` - On-screen jump button (mobile-only)
  - **Features:**
    - Automatically hidden on non-mobile devices
    - Positioned bottom-right by default
    - Customizable via CSS targeting `.viverse-jump` class
  - **No configurable options**

**Customizing Mobile UI:**

You can customize the appearance and position of mobile controls using CSS:

```css
/* Customize joystick position and appearance */
.viverse-joystick {
  bottom: 40px !important;
  left: 40px !important;
  width: 140px !important;
  height: 140px !important;
  background: rgba(100, 150, 255, 0.3) !important;
}

/* Customize jump button */
.viverse-jump {
  bottom: 50px !important;
  right: 50px !important;
  background: rgba(255, 100, 100, 0.4) !important;
}
```

### `model` Options

- **url:** `string` - URL to VRM or GLTF model file
- **type:** `"gltf" | "vrm"` - the type of file to be loaded (optional)
- **castShadow:** `boolean` - Enable shadow casting (default: `true`)
- **receiveShadow:** `boolean` - Enable shadow receiving (default: `true`)
- **boneRotationOffset:** `Quaternion | undefined` - Allows to apply an rotation offset when placing objects as children of the character's bones (default: `undefined`)
- Set to `false` to disable model loading
- Set to `true` or omit to use default robot model

### `physics` Options

- **capsuleRadius:** `number` - Character collision capsule radius (default: `0.4`)
- **capsuleHeight:** `number` - Character collision capsule height (default: `1.7`)
- **gravity:** `number` - Gravity acceleration in m/s² (default: `-20`)
- **linearDamping:** `number` - Air resistance coefficient (default: `0.1`)
- **maxGroundSlope:** `number` - Max slope tangent value for a collider to be detected as a walkable ground (default: `0.5`, approximately 26.6°)
- **updatesPerSecond:** `number` - Physics update rate (default: `60`)

### `cameraBehavior` Options

- **collision:** `object | boolean` - Enable camera collision (default: `true`)
  - **offset:** `number` - Collision offset distance (default: `0.2`)

- **characterBaseOffset:** `Vector3 | [number, number, number]` - Camera position relative to character (default: `[0, 1.3, 0]`)

- **rotation:** `object | boolean` - Enable camera rotation (default: `true`)
  - **minPitch:** `number` - Minimum pitch angle (default: `-Math.PI/2`)
  - **maxPitch:** `number` - Maximum pitch angle (default: `Math.PI/2`)
  - **minYaw:** `number` - Minimum yaw angle (default: `-Infinity`)
  - **maxYaw:** `number` - Maximum yaw angle (default: `+Infinity`)
  - **speed:** `number` - Rotation speed multiplier (default: `1000`)

- **zoom:** `object | boolean` - Enable camera zoom (default: `true`)
  - **speed:** `number` - Zoom speed multiplier (default: `1000`)
  - **minDistance:** `number` - Minimum camera distance (default: `1`)
  - **maxDistance:** `number` - Maximum camera distance (default: `7`)

### `animation` Options

- **yawRotationBasedOn:** `'camera' | 'movement'` - Character rotation basis (default: `'movement'`)
- **maxYawRotationSpeed:** `number` - Maximum rotation speed (default: `10`)
- **crossFadeDuration:** `number` - Animation blend time in seconds (default: `0.3`)

The `SimpleCharacter` uses the following animations `walk`, `run`, `idle`, `jumpForward`, `jumpUp`, `jumpLoop`, `jumpDown` each with the following options:

- **url:** `string` - Animation file URL
- **type:** `'fbx' | 'gltf' | 'vrma'` - Animation file type (optional)
- **boneMap** - Allows to map the bone names of the animation amature to the standard VRM bone names
- **removeXZMovement:** `boolean` - Remove horizontal movement from animation
- **trimTime:** `{ start?: number; end?: number }` - Trim animation timing
- **scaleTime:** `number` - Scale animation playback speed

## PrototypeMaterial

The `<prototypeMaterial>` component provides a textured material for prototyping using kenney.nl's prototype texture.

- **color:** `ColorRepresentation` - Material color tint
- **repeat:** `Vector2` - Texture repeat pattern (accessible as `materialRef.current.repeat`)
- All standard Three.js MeshPhongMaterial properties

```tsx
// As JSX element
<mesh>
  <boxGeometry />
  <prototypeMaterial color="blue" />
</mesh>
```
