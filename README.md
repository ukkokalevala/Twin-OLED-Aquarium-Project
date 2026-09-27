TwinOLED Aquarium — Project Description
A Two-Screen Wireless Fish Tank
What It Is
TwinOLED Aquarium is a project that uses two ESP32 boards, each with its own small OLED display, to create one seamless virtual fish tank. The two screens sit side by side and act as a single 256×64 pixel canvas. Fish swim across the tank as if it were one continuous display, crossing invisibly from one screen to the other. A snail crawls along the sand, plants sway, and bubbles rise.

The two boards talk to each other wirelessly using ESP-NOW — a low-latency, peer-to-peer wireless link built into the ESP32. No router, no WiFi network, no internet. Just two chips talking directly.

The Core Idea
Picture two OLED screens sitting next to each other:

text
┌──────────────┐ ┌──────────────┐
│  🫧    🐟    │ │   🐟    🫧   │
│  🪸       🫧 │ │ 🫧       🪸  │
│  ▓▓▓▓▓▓▓▓▓▓▓ │ │ ▓▓▓▓▓▓▓▓▓▓▓ │
└──────────────┘ └──────────────┘
   ESP32 "A"        ESP32 "B"
   (left half)     (right half)
Together they form a virtual 256×64 tank. A fish swims off the right edge of Board A, and in the very same frame, it appears at the left edge of Board B — as if the two screens were always one.

To the viewer, it looks like one wide display. Under the hood, it's two independent boards coordinating in real time.

What Lives in the Tank
Two fish — each with its own speed, vertical bob, and tail animation. They swim left and right, occasionally turning around at the edges. One is slightly smaller than the other so the tank feels alive and varied.

A snail — crawling very slowly along the sand at the bottom. It moves almost imperceptibly, sometimes resting, always inching along.

Swaying plants — anchored to the sand, gently leaning back and forth with a slow sine wave.

Rising bubbles — small dots that drift upward at varying speeds, refreshing as they reach the surface.

A sand floor — a solid strip along the bottom of both screens that anchors the whole scene.

Nothing here needs a sensor. The whole tank is software-simulated, and the only thing that travels between boards is a tiny 32-byte packet per frame.

How the Two Boards Work Together
Board	Role	Responsibility
ESP32-A	Simulator + Left Renderer	Runs all the physics, owns every position, broadcasts every frame, draws the left half
ESP32-B	Right Renderer	Listens for the broadcast, redraws the same scene shifted 128 pixels, draws the right half
This simple split removes any ambiguity about who owns the scene — Board A drives, Board B mirrors. No handoff logic, no drift, no jitter.

The MAC Address Problem — and the Easy Fix
Normally, ESP-NOW is like mailing a letter: to send data from one board to another, you must know the receiver's exact MAC address (a unique hardware ID). Every ESP32 has a different one, so you would have to look it up, write it down, and hardcode it into your program. That's a hassle — and it breaks the moment you swap boards.

The Solution: Broadcast
Instead of mailing a letter to one specific address, we shout into the room. In ESP-NOW, this is called broadcast mode. The sender doesn't need to know anyone's MAC address — it just sends the packet out to everyone listening on the same channel.

In code, this is a one-line change: replace the target MAC with the broadcast address — six bytes, each one the hex value F F.

You can think of it as writing "EVERYONE" on the envelope instead of one person's name. Any board in range hears the message.

The whole MAC lookup problem disappears. No discovery code. No hardcoding. No pairing step.

Two Small Details That Make It Clean
1. The echo problem. When a board shouts, it also hears its own packet come back. Instead of fighting that, we simply decide that only one board listens. Board A talks and never registers a receive callback; Board B listens and never sends. No conflict, no echo issue.

2. Strangers in the room. In a space with other wireless devices, you might hear packets that aren't yours. To filter them out, every packet carries a small tag byte — a known "magic" number. If the tag doesn't match, the packet is dropped. Simple, reliable, no errors.

Hardware
Per board (×2):

1× ESP32 development board (any variant with WiFi)

1× SSD1306 OLED display (128×64, I²C)

4× jumper wires (VCC, GND, SDA, SCL)

USB power source

No sensors. No buttons. No MPU. No extra modules.

Wiring
OLED	ESP32
VCC	3.3V
GND	GND
SDA	GPIO 7
SCL	GPIO 6
Identical on both boards.

Software Stack
ESP-NOW — built into the ESP32 Arduino core, no library to install

Adafruit SSD1306 — for the OLED

Adafruit GFX — for shapes, lines, circles, and automatic clipping

A shared 32-byte packet carrying the state of both fish, the snail, and the bubble seed

Why the Seam Is Invisible
Both boards share the same virtual coordinate system. Board A covers x = 0 to 127, Board B covers x = 128 to 255. Each board draws the whole scene at the correct virtual position and lets the OLED library automatically clip anything outside its own 128-pixel window.

Result:

Objects at x = 0–127 appear on Board A only

Objects at x = 128–255 appear on Board B only

Objects outside 0–255 appear on neither

No manual split logic. Each board just draws the scene — the hardware quietly handles the rest.

The only rule is: Board B must subtract exactly 128 from every virtual x coordinate. Off by even one pixel and you get a visible stutter at the seam.

Bonus trick: a plant is deliberately placed right on the seam — half drawn by Board A, half drawn by Board B. When the two boards are aligned, it looks like one tall plant straddling the join, and it hides any 1-pixel mismatch between the screens. It's the single best way to make the two panels feel truly joined.

What Makes It Look Alive
Even though the scene is simple, a few small touches make it feel like a real tank:

Bob and sway — both fish drift up and down on a slow sine wave, plants lean side to side

Tail flap — the tail of each fish alternates every few frames

Different speeds — the two fish move at different rates and turn at different times

The snail's patience — it barely moves, and that's what makes it feel real

Bubble drift — bubbles rise at slightly different speeds and reappear at random heights

One fish is smaller — visual variety, and it makes the tank feel populated

None of these are complicated. Together they turn a static scene into something that feels alive.

What You Can Do With It
Desk ornament — a tiny, living aquarium on a bookshelf

Gift — a charming, quiet piece of electronics with no screen glare

Teaching tool — a friendly introduction to distributed rendering and wireless protocols

Base for more — add sound, buttons, sensors, or a third screen

Ambient display — it's calm enough to leave on all day

Because the base is just "state broadcast + two boards drawing halves," the whole tank can grow: more fish, a jellyfish, a crab, light rays, a castle, day/night cycles.

What It Demonstrates
ESP-NOW low-latency peer-to-peer wireless

Broadcast mode — no MAC lookup, no pairing, no discovery code

Distributed rendering — one logical canvas split across two devices

Master/slave coordination — one board drives, one mirrors

Automatic clipping as a design tool — no manual boundary math

A shared virtual coordinate space — the trick that makes the seam disappear

A reusable foundation for other multi-board projects: scrolling text, particle effects, games, dashboards

Tuning Options
Change	Effect
Fish speed	How fast each fish swims
Bob depth	How much they rise and fall
Snail tick rate	How often the snail moves
Plant height & count	Density of the underwater foliage
Bubble rate	How busy the water feels
Frame delay	Smoothness of the whole scene
Seam plant placement	How invisible the screen boundary becomes
The One-Line Description
TwinOLED Aquarium — two OLED displays acting as one continuous fish tank, with fish, a snail, plants, and bubbles, linked wirelessly by ESP-NOW with no MAC lookup, no pairing, and no visible seam.
