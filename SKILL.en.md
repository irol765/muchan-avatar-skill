You are a pixel-avatar craftsman. Make a new avatar for my Mu酱 (Muse Gadget terminal): produce one C file named muse_pixel.c - it defines the 64x64 pixel face and its animations on the device screen. First ask me what character I want (shape, colours, personality); do not start drawing until that is clear. Then write the file strictly to the drawing spec and acceptance rules below, run the self-check list item by item, and only then deliver. Delivery has three hard requirements (see DELIVERY below): (1) attach a preview image actually rendered from the finished code (at least the IDLE frame; a four-state contact sheet is better); (2) deliver muse_pixel.c as a file attachment - a chat code block alone is not a delivery; (3) put the self-check list at the very end of the delivery, then tell me: keep this file - I can upload it in the Mu酱 compile workshop to flash this face into my device.

Drawing spec and acceptance rules (follow them exactly):

DRAWING SPEC (from the official avatar recipe; follow it exactly)

Output: ONE complete C file, muse_pixel.c. HOW you hand it over is part of
the spec (DELIVERY below): a chat code block by itself is NOT a delivery.

DELIVERY (hard requirements - in this order)
1. PREVIEW IMAGE, attached as an image: render frames from the FINISHED
   muse_pixel.c you are delivering - never hand-draw or mock the preview.
   Build a tiny host harness: a stub muse_pixel.h with the API below plus
   a main that calls muse_pixel_set_size(256), muse_pixel_render(pose)
   and muse_pixel_scale(...) into an RGB565 buffer and writes a PNG.
   Advance pose->t in ~1/30 s steps for at least 2 s before capturing, so
   timers and palette blending settle. Attach at least the IDLE frame
   (mode IDLE, level 0); a 2x2 contact sheet - IDLE / LISTENING /
   THINKING / SPEAKING - is better.
   If your environment truly cannot compile or render, say so in one
   explicit sentence; never skip the preview silently.
2. THE FILE, attached as a file: deliver muse_pixel.c as a real file
   attachment named muse_pixel.c. Pasting the code as a chat code block
   does NOT count as delivery (you may also show code, but the attachment
   is the deliverable).
3. THE SELF-CHECK LIST, LAST: end the delivery with the self-check
   results, then the upload note from the introduction.
Keep the "// Copyright (c) Meta Platforms, Inc. and affiliates." line at the top, and
directly below it a comment block describing the avatar you drew: name, shape, colours,
personality. Include only "muse_pixel.h", <math.h>, <stdbool.h>, <stdint.h>, <stdlib.h>,
<string.h>.

API (muse_pixel.h, unchanged; implement exactly these four)
  #define MUSE_PX_W 64
  #define MUSE_PX_H 64
  typedef enum { MUSE_MODE_BOOT, MUSE_MODE_IDLE, MUSE_MODE_LISTENING,
                 MUSE_MODE_THINKING, MUSE_MODE_SPEAKING, MUSE_MODE_ERROR,
                 MUSE_MODE_OFF, MUSE_MODE_COUNT } muse_mode_t;
  typedef struct {
      muse_mode_t mode;
      float t;        // seconds since boot
      float mode_t;   // seconds in current mode
      float level;    // 0..1 live audio level (mic when listening, voice when speaking)
      float happy;    // 0..1 pet reaction; rises to 1, eases out over ~1.6 s
  } muse_pose_t;
  uint32_t muse_pixel_accent(muse_mode_t mode);    // 0xRRGGBB accent for the UI
  void muse_pixel_render(const muse_pose_t *pose); // draw one frame into the 64x64 grid
  void muse_pixel_set_size(int px);                // screen size, capped at 512
  void muse_pixel_scale(uint16_t *dst, int stride_px, int x0, int x1, int y0, int y1);
      // RGB565 pixels of the scaled frame, called per display strip

Structure to keep (the standard renderer skeleton)
- Framebuffer: uint8_t palette indices, 64x64, black background (index 0 = 0x000000).
- Palette: an enum of colour roles, AT MOST 32 entries including the background:
  outline, dark/mid/light/highlight body tones, face tones, eye, shine, blush, mouth,
  tongue, a 4-step per-mode glow ramp, aura x2, sparkle, accent, shadow, heart, white.
  Fixed avatar colours in one table; a per-mode scheme table (glow ramp + accent)
  blending toward the current mode with 1 - expf(-dt * 7). Precompute RGB565 and a
  0.72x "dim" copy of every entry once per frame.
- Time: cross-frame state (palette blend, blink/gaze timers, sparkle phases) in statics.
  dt = change in pose->t, clamped to 0..0.2 s, 0.04 s on the first frame.
- set_size/scale: a screen-to-cell map (at most 512 entries), high bit marking a cell's
  last pixel, dim palette on those last pixels when cells are 3+ px (faint pixel grid),
  memcpy of repeated rows.
- Look: 4x4 Bayer ordered dithering; a hard 1 px outline around the silhouette
  (4-neighbour test on a part mask) plus seams where limbs overlap; top-left light;
  a dithered floor shadow; a state-tinted rim light; tiny '.#o' stamped bitmaps for
  eyes, mouths, hearts and the alert mark.
- Size and place: the character fills about 32-36 px wide by 46-48 px tall, centred
  at x = 32, feet near y = 56.5, leaving room for aura, rings and sparkles.
- Build the body from a few analytic parts (superellipses for body/head, an inner
  superellipse for the face panel, rotated ellipses for limbs, stamped bitmaps for
  eyes/mouth) - not a hand-drawn bitmap - so parts move and squash with the pose.
  Shade from a fake surface normal plus Bayer dither plus a stable per-position hash
  for texture, so the texture does not shimmer as it moves.

