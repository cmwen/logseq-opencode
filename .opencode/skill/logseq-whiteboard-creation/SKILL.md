---
name: logseq-whiteboard-creation
description: Create and manage whiteboards in LogSeq graph using EDN format
license: MIT
compatibility: opencode
metadata:
  audience: knowledge-workers
  workflow: logseq-graph
---

# LogSeq Whiteboard Creation Skill

This skill teaches LLM agents how to create and manage whiteboards in LogSeq graph folders using the EDN (Extensible Data Notation) format.

## What I Do

- Create new whiteboard EDN files from scratch
- Add shapes (text, ellipse, box, line, polygon, etc.) to whiteboards
- Generate proper EDN syntax for LogSeq whiteboard storage
- Manage UUIDs, timestamps, and shape indexing
- Integrate whiteboards into the LogSeq graph structure

## When to Use Me

Use this skill when you need to:
- Programmatically generate whiteboard files for the LogSeq graph
- Create diagram templates or shape collections as EDN
- Automate whiteboard creation as part of knowledge management workflows
- Ensure proper LogSeq whiteboard format and compatibility

---

## EDN Format Basics for LogSeq Whiteboards

### File Structure

LogSeq whiteboards are stored as `.edn` files in the `whiteboards/` directory. The root structure contains two main keys:

```clojure
{:blocks (shape1 shape2 shape3 ...)   ; All shapes in the whiteboard
 :pages (page1 page2 page3 ...)}      ; Whiteboard page metadata
```

### Core Concepts

**Clojure EDN Syntax:**
- Maps: `{:key value :key2 value2}` (key-value pairs)
- Vectors: `[1 2 3]` (ordered arrays)
- Keywords: `:keyword` (prefixed with colon)
- UUIDs: `#uuid "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"`
- Comments: `;` prefix

---

## Part 1: Creating a New Whiteboard File

### Step 1: Create the EDN File

Create a new file in the `whiteboards/` directory with a meaningful name:

```
whiteboards/my-diagram.edn
```

### Step 2: Define the Basic Structure

Start with an empty whiteboard structure:

```clojure
{:blocks ()
 :pages ({:block/uuid #uuid "YOUR-UUID-HERE"
          :block/properties
          {:ls-type :whiteboard-page
           :logseq.tldraw.page
           {:id "YOUR-UUID-HERE"
            :name "YOUR-WHITEBOARD-NAME"
            :bindings {}
            :nonce 1
            :assets []
            :shapes-index []}}
          :block/updated-at TIMESTAMP
          :block/created-at TIMESTAMP
          :block/type "whiteboard"
          :block/name "YOUR-WHITEBOARD-NAME"
          :block/original-name "YOUR-WHITEBOARD-NAME"})}
```

### Step 3: Generate Required Values

**UUID Generation:**
- Use any UUID v4 generator
- Ensure consistency: the same UUID appears in `:block/uuid`, `:id`, and shape `:parentId`
- Format: `#uuid "550e8400-e29b-41d4-a716-446655440000"`

**Timestamp Generation:**
- Use milliseconds since Unix epoch (January 1, 1970)
- Example: `1768611047550` (January 17, 2025)
- Both `:block/created-at` and `:block/updated-at` should match initially

### Step 4: Reference the Page UUID

The page UUID must match everywhere it appears:
- `:block/uuid` - page identifier
- `:id` inside `:logseq.tldraw.page` - internal reference
- `:parentId` in each shape - links shapes to their parent page

---

## Part 2: Adding Shapes to Whiteboard

### Shape Block Structure

Each shape is a block with standard LogSeq properties plus tldraw-specific geometry:

