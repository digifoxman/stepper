# Stepper

A click-through, step-by-step diagram explainer for Astro sites. You describe the diagram and the steps in two JSON files; the `Stepper` component lays the boxes out on an 8 × 12 grid, draws the arrows between them, and reveals and highlights them step by step.

## Getting started

You need Node.js 22.23 or later (`node --version` to check).

1. Open the workspace file in VS Code:

   ```sh
   code ~/development/stepper/stepper.code-workspace
   ```

   Or use **File → Open Workspace from File…** and pick `stepper.code-workspace`. Opening the workspace file, rather than just the folder, is what switches on the run configuration and the autocomplete for the JSON files.

2. When VS Code offers to install the recommended **Astro** extension, accept. It gives you syntax highlighting, autocomplete and error checking in `.astro` files.

3. In VS Code's terminal (**Terminal → New Terminal**), install the dependencies and start the development server:

   ```sh
   npm install
   npm run dev
   ```

   You can also start the server from **Run and Debug** with the **Development server** configuration.

4. Open <http://localhost:4321> for the example explainer, or <http://localhost:4321/layout> for the layout view, which shows the grid with row and column numbers. Pages reload as you save changes.

## Project structure

```text
stepper/
├── stepper.code-workspace     VS Code workspace: recommends the Astro extension,
│                              adds the "Development server" run configuration
│                              and links the JSON files to their schemas
├── public/
│   └── favicon.svg
├── schemas/
│   ├── diagram.schema.json    describes the diagram file, for autocomplete
│   └── steps.schema.json      describes the steps file, for autocomplete
├── src/
│   ├── components/
│   │   └── Stepper.astro      the component
│   ├── data/
│   │   ├── order-diagram.json example diagram
│   │   └── order-steps.json   example steps
│   ├── layouts/
│   │   └── Base.astro         page shell shared by the pages
│   └── pages/
│       ├── index.astro        the example explainer
│       └── layout.astro       the example with the grid shown
├── astro.config.mjs
├── package.json
└── tsconfig.json
```

## Making a new explainer

1. In `src/data/`, create `<name>-diagram.json` and `<name>-steps.json`, for example `signup-diagram.json` and `signup-steps.json`. Keep those name endings: they're how VS Code knows which schema to use for autocomplete and error checking.
2. Create a page in `src/pages/`, for example `signup.astro`:

   ```astro
   ---
   import Base from "../layouts/Base.astro";
   import Stepper from "../components/Stepper.astro";
   import diagram from "../data/signup-diagram.json";
   import steps from "../data/signup-steps.json";
   ---

   <Base title="How sign-up works">
     <h1>How sign-up works</h1>
     <Stepper diagram={diagram} steps={steps} />
   </Base>
   ```

3. Open <http://localhost:4321/signup>. While placing boxes, add `showGrid` to the component to see the numbered grid.

## The diagram file

```json
{
  "label": "Diagram of an online order moving from the customer to the shop.",
  "boxes": [
    { "id": "customer", "label": "Customer", "row": 3, "col": 1 },
    { "id": "shop", "label": "Online shop", "row": 3, "col": 5 }
  ],
  "arrows": [
    { "from": "customer", "to": "shop", "label": "Order" }
  ],
  "badges": [
    { "id": "paid", "box": "shop", "label": "✓ Paid", "position": "bottom-right" }
  ]
}
```

**`label`** describes the diagram for screen readers. If you leave it out, the component lists the box labels instead.

**Boxes**

| Field | Required | Meaning |
| --- | --- | --- |
| `id` | yes | Unique name, used by arrows and in the steps file |
| `label` | yes | Text in the box. Long labels wrap; `\n` forces a line break |
| `row` | yes | 1 (top) to 8 |
| `col` | yes | 1 (left) to 12 |
| `rowSpan`, `colSpan` | no | Make a box cover more than one cell |

**Arrows** run from the side of one box to the side of another. Boxes that share a row or column get a straight arrow; others get an L-shaped one.

| Field | Required | Meaning |
| --- | --- | --- |
| `from`, `to` | yes | The ids of the two boxes |
| `id` | no | Name used in the steps file. Default: `"from->to"`, e.g. `"customer->shop"` |
| `label` | no | Text beside the arrow |
| `dashed` | no | `true` for a dashed line, e.g. an email or other indirect message |
| `offset` | no | Shift the arrow sideways (one column is 100). Use `8` and `-8` for two arrows between the same boxes |
| `bend` | no | `"horizontal"` (default) or `"vertical"`: which way an L-shaped arrow leaves its box |
| `labelSide` | no | `"above"` (default) or `"below"` a horizontal arrow; `"right"` (default) or `"left"` of a vertical one |

