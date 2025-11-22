# Defold 3D Depth of Field (DoF)


> [!WARNING]
> You have to set `shader.exclude_gles_sm100 = 1 in your game.project`

```ini
[shader]
exclude_gles_sm100 = 1
```


---

## Toss a Coin to Your Witcher
If you find my [Defold Extensions](https://github.com/selimanac) useful for your projects, please consider [supporting](https://github.com/sponsors/selimanac) it.  
I'd love to hear about your projects! Please share your released projects that use my native extensions. It would be very motivating for me.


---

## API Reference

The DoF module (`dof/dof.lua`) provides the following API functions:

### Configuration Functions

#### `dof.set_focus(focus_x, focus_y)`
Set the focus point in screen space (normalized coordinates 0.0-1.0).

**Parameters:**
- `focus_x` (number): X coordinate (0.0 = left, 1.0 = right, 0.5 = center)
- `focus_y` (number): Y coordinate (0.0 = bottom, 1.0 = top, 0.5 = center)

**Example:**
```lua
local dof = require("dof.dof")

-- Set focus to screen center
dof.set_focus(0.5, 0.5)

-- Update focus based on mouse position
function on_input(self, action_id, action)
    local w, h = window.get_size()
    dof.set_focus(action.screen_x / w, action.screen_y / h)
end
```

#### `dof.set_distance(min_distance, max_distance)`
Set the depth difference range for blur effect.

**Parameters:**
- `min_distance` (number): Depth difference where blur starts
- `max_distance` (number): Depth difference where blur reaches maximum strength

**Example:**
```lua
local dof = require("dof.dof")

-- Blur objects with depth difference greater than 0.8 from focus point
dof.set_distance(0.8, 1.0)

-- Larger range for gradual blur transition
dof.set_distance(5.0, 15.0)
```

#### `dof.set_camera_params(near_plane, far_plane)`
Set the camera near and far plane distances for depth linearization.

**Parameters:**
- `near_plane` (number): Camera near plane distance (must match your camera settings)
- `far_plane` (number): Camera far plane distance (must match your camera settings)

**Example:**
```lua
local dof = require("dof.dof")

-- Set camera parameters matching your camera component
-- These values must match the camera's near_z and far_z settings
dof.set_camera_params(0.1, 1000.0)
```

#### `dof.set_gaussian_blur(sigma, kernel_size)`
Configure Gaussian blur parameters (recommended).

**Parameters:**
- `sigma` (number): Standard deviation controlling blur strength (typical: 1.0-5.0, recommended: 2.0)
- `kernel_size` (number): Samples per direction (typical: 3-5, higher = better quality but slower)

**Performance:**
- kernel_size 3: 14 samples total (7 horizontal + 7 vertical)
- kernel_size 5: 22 samples total (11 horizontal + 11 vertical)

**Example:**
```lua
local dof = require("dof.dof")

-- Balanced quality and performance
dof.set_gaussian_blur(2.0, 3)

-- High quality for desktop
dof.set_gaussian_blur(3.0, 5)

-- Performance mode for mobile
dof.set_gaussian_blur(1.5, 2)
```

#### `dof.set_box_blur(size, separation)`
Configure box blur parameters (alternative method, slower than Gaussian).

**Parameters:**
- `size` (number): Kernel radius in pixels (typical: 2-5)
- `separation` (number): Spacing between samples (typical: 1.0-2.0)

**Note:** To use box blur, you must modify `dof/dof.lua` to uncomment box blur code and comment out Gaussian blur code.

**Example:**
```lua
local dof = require("dof.dof")

-- Basic box blur
dof.set_box_blur(2, 1.0)
```

### Render Functions

#### `dof.init(self)`
Initialize the DoF rendering. Must be called from your render script's `init()` function.

**Parameters:**
- `self` (table): Render script instance

**Example:**
```lua
local dof = require("dof.dof")

function init(self)
    -- Your existing initialization...
    dof.init(self)
end
```

#### `dof.render_update(self, state, predicates, draw_options_world)`
Render scene with depth buffer. Call from render script's `update()` before `dof.render()`.

**Parameters:**
- `self` (table): Render script instance
- `state` (table): Render state with `window_width`, `window_height`, `clear_buffers`
- `predicates` (table): Render predicates (must include `predicates.model`)
- `draw_options_world` (table): World rendering options

**Example:**
```lua
local dof = require("dof.dof")

function update(self)
    -- Your rendering setup...
    dof.render_update(self, state, predicates, draw_options_world)
    dof.render(self, state, predicates)
end
```

#### `dof.render(self, state, predicates)`
Apply blur and composite final DoF result. Call from render script's `update()` after `dof.render_update()` and `reset_camera_world(state)`.

**Parameters:**
- `self` (table): Render script instance
- `state` (table): Render state with `window_width`, `window_height`
- `predicates` (table): Render predicates (must include `predicates.dof_rt`)

## Gaussian Blur Tuning Guide

### Parameters Overview

#### `gaussian_sigma` (Standard Deviation)
Controls the **strength/spread** of the blur.

- **Lower values (1.0-2.0)**: Subtle, sharp blur
- **Higher values (3.0-5.0)**: Strong, soft blur
- **Recommended**: 2.0 for balanced quality

#### `gaussian_kernel_size` (Kernel Radius)
Controls the **number of samples** taken in each direction.

- kernel_size 3 = 7 samples per pass (center + 3 on each side)
- kernel_size 5 = 11 samples per pass (center + 5 on each side)
- **Recommended**: 3-5 for good quality/performance balance

### Relationship Between Sigma and Kernel Size

For optimal results, kernel size should be approximately `3 × sigma`:
- sigma = 1.0 → kernel_size = 3
- sigma = 2.0 → kernel_size = 5-7
- sigma = 3.0 → kernel_size = 7-11

This ensures the Gaussian kernel captures ~99% of the blur contribution.

### Performance vs Quality

#### For Better Performance
- Reduce `gaussian_kernel_size` (e.g., 2-3)
- Keep `gaussian_sigma` moderate (1.5-2.5)

#### For Better Quality
- Increase `gaussian_kernel_size` (e.g., 5-7)
- Increase `gaussian_sigma` for softer blur (2.5-4.0)

### Platform-Specific Recommendations

#### Mobile Devices
```lua
dof.set_gaussian_blur(1.5, 2)  -- Optimized for performance
dof.set_distance(0.8, 1.0)     -- Tighter distance range
```

#### Desktop
```lua
dof.set_gaussian_blur(2.5, 5)  -- Higher quality
dof.set_distance(5.0, 15.0)    -- Wider distance range
```

---

## Credits

* Textures by Kenney (https://www.kenney.nl)
* Original DoF shader from [3D Game Shaders for Beginners](https://lettier.github.io/3d-game-shaders-for-beginners/depth-of-field.html)
* [Wang Dada](https://www.artstation.com/ggfif1234) Character by PixelAudaz   (https://sketchfab.com/3d-models/wang-dada-character-1f756491654844e789175c8f9d3efd93)
* Moped by Alyona Shek (https://sketchfab.com/3d-models/moped-a8f0d74334034ca1bcf397f21c26a4c3)
* Gaussian Blur reference by Nikita Lisitsa (https://lisyarus.github.io/blog/posts/blur-coefficients-generator.html)