```clojure
{:block/created-at TIMESTAMP
 :block/properties
 {:ls-type :whiteboard-shape
  :logseq.tldraw.shape
  {:id "SHAPE-UUID"                 ; unique identifier for this shape
   :type "SHAPE-TYPE"               ; shape type: "text", "box", "ellipse", "line", "polygon", "pencil", "highlighter", "logseq-portal"
   :point [X Y]                     ; [left, top] position in pixels
   :size [WIDTH HEIGHT]             ; [width, height] in pixels
   :parentId "PAGE-UUID"            ; links to parent page
   :index INDEX                     ; z-order (0, 1, 2...) - higher numbers appear on top
   :scale [1 1]                     ; scale multiplier [x y]
   :scaleLevel "md"                 ; scale level: "sm", "md", "lg", "xl", "xxl"
   :opacity 1                       ; 0.0 to 1.0 (transparency)
   :nonce TIMESTAMP                 ; creation time in milliseconds
   
   ; Stroke/Border Properties
   :stroke ""                       ; stroke color: "" (none), "blue", "green", "red", "purple", or hex "#333333"
   :strokeWidth 2                   ; border thickness in pixels (affected by scaleLevel)
   :strokeType "line"               ; "line", "dotted", "dashed"
   
   ; Fill Properties
   :noFill true                     ; true = transparent, false = filled
   :fill ""                         ; fill color: "" (none), "blue", "green", "red", "purple", or hex "#e3f2fd"
   
   ; Text Properties (for text shapes)
   :text "Shape Text"               ; the displayed text content
   :fontSize 20                     ; font size in pixels (affected by scaleLevel)
   :fontFamily "var(--ls-font-family)"  ; font family (use this default)
   :fontWeight 400                  ; 400 (normal), 700 (bold)
   :italic false                    ; true or false
   :lineHeight 1.2                  ; line spacing multiplier
   :padding 4                       ; internal text padding
   :isSizeLocked true               ; prevent resizing (text shapes)
   
   ; Labels (for shapes like ellipse, box)
   :label "Shape Label"             ; optional label/text displayed on shape
   
   ; Geometry Properties (shape-specific)
   :borderRadius 2                  ; corner radius for box shapes (0-8+ pixels)
   
   ; References (optional)
   :refs []                         ; array of page/tag references ["TODO", "cheatsheet"]
   
   ; Rotation (optional)
   :rotation 0}}                    ; rotation in degrees
 :block/updated-at TIMESTAMP}
```

### Scale Levels and Their Effects

The `:scaleLevel` property affects visual rendering. Common values:

| Scale Level | Stroke Width Multiplier | Font Size Multiplier | Use Case |
|-------------|-------------------------|----------------------|----------|
| `"sm"` | 0.8x (1.6px for strokeWidth 2) | 0.5x (16px for fontSize 32) | Small annotations |
| `"md"` | 1.0x (2px for strokeWidth 2) | 1.0x (32px for fontSize 32) | Default shapes |
| `"lg"` | 1.6x (3.2px for strokeWidth 2) | 1.0x (32px for fontSize 32) | Emphasized shapes |
| `"xl"` | 2.4x | 1.5x | Large diagrams |
| `"xxl"` | 3.0x (6px for strokeWidth 2) | 1.875x (60px for fontSize 32) | Titles, headers |

**Important:** strokeWidth and fontSize in the shape definition are BASE values. The actual rendered size = base × scaleLevel multiplier.

### Shape Types Reference

LogSeq uses tldraw shape types. All supported types:

| Type | Purpose | Key Properties | Notes |
|------|---------|----------------|-------|
| `text` | Text boxes | `:text`, `:fontSize`, `:fontWeight`, `:italic`, `:isSizeLocked` | Use for labels and annotations |
| `box` | Rectangular boxes | `:label`, `:borderRadius`, `:fill`, `:noFill` | Default borderRadius is 2 |
| `ellipse` | Circles and ovals | `:label`, `:fill`, `:noFill` | Use equal width/height for circles |
| `line` | Connector lines with arrows | `:handles`, `:decorations` | Requires binding configuration |
| `polygon` | Multi-sided shapes | `:sides`, `:ratio`, `:isFlippedY` | Use `:sides 3` for triangle |
| `pencil` | Freehand drawings | `:points`, `:isComplete` | Array of [x, y, pressure] points |
| `highlighter` | Highlight overlays | `:points`, `:isComplete`, `:opacity 0.5` | Use opacity 0.5 for transparency |
| `logseq-portal` | Embedded page blocks | `:pageId`, `:blockType`, `:collapsed` | Links to LogSeq pages |

---

### Line Shapes with Connectors

Line shapes in LogSeq whiteboards support sophisticated connections between shapes using **handles** and **bindings**. This allows arrows to snap to shapes and maintain connections when shapes are moved.

#### Basic Line Structure

```clojure
{:block/created-at TIMESTAMP
 :block/properties
 {:ls-type :whiteboard-shape
  :logseq.tldraw.shape
  {:id "LINE-UUID"
   :type "line"
   :point [X Y]                     ; starting position
   :strokeType "line"               ; "line", "dotted", "dashed"
   :strokeWidth 1                   ; line thickness
   :opacity 1
   :label ""                        ; empty for connectors
   :fontWeight 400
   :noFill true
   :fontSize 20
   :parentId "PAGE-UUID"
   :index INDEX
   :nonce TIMESTAMP
   :italic false
   
   ; Handles define connection points
   :handles
   {:start
    {:id "start"
     :canBind true                  ; allows binding to shapes
     :point [X Y]                   ; relative offset from shape :point
     :bindingId "BINDING-UUID"}     ; links to binding in page
    :end
    {:id "end"
     :canBind true
     :point [X Y]
     :bindingId "BINDING-UUID"}}
   
   ; Decorations add arrowheads
   :decorations
   {:start "arrow"                  ; arrowhead at start (optional)
    :end "arrow"}}}                 ; arrowhead at end (optional)
 :block/updated-at TIMESTAMP}
```

