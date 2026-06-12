# Black Hole Visualizer

A real-time Schwarzschild black hole visualizer that ray-traces actual null
geodesics of general relativity per pixel, on the GPU. You can watch light
bend around the event horizon, see the photon ring, the lensed (warped)
image of the far side of the accretion disk above and below the shadow, and
the relativistic Doppler asymmetry that makes one side of the disk brighter
and bluer.

No libraries, no build step — a single `index.html` with a WebGL2 fragment
shader.

## Run it

Open `index.html` in any modern browser (WebGL2 required), or serve it:

```sh
python3 -m http.server
# then visit http://localhost:8000
```

**Controls:** drag to orbit the camera, scroll / pinch to zoom. The panel
lets you change disk temperature and size, exposure, quality, and toggle
each physical effect (lensing, Doppler beaming, gravitational redshift)
independently — handy for seeing what each one contributes. Switch the
background to "Checker grid" to make the lensing distortion explicit.

## The physics

Geometric units are used throughout: `G = c = 1`, lengths measured in
Schwarzschild radii (`r_s = 2GM/c² = 1`, so `M = 1/2`).

### Light bending — null geodesics

For each pixel a ray is launched from the camera and integrated backwards
through the Schwarzschild metric

```
ds² = −(1 − r_s/r) dt² + dr²/(1 − r_s/r) + r² dΩ²
```

Null geodesics in this metric obey the Binet equation (with `u = 1/r`):

```
d²u/dφ² + u = (3/2) r_s u²
```

The right-hand side is the GR correction — drop it and you get straight
lines. The shader integrates the equivalent 3D vector form

```
d²r/ds² = −(3/2) r_s h² r / |r|⁵ ,   h = |r × dr/ds| (conserved)
```

with a 4th-order Runge–Kutta scheme and adaptive step size (small steps
near the photon sphere where curvature is strongest). Rays that fall inside
`r = r_s` hit the horizon (black); rays that escape sample the background
along their final, deflected direction — which is what produces the Einstein
ring and the lensed starfield. Rays that get trapped winding around
`r = 1.5 r_s` form the bright thin **photon ring**.

### Accretion disk

The disk is a geometrically thin, optically thick equatorial disk between
the **ISCO** at `r = 6GM/c² = 3 r_s` (the innermost stable circular orbit,
below which gas plunges in) and an adjustable outer radius.

- **Temperature** follows the Shakura–Sunyaev thin-disk profile
  `T(r) ∝ r^(−3/4) · (1 − √(r_in/r))^(1/4)`, normalized so the slider sets
  the peak temperature.
- **Orbital motion** is exactly Keplerian in Schwarzschild:
  `Ω = √(GM/r³)`, and the speed measured by a local static observer is
  `v = √(M/(r − 2M))` — which correctly reaches `c` at the photon sphere.
- **Frequency shift**: the combined gravitational + Doppler factor for an
  emitter on a circular orbit is

  ```
  g = ν_obs/ν_emit = √(1 − r_s/r) / [γ (1 − β·n̂)]
  ```

  where `n̂` is the photon's propagation direction at emission.
- **Beaming for free**: because `I_ν/ν³` is Lorentz-invariant, a blackbody
  at temperature `T` observed with shift `g` is exactly a blackbody at
  `g·T`. The shader therefore just renders the Planck spectrum at
  `T_obs = g·T`, and the `g⁴` intensity boost (Doppler beaming on the
  approaching side, dimming on the receding side) falls out of `σT⁴`
  automatically.
- **Color** comes from Planck's law `B(λ,T) = (2hc²/λ⁵)/(e^(hc/λkT) − 1)`
  sampled at representative R/G/B wavelengths.

The turbulent banding is procedural fBm noise advected by the differential
rotation `Ω(r)`, so inner annuli visibly shear past outer ones.

### What to look for

- The **shadow** is noticeably larger than the horizon: its apparent radius
  is `√27/2 · r_s ≈ 2.6 r_s`, set by the critical impact parameter of the
  photon sphere.
- The far side of the disk appears **folded over and under** the shadow —
  light from behind the black hole is bent toward the camera.
- With Doppler beaming on, the side rotating toward you is **brighter and
  bluer**; turn it off and the disk becomes symmetric.
- Zoom in near `r ≈ 1.5 r_s` to see the photon ring built from light that
  orbited the hole one or more times.