Tip: leave an empty column or row between boxes when the arrow between them has a label, so the label has room.

**Badges** are short labels pinned to a corner of a box, such as a status that changes during the process ("✓ Paid", "Packed", "3 items"). A badge sits centred on the corner, so it overlaps the box's edge.

| Field | Required | Meaning |
| --- | --- | --- |
| `id` | yes | Unique name, used in the steps file |
| `box` | yes | The id of the box it's pinned to |
| `label` | yes | Short text. Keep it to a word or two; the badge grows with the text |
| `position` | no | `"top-right"` (default), `"top-left"`, `"bottom-right"` or `"bottom-left"` |

Pick a corner away from where arrows leave the box: arrows start and end at the middle of a side, so a badge on a corner rarely collides with them.

## The steps file

```json
[
  {
    "title": "Payment is authorised",
    "text": "The shop asks the payment provider to reserve the amount.",
    "show": ["payment", "shop->payment"],
    "highlight": ["payment"]
  }
]
```

| Field | Required | Meaning |
| --- | --- | --- |
| `title` | yes | Short heading for the step |
| `text` | yes | One or two sentences explaining the step |
| `show` | no | Boxes, arrows and badges that appear at this step and stay |
| `highlight` | no | What's drawn in the accent colour during this step. Default: the `show` list |
| `hide` | no | Boxes, arrows and badges removed from this step on |

Anything no step shows is visible from the start. Within a step, an arrow draws once the box it leaves from is on screen, a box appears once the arrows pointing to it have finished drawing, and a badge pops up once its box is there and those arrows have arrived.

If either file has a mistake (an id that doesn't exist, a box off the grid, two boxes in the same cell), `npm run dev` and `npm run build` stop with a message saying what to fix.

## Component options

| Prop | Default | Meaning |
| --- | --- | --- |
| `diagram` | | The diagram file |
| `steps` | | The steps file |
| `start` | `1` | Step to open on |
| `showGrid` | `false` | Draw the grid with row and column numbers |
| `label` | | Overrides the diagram file's `label` |
| `rows`, `cols` | `8`, `12` | Grid size. If you change these, update the maximums in `schemas/diagram.schema.json` too |

Colours and sizes are CSS custom properties, set like this:

```astro
<Stepper diagram={diagram} steps={steps} style="--stepper-accent: #c2410c" />
```

| Property | What it colours |
| --- | --- |
| `--stepper-accent` | Highlighted boxes, arrows and badges, the step number, the Next button |
| `--stepper-accent-soft` | Fill of highlighted boxes |
| `--stepper-text` | Box labels and caption text |
| `--stepper-label` | Arrow labels, and badges that aren't highlighted |
| `--stepper-muted` | Arrows that aren't highlighted |
| `--stepper-line` | Box borders |
| `--stepper-surface` | Box fill, badge text and button background |
| `--stepper-min-width` | Narrowest the diagram gets (default `720px`); on narrower screens it scrolls sideways |

## Behaviour

- **Keyboard:** the left and right arrow keys move between steps, and Home and End jump to the first and last step.
- **Phones:** the diagram scrolls sideways and centres on each step's highlighted part.
- **No JavaScript:** the finished diagram and all captions are shown as a list.
- **Reduced motion:** animations are switched off for people who have turned off animation in their system settings.
- **Screen readers:** each step's title and text is announced when it changes.
- **View transitions:** the component works with Astro's `<ClientRouter />` without extra code.

## Commands

| Command | What it does |
| --- | --- |
| `npm install` | Install dependencies |
| `npm run dev` | Start the development server at <http://localhost:4321> |
| `npm run build` | Build the site into `dist/` |
| `npm run preview` | Serve the built site locally |
| `npm run check` | Check `.astro` and TypeScript files for errors |

## Using the component in another Astro site

Copy `src/components/Stepper.astro` into that site's `src/components/` folder, and copy the two schemas and the `json.schemas` setting from `stepper.code-workspace` if you want autocomplete there too. The component has no dependencies beyond Astro.