#### Page Bindings Configuration

Lines that connect to shapes require **bindings** in the `:logseq.tldraw.page` section. Each binding links a line's handle to a target shape:

```clojure
:pages (
 {:block/uuid #uuid "PAGE-UUID"
  :block/properties
  {:ls-type :whiteboard-page
   :logseq.tldraw.page
   {:id "PAGE-UUID"
    :name "my-whiteboard"
    :bindings
    {; Binding for line start handle
     :BINDING-START-UUID
     {:id "BINDING-START-UUID"
      :type "line"                  ; binding type
      :fromId "LINE-UUID"           ; the line shape ID
      :toId "SHAPE-UUID"            ; target shape to connect to
      :handleId "start"             ; which handle: "start" or "end"
      :point [0.5 0.5]              ; attachment point on target (0-1, center = 0.5)
      :distance 4}                  ; snap distance
     
     ; Binding for line end handle
     :BINDING-END-UUID
     {:id "BINDING-END-UUID"
      :type "line"
      :fromId "LINE-UUID"
      :toId "OTHER-SHAPE-UUID"
      :handleId "end"
      :point [0.5 0.5]
      :distance 4}}
    :shapes-index [...]}}})
```

#### Complete Line Example with Bindings

Here's a complete example of a line connecting two boxes:

**Line Shape:**
```clojure
{:block/created-at 1768621811641
 :block/properties
 {:ls-type :whiteboard-shape
  :logseq.tldraw.shape
  {:id "line-001"
   :type "line"
   :point [799.66 676]
   :strokeType "line"
   :strokeWidth 1
   :opacity 1
   :label ""
   :fontWeight 700
   :noFill true
   :fontSize 20
   :parentId "page-uuid"
   :index 3
   :nonce 1768621811641
   :italic false
   :handles
   {:start
    {:id "start"
     :canBind true
     :point [24.68 0]               ; offset from line :point
     :bindingId "binding-start-001"}
    :end
    {:id "end"
     :canBind true
     :point [0 144]                 ; offset from line :point
     :bindingId "binding-end-001"}}
   :decorations
   {:start "arrow"                  ; arrowhead at both ends
    :end "arrow"}}}
 :block/updated-at 1768621811641}
```

**Page Bindings:**
```clojure
:bindings
{:binding-start-001
 {:id "binding-start-001"
  :type "line"
  :fromId "line-001"                ; the line
  :toId "box-001"                   ; connects to first box
  :handleId "start"
  :point [0.5 0.5]                  ; center of box
  :distance 4}
 :binding-end-001
 {:id "binding-end-001"
  :type "line"
  :fromId "line-001"
  :toId "box-002"                   ; connects to second box
  :handleId "end"
  :point [0.5 0.5]
  :distance 4}}
```

#### Line Decoration Options

Control arrowhead appearance with `:decorations`:

| Configuration | Visual Result |
|---------------|---------------|
| `{:end "arrow"}` | Arrow at end only → |
| `{:start "arrow"}` | ← Arrow at start only |
| `{:start "arrow" :end "arrow"}` | ↔ Arrows at both ends |
| `{}` or omit | Plain line (no arrows) |

#### Tips for Creating Connected Lines

1. **Generate UUIDs**: Create unique IDs for line shape, start binding, and end binding
2. **Position line roughly**: Set `:point` near the midpoint between shapes
3. **Calculate handle offsets**: `:handles :start :point` and `:end :point` are relative to line `:point`
4. **Create bindings**: Add both bindings to page `:bindings` map with matching IDs
5. **Reference target shapes**: Use actual shape UUIDs in `:toId`
6. **Set attachment points**: Use `[0.5 0.5]` for center, `[0 0.5]` for left-center, `[1 0.5]` for right-center, etc.

#### CRITICAL: Binding Lines to the Correct Shapes

**Common Mistake:** When creating flow diagrams with multiple boxes representing steps, developers often try to bind connector lines to the wrong shapes.