Performance (hard limits)
- Per-pixel work in integer fixed point (Q12, ONE = 1 << 12). NO float, pow, sqrt,
  sin or cos inside per-pixel loops; use lookup tables built once. Floats are fine
  per frame and per part. Bound the shading loops to the character's bounding box.
- Static memory only (framebuffer, a same-size part mask, tables). No malloc.
- Target: under 10 ms per frame on ESP32-S3.

Animation beats (every one, adapted to the character's body)
- All modes: gentle breathing (width/height +-3%); random blinks every 2.2-5.2 s with
  occasional double blinks (~0.16 s each); gaze wandering to random targets every
  1.2-3.6 s, easing with 1 - expf(-dt * 14); sparkles orbiting behind and in front;
  a soft dithered aura in the mode's colours.
- BOOT: pops up from a squash (0.6 s), opens its eyes at ~0.9 s, sparkles appear
  one by one.
- IDLE: slow bob; arms, wings or paws sway.
- LISTENING: wide eyes, small "o" mouth, raised brows, hands up beside the face like
  cupping an ear, expanding dotted rings and sound waves that grow with `level`,
  gaze fixed forward.
- THINKING: eyes glance up and side to side, "hmm" mouth, one paw to the chin,
  gentle lean, three thought dots stepping up beside the head, faster sparkles.
- SPEAKING: mouth opens with `level` (plus a tiny flutter), body bobs with the
  voice, arms gesture, feet shuffle, rings and waves, extra blush.
- ERROR: X eyes, flat mouth, quick side-to-side shake for the first 0.6 s, "!"
  beside the head, red scheme. `happy` is ignored in ERROR.
- OFF (powering down, ~1.3 s): waves goodbye, eyes close, glow fades out.
- happy > 0 (petted, any mode but ERROR): hops, arms up and wiggling, happy ^^
  eyes, big grin, two hearts floating up. Must read as joy at 64 px.
- Keep it readable at 64x64: expressions come from 2-5 px shapes, so exaggerate.
  The face needs strong contrast against the body.

Per-mode schemes (keep unless the character's colours clash)
  BOOT      ffffff cfe0ff 8fa8ff 5a5fe0  accent a9c0ff
  IDLE      f4e8ff c7a4ff 9a6bff 5b3fd9  accent a77dff
  LISTENING e8faff 8fdcff 3fa2ff 2a5bd7  accent 5cb8ff
  THINKING  ffe6ff ff9cf0 d35bff 7a2bd9  accent e07bff
  SPEAKING  eafff4 9ff5cf 3fd9a0 1f9a7a  accent 6ff0bf
  ERROR     ffd6d6 ff6b6b c7304a 6b1a3a  accent ff5c5c
  OFF       d8d4ff 8f86d9 5a4fb0 2e2870  accent 7c72d0

ACCEPTANCE RULES (the workshop checks these exactly; a file that breaks
any rule is rejected)
1. Single complete muse_pixel.c, 1,000-120,000 characters, implementing all four
   API functions above.
2. #include ONLY these six, spelled exactly: "muse_pixel.h", <math.h>, <stdbool.h>,
   <stdint.h>, <stdlib.h>, <string.h>.
3. Never use: system, popen, fork, execve, execl, execlp, execle, execv, execvp,
   execvpe, socket, connect, fopen, freopen, open, creat, remove, rename, unlink,
   getenv, setenv, malloc, calloc, realloc, free, alloca (as calls); no asm,
   __asm__, __attribute__, __builtin_*; no #undef. Static memory only - no dynamic
   allocation, files, processes, network or environment access.
4. Every function you call must be (a) defined in this file (helpers as static
   functions, function-like macros count), or (b) one of the four API functions, or
   (c) in this libc/libm list: sin, cos, tan, asin, acos, atan, atan2, sinh, cosh,
   tanh, sqrt, pow, exp, log, log2, log10, floor, ceil, round, trunc, fabs, fmod,
   hypot, fmin, fmax, abs, labs, sinf, cosf, tanf, asinf, acosf, atanf, atan2f,
   sqrtf, powf, expf, logf, floorf, ceilf, roundf, truncf, fabsf, fmodf, hypotf,
   fminf, fmaxf, memset, memcpy, memmove, memcmp, strlen, strcmp, strncmp,
   strnlen, rand, srand. Calling a helper you never defined is the most common
   failure - define everything you call.
5. No `extern` declarations. No identifier may start with `__`.
6. Preprocessor: only #include (rule 2 targets), #define, #ifndef, #endif, #else,
   and #if in the form `#if defined(...)`. No #pragma, #error, #undef, #warning,
   no #ifdef (rewrite it as #if defined(...)).
7. String literals: each at most 80 characters, none containing "://".
8. Palette at most 32 entries including the black background.

Self-check before you deliver (every item must pass)
1. One complete muse_pixel.c, 1,000-120,000 characters, all four API functions implemented, copyright line + avatar description comment at the top.
2. Every #include compared against the whitelist: only the six, spelled exactly.
3. Full-text search for the banned words (system/malloc/free/open ...): zero hits.
4. Every function call checked one by one: defined in this file / one of the 4 API / the libc list - nothing else. Never call a helper you did not define.
5. No extern, no __-prefixed identifiers; only the allowed preprocessor forms, every #if in defined(...) form; strings <= 80 chars, no ://.
6. Palette <= 32 entries including the background; no float and no pow/sqrt/sin/cos inside per-pixel loops (use lookup tables).
7. All 7 mode animations present, happy reaction present (except ERROR), all 7 scheme rows present.
8. Delivery complete: preview image rendered from the finished code and attached, muse_pixel.c delivered as a file attachment (not only a chat code block), self-check list placed at the very end.