**Example Problem:**
```clojure
; You have these shapes:
; - "actor-box-1" at [80 160] (top left - represents an actor/entity)
; - "step-box-1" at [80 320] (middle - represents a step in the flow)
; - "step-box-2" at [480 320] (middle right - next step)
; - "arrow-1" connecting step-box-1 to step-box-2

; ❌ WRONG: Binding arrow to actor boxes instead of step boxes
:bindings {
  :binding-arrow-1-start
  {:fromId "arrow-1"
   :toId "actor-box-1"    ; WRONG - arrow is near step-box-1, not actor-box-1
   :handleId "start"}}

; ✅ CORRECT: Binding arrow to the actual step boxes it connects
:bindings {
  :binding-arrow-1-start
  {:fromId "arrow-1"
   :toId "step-box-1"     ; CORRECT - arrow actually connects these step boxes
   :handleId "start"}
  :binding-arrow-1-end
  {:fromId "arrow-1"
   :toId "step-box-2"
   :handleId "end"}}
```

**Rule of Thumb:**
- Look at the arrow's `:point` position (e.g., `[320 360]`)
- Check which shapes are physically near that position
- Bind the arrow to those nearby shapes, NOT to shapes that are far away
- If an arrow is between two boxes at Y=320-400, bind it to those boxes, not to boxes at Y=160

**Visual Example:**
```
Actor Box (Y=160)         Actor Box (Y=160)
     ↓                          ↓
Step Box 1 (Y=320) ----→ Step Box 2 (Y=320)
                   ↑
              Arrow should bind to Step Boxes,
              NOT Actor Boxes!
```

---

### Polygon Shapes

Polygons support multi-sided shapes like triangles, pentagons, hexagons:

```clojure
{:block/created-at TIMESTAMP
 :block/properties
 {:ls-type :whiteboard-shape
  :logseq.tldraw.shape
  {:id "POLYGON-UUID"
   :type "polygon"
   :point [X Y]
   :size [WIDTH HEIGHT]
   :sides 3                         ; number of sides (3=triangle, 5=pentagon, 6=hexagon, etc.)
   :ratio 1                         ; aspect ratio
   :isFlippedY false                ; flip vertically
   :stroke "green"
   :strokeWidth 6
   :strokeType "dashed"
   :fill "green"
   :noFill true
   :opacity 1
   :label "Triangle"
   :fontSize 60
   :fontWeight 700
   :italic false
   :scaleLevel "xxl"
   :parentId "PAGE-UUID"
   :index INDEX
   :nonce TIMESTAMP}}
 :block/updated-at TIMESTAMP}
```

**Common Polygon Sides:**
- `:sides 3` - Triangle
- `:sides 4` - Square/Diamond
- `:sides 5` - Pentagon
- `:sides 6` - Hexagon
- `:sides 8` - Octagon

---

### Pencil and Highlighter Shapes

For freehand drawing and highlighting:

**Pencil (Freehand Drawing):**
```clojure
{:type "pencil"
 :points [[0 57.01875 0.5] [1.606 57.553 0.5] ...]  ; [x, y, pressure] array
 :isComplete true
 :stroke ""
 :strokeWidth 2
 :opacity 1
 :noFill true}
```

**Highlighter (Overlay Highlight):**
```clojure
{:type "highlighter"
 :points [[0 0 0.5] [130.46 16.95 0.5] ...]
 :isComplete true
 :stroke "green"
 :strokeWidth 6
 :opacity 0.5                       ; IMPORTANT: use 0.5 for transparency
 :noFill true}
```

**Note:** Pencil and highlighter shapes use `:points` arrays with [x, y, pressure] values. The third value (pressure) is typically 0.5 for consistent line width.

---

### LogSeq Portal Shapes

Embed LogSeq page content directly in whiteboards:

```clojure
{:type "logseq-portal"
 :pageId "696a2938-dfed-4caf-84e3-fc363a316477"  ; UUID of LogSeq page
 :blockType "B"                     ; block type
 :collapsed false                   ; expand/collapse state
 :collapsedHeight 0
 :compact true
 :isAutoResizing true
 :borderRadius 8
 :fill "green"
 :noFill false
 :stroke "green"
 :strokeWidth 2
 :size [400 161]
 :scaleLevel "md"}
```

### Arrow/Connector Shapes

Arrow shapes are used to show flow, relationships, and directional connections between shapes. Key properties:

```clojure
{:block/created-at TIMESTAMP
 :block/properties
 {:ls-type :whiteboard-shape
  :logseq.tldraw.shape
  {:id "ARROW-UUID"
   :type "arrow"                      ; connector shape
   :point [X Y]                       ; starting point [left, top]
   :size [WIDTH HEIGHT]               ; size of connector bounds (can be [0 0])
   :stroke "#333333"                  ; arrow color (hex)
   :strokeWidth 2                     ; line thickness in pixels
   :strokeType "line"                 ; "line", "dotted", "dashed"
   :parentId "PAGE-UUID"              ; links to parent page
   :index INDEX                       ; z-order
   :opacity 1                         ; transparency 0.0-1.0
   :nonce TIMESTAMP}}                 ; creation time
 :block/updated-at TIMESTAMP}
```

**Arrow Positioning Tips:**
- `:point` is the starting coordinate of the arrow
- Arrows in tldraw don't have explicit endpoints; position them visually between shapes
- Use `:strokeWidth` 2-3 for visibility
- Use darker colors (#333333, #666666) for flow arrows

**Arrow Examples by Use Case:**

1. **Sequential Flow Arrow** (between steps):
   ```clojure
   {:point [450 270]  ; positioned between Step 1 and Step 2
    :size [0 0]      ; minimal bounds
    :stroke "#333333"
    :strokeWidth 2}
   ```

2. **Feedback/Loop Arrow** (curved flow back):
   ```clojure
   {:point [600 300]  ; positioned for return path
    :size [0 0]
    :stroke "#666666" ; darker for emphasis
    :strokeWidth 2}
   ```

3. **Conditional Arrow** (with different styling):
   ```clojure
   {:point [500 400]
    :size [0 0]
    :stroke "#ff9800" ; orange for "then" path
    :strokeWidth 2
    :strokeType "dashed"}  ; dashed for optional/alternative flow
   ```

---

## Part 3: Complete Examples - Common Shape Patterns

### Example 1: Transparent Box (Border Only)

```clojure
{:block/created-at 1768621679114
 :block/properties
 {:ls-type :whiteboard-shape
  :logseq.tldraw.shape
  {:id "box-transparent-001"
   :type "box"
   :label "Transparent Rectangle"
   :point [640 544]
   :size [392 128]
   :borderRadius 2
   :stroke ""                        ; empty = default black border
   :strokeWidth 3.2                  ; actual width = 3.2 with scaleLevel lg
   :strokeType "line"
   :fill ""                          ; empty = no fill color
   :noFill true                      ; transparent interior
   :opacity 1
   :fontSize 32
   :fontWeight 400
   :italic false
   :scale [1 1]
   :scaleLevel "lg"                  ; large scale multiplier
   :parentId "page-uuid"
   :index 1
   :nonce 1768621679114}}
 :block/updated-at 1768621679114}
```

### Example 2: Filled Colored Box

```clojure
{:block/created-at 1768621811640
 :block/properties
 {:ls-type :whiteboard-shape
  :logseq.tldraw.shape
  {:id "box-filled-001"
   :type "box"
   :label "Colored Rectangle"
   :point [592 824]
   :size [392 128]
   :borderRadius 2
   :stroke "blue"                    ; named color
   :strokeWidth 3.2
   :strokeType "line"
   :fill "blue"                      ; match stroke color
   :noFill false                     ; filled interior
   :opacity 1
   :fontSize 32
   :fontWeight 400
   :italic false
   :refs []                          ; can reference tags/pages
   :scale [1 1]
   :scaleLevel "lg"
   :parentId "page-uuid"
   :index 2
   :nonce 1768621811640}}
 :block/updated-at 1768621811640}
```

### Example 3: Dashed Rectangle (Low Opacity)

```clojure
{:block/created-at 1768621993112
 :block/properties
 {:ls-type :whiteboard-shape
  :logseq.tldraw.shape
  {:id "box-dashed-001"
   :type "box"
   :label "Dashed Rectangle"
   :point [528 1032]
   :size [392 128]
   :borderRadius 2
   :stroke ""
   :strokeWidth 3.2
   :strokeType "dashed"              ; dashed border
   :fill ""
   :noFill false
   :opacity 0.6                      ; 60% opacity for subtle effect
   :fontSize 32
   :fontWeight 400
   :italic false
   :refs []
   :scale [1 1]
   :scaleLevel "lg"
   :parentId "page-uuid"
   :index 10
   :nonce 1768621993112}}
 :block/updated-at 1768621993112}
```

### Example 4: Filled Circle (Ellipse)

```clojure
{:block/created-at 1768621886060
 :block/properties
 {:ls-type :whiteboard-shape
  :logseq.tldraw.shape
  {:id "circle-001"
   :type "ellipse"
   :label "Circle"
   :point [1144 552]
   :size [384 128]                   ; unequal dimensions = oval
   :stroke "green"
   :strokeWidth 3.2
   :strokeType "line"
   :fill "green"
   :noFill false
   :opacity 1
   :fontSize 32
   :fontWeight 700                   ; bold label
   :italic false
   :refs ["TODO"]                    ; tag reference
   :scale [1 1]
   :scaleLevel "lg"
   :parentId "page-uuid"
   :index 4
   :nonce 1768621886060}}
 :block/updated-at 1768621886060}
```

### Example 5: Transparent Circle with Italic Label

```clojure
{:block/created-at 1768621866247
 :block/properties
 {:ls-type :whiteboard-shape
  :logseq.tldraw.shape
  {:id "circle-transparent-001"
   :type "ellipse"
   :label "Transparent Circle"
   :point [1136 848]
   :size [384 128]
   :stroke "green"
   :strokeWidth 1.6                  ; actual width with scaleLevel sm
   :strokeType "line"
   :fill "green"
   :noFill true                      ; transparent
   :opacity 1
   :fontSize 16                      ; smaller font with scaleLevel sm
   :fontWeight 400
   :italic true                      ; italic label
   :refs []
   :scale [1 1]
   :scaleLevel "sm"                  ; small scale
   :parentId "page-uuid"
   :index 5
   :nonce 1768621866247}}
 :block/updated-at 1768621866247}
```

### Example 6: Dashed Circle (Purple)

```clojure
{:block/created-at 1768621921302
 :block/properties
 {:ls-type :whiteboard-shape
  :logseq.tldraw.shape
  {:id "circle-dashed-001"
   :type "ellipse"
   :label "Dashed Circle"
   :point [1628 552]
   :size [384 128]
   :stroke "purple"
   :strokeWidth 3.2
   :strokeType "dashed"              ; dashed border
   :fill "purple"
   :noFill false
   :opacity 1
   :fontSize 32
   :fontWeight 400
   :italic false
   :refs []
   :scale [1 1]
   :scaleLevel "lg"
   :parentId "page-uuid"
   :index 7
   :nonce 1768621921302}}
 :block/updated-at 1768621921302}
```

### Example 7: Triangle (Polygon with 3 Sides)

```clojure
{:block/created-at 1768621953661
 :block/properties
 {:ls-type :whiteboard-shape
  :logseq.tldraw.shape
  {:id "triangle-001"
   :type "polygon"
   :label "Triangle"
   :point [1688 816]
   :size [256 184]
   :sides 3                          ; triangle
   :ratio 1
   :isFlippedY false
   :stroke "green"
   :strokeWidth 6                    ; actual width with scaleLevel xxl
   :strokeType "dashed"
   :fill "green"
   :noFill true
   :opacity 1
   :fontSize 60                      ; large font with scaleLevel xxl
   :fontWeight 700
   :italic false
   :refs []
   :scale [1 1]
   :scaleLevel "xxl"                 ; extra large
   :parentId "page-uuid"
   :index 9
   :nonce 1768621953661}}
 :block/updated-at 1768621953661}
```

### Example 8: Text Shape with Large Font

```clojure
{:block/created-at 1768622000065
 :block/properties
 {:ls-type :whiteboard-shape
  :logseq.tldraw.shape
  {:id "text-001"
   :type "text"
   :text "Text"
   :point [1208 1096]
   :size [134 82]
   :isSizeLocked true                ; prevent resizing
   :borderRadius 0
   :stroke "green"
   :strokeWidth 2
   :strokeType "line"
   :fill "green"
   :noFill true
   :opacity 1
   :fontSize 60                      ; base font size
   :fontFamily "var(--ls-font-family)"
   :fontWeight 400
   :italic false
   :lineHeight 1.2
   :padding 4
   :scale [1 1]
   :scaleLevel "xxl"                 ; xxl multiplier applied
   :parentId "page-uuid"
   :index 12
   :nonce 1768622000065}}
 :block/updated-at 1768622000065}
```

### Example 9: Line with Bidirectional Arrows

```clojure
{:block/created-at 1768621811641
 :block/properties
 {:ls-type :whiteboard-shape
  :logseq.tldraw.shape
  {:id "line-001"
   :type "line"
   :point [799.66 676]
   :strokeType "line"
   :strokeWidth 1
   :stroke ""
   :fill ""
   :noFill true
   :opacity 1
   :label ""
   :fontSize 20
   :fontWeight 700
   :italic false
   :refs ["cheatsheet"]              ; optional reference
   :parentId "page-uuid"
   :index 3
   :nonce 1768621811641
   :handles
   {:start
    {:id "start"
     :canBind true
     :point [24.68 0]
     :bindingId "binding-start-001"}
    :end
    {:id "end"
     :canBind true
     :point [0 144]
     :bindingId "binding-end-001"}}
   :decorations
   {:start "arrow"                   ; arrow at both ends
    :end "arrow"}}}
 :block/updated-at 1768621811641}
```

### Example 10: Complete Whiteboard with Page and Bindings

Here's a minimal complete whiteboard showing two boxes connected by a line:

```clojure
{:blocks (
  ; First box
  {:block/created-at 1768621679114
   :block/properties
   {:ls-type :whiteboard-shape
    :logseq.tldraw.shape
    {:id "box-001"
     :type "box"
     :label "Box A"
     :point [640 544]
     :size [392 128]
     :borderRadius 2
     :stroke ""
     :strokeWidth 3.2
     :strokeType "line"
     :fill ""
     :noFill true
     :opacity 1
     :fontSize 32
     :fontWeight 400
     :italic false
     :scale [1 1]
     :scaleLevel "lg"
     :parentId "page-uuid-001"
     :index 0
     :nonce 1768621679114}}
   :block/updated-at 1768621679114}
  
  ; Second box
  {:block/created-at 1768621811640
   :block/properties
   {:ls-type :whiteboard-shape
    :logseq.tldraw.shape
    {:id "box-002"
     :type "box"
     :label "Box B"
     :point [592 824]
     :size [392 128]
     :borderRadius 2
     :stroke "blue"
     :strokeWidth 3.2
     :strokeType "line"
     :fill "blue"
     :noFill false
     :opacity 1
     :fontSize 32
     :fontWeight 400
     :italic false
     :refs []
     :scale [1 1]
     :scaleLevel "lg"
     :parentId "page-uuid-001"
     :index 1
     :nonce 1768621811640}}
   :block/updated-at 1768621811640}
  
  ; Connecting line
  {:block/created-at 1768621811641
   :block/properties
   {:ls-type :whiteboard-shape
    :logseq.tldraw.shape
    {:id "line-001"
     :type "line"
     :point [800 676]
     :strokeType "line"
     :strokeWidth 1
     :stroke ""
     :fill ""
     :noFill true
     :opacity 1
     :label ""
     :fontSize 20
     :fontWeight 700
     :italic false
     :refs []
     :parentId "page-uuid-001"
     :index 2
     :nonce 1768621811641
     :handles
     {:start
      {:id "start"
       :canBind true
       :point [24 0]
       :bindingId "binding-start-001"}
      :end
      {:id "end"
       :canBind true
       :point [0 144]
       :bindingId "binding-end-001"}}
     :decorations
     {:start "arrow"
      :end "arrow"}}}
   :block/updated-at 1768621811641})
 
 :pages (
  {:block/uuid #uuid "page-uuid-001"
   :block/properties
   {:ls-type :whiteboard-page
    :logseq.tldraw.page
    {:id "page-uuid-001"
     :name "my-connected-diagram"
     :bindings
     {; Line connects box-001 at start
      :binding-start-001
      {:id "binding-start-001"
       :type "line"
       :fromId "line-001"
       :toId "box-001"
       :handleId "start"
       :point [0.5 0.5]              ; center attachment
       :distance 4}
      ; Line connects box-002 at end
      :binding-end-001
      {:id "binding-end-001"
       :type "line"
       :fromId "line-001"
       :toId "box-002"
       :handleId "end"
       :point [0.5 0.5]
       :distance 4}}
     :nonce 1
     :assets []
     :shapes-index ["box-001" "box-002" "line-001"]}}
   :block/updated-at 1768621811641
   :block/created-at 1768621679114
   :block/type "whiteboard"
   :block/name "my-connected-diagram"
   :block/original-name "my-connected-diagram"})}
```
      :size [0 0]
      :strokeType "line"
      :strokeWidth 2
      :opacity 1
      :id "shape-004"
      :point [350 260]
      :parentId "page-001-uuid"
      :nonce 1768611080000}}
    :block/updated-at 1768611080000})
 
 :pages (
  {:block/uuid #uuid "550e8400-e29b-41d4-a716-446655440000"
   :block/properties
   {:ls-type :whiteboard-page
    :logseq.tldraw.page
    {:id "page-001-uuid"
     :name "page-001-uuid"
     :bindings {}
     :nonce 1
     :assets []
     :shapes-index ["shape-001" "shape-002" "shape-003" "shape-004"]}}
   :block/updated-at 1768611080000
   :block/created-at 1768611047550
   :block/type "whiteboard"
   :block/name "my-diagram"
   :block/original-name "my-diagram"})}
```

---

## Part 4: Color Reference

Common color hex values for whiteboard shapes:

| Color | Hex | Usage |
|-------|-----|-------|
| Red | `#f44336` | Warnings, errors, critical paths |
| Blue | `#2196f3` | Primary concepts, information |
| Green | `#4caf50` | Success, completed tasks |
| Yellow | `#ffc107` | Caution, attention needed |
| Orange | `#ff9800` | Secondary emphasis |
| Purple | `#9c27b0` | Special topics, ideas |
| Gray | `#9e9e9e` | Neutral, disabled states |
| Light Blue | `#e3f2fd` | Soft backgrounds |
| Light Orange | `#fff3e0` | Soft backgrounds |

---

## Part 5: Best Practices for EDN Editing

### Validation Rules

1. **UUID Format**: Must be valid v4 format with hyphens
   - Valid: `#uuid "550e8400-e29b-41d4-a716-446655440000"`
   - Invalid: `#uuid "550e8400e29b41d4a716446655440000"` (no hyphens)

2. **Map Closing**: Every opening `{` must have matching `}`
   - Track opening/closing brackets carefully
   - Use proper indentation for readability

3. **Keyword Consistency**: Keywords must start with `:` and use kebab-case
   - Valid: `:ls-type`, `:block/uuid`, `:logseq.tldraw.page`
   - Invalid: `:lsType`, `:block_uuid`

4. **Vectors**: Coordinates and sizes must be in `[x y]` format
   - Valid: `:point [100 200]`, `:size [300 150]`
   - Invalid: `:point {x: 100, y: 200}`

5. **String Escaping**: Escape special characters in text
   - Valid: `"Quote: \"Hello\""`
   - Invalid: `"Quote: "Hello""`

### Editing Tips

- **Use a JSON/EDN formatter**: Many tools can validate EDN syntax
- **Test incrementally**: Create simple shapes first, then add complexity
- **Copy-paste patterns**: Use existing shapes as templates
- **Maintain index order**: `:shapes-index` should list shapes in Z-order (back to front)
- **Comments are safe**: Use `;` to add notes without breaking syntax

### Common Mistakes to Avoid

```clojure
; ❌ WRONG: Missing closing bracket
{:block/created-at 1768611047550
 :block/properties {...

; ❌ WRONG: Unmatched quotes
:text "Unmatched quote

; ❌ WRONG: Missing colons on keywords
{block/uuid #uuid "..."

; ✅ CORRECT: Properly formatted
{:block/created-at 1768611047550
 :block/properties {...}}
```

---

## Integration Workflow

### For New Whiteboards:

1. Generate unique UUIDs and timestamps
2. Create `.edn` file in `whiteboards/` directory
3. Add page definition with UUID
4. Add shape blocks with correct parent references
5. Update `:shapes-index` with all shape IDs in Z-order
6. Validate EDN syntax before saving
7. Reload LogSeq to see the whiteboard

### For Existing Whiteboards:

1. Read the existing `.edn` file
2. Extract page UUID and current shape count
3. Generate new shape blocks with unique IDs
4. Append to `:blocks` vector
5. Append shape IDs to `:shapes-index`
6. Update page timestamps
7. Save and reload LogSeq

---

## Debugging

**Whiteboard not appearing in LogSeq?**
- Check that `:feature/enable-whiteboards? true` in `logseq/config.edn`
- Verify file is in `whiteboards/` directory with `.edn` extension
- Validate EDN syntax (no mismatched brackets or quotes)
- Ensure page UUID matches in `:block/uuid`, `:id`, and all `:parentId` fields

**Shapes not visible?**
- Check shape `:point` and `:size` are within viewport bounds
- Verify `:opacity` is > 0
- Ensure shape ID is in page's `:shapes-index` array
- Check `:type` is a valid shape type

**Styling not applied?**
- Validate color hex values (must be proper 6-digit hex or empty string)
- Check `:fontSize` is reasonable (10-100 pixels typical)
- Verify `:fontWeight` is valid (400, 700, etc.)
- Ensure `:scale` is `[1 1]` for normal display

---

## Resources

- **LogSeq Official**: https://logseq.com
- **EDN Specification**: https://github.com/edn-format/edn
- **tldraw (shape library)**: https://tldraw.com
- **LogSeq Graph Structure**: See `logseq/config.edn` for configuration options
