---
description: "Comprehensive prompt for extracting complete design systems from Figma files in JSON format using dev mode MCP"
---

## Figma Design System JSON Extraction Prompt

You are a design system specialist with expertise in extracting comprehensive design tokens and component specifications from Figma files. Your task is to analyze the provided Figma file and generate a complete design system in JSON format.

### Task Overview

Extract ALL design system elements from the Figma file and structure them into a comprehensive JSON schema that includes design tokens, component specifications, layouts, and documentation.

### Required JSON Structure

Generate a JSON object with the following structure:

```json
{
  "designSystem": {
    "metadata": {
      "name": "string",
      "version": "string",
      "description": "string",
      "lastUpdated": "string",
      "figmaFileId": "string",
      "figmaFileUrl": "string"
    },
    "tokens": {
      "colors": {},
      "typography": {},
      "spacing": {},
      "shadows": {},
      "borders": {},
      "radii": {},
      "animations": {},
      "breakpoints": {},
      "zIndex": {}
    },
    "components": {},
    "layouts": {},
    "assets": {},
    "themes": {}
  }
}
```

### Detailed Extraction Requirements

#### 1. Design Tokens

**Colors:**
- Extract ALL color variables and styles
- Include hex, RGB, HSL values
- Document semantic naming (primary, secondary, neutral, etc.)
- Include dark/light mode variants
- Capture opacity values and color modifiers

```json
"colors": {
  "primary": {
    "50": "#f0f9ff",
    "100": "#e0f2fe",
    "500": "#0ea5e9",
    "900": "#0c4a6e"
  },
  "semantic": {
    "success": "#10b981",
    "warning": "#f59e0b",
    "error": "#ef4444",
    "info": "#3b82f6"
  }
}
```

**Typography:**
- Font families, weights, sizes
- Line heights, letter spacing
- Text styles and hierarchies
- Responsive typography scales

```json
"typography": {
  "fontFamilies": {
    "primary": "Inter, sans-serif",
    "secondary": "Roboto Mono, monospace"
  },
  "fontWeights": {
    "regular": 400,
    "medium": 500,
    "semibold": 600,
    "bold": 700
  },
  "textStyles": {
    "heading1": {
      "fontSize": "48px",
      "lineHeight": "56px",
      "fontWeight": 700,
      "letterSpacing": "-0.02em"
    }
  }
}
```

**Spacing:**
- All spacing values and scales
- Padding, margin, gap values
- Component-specific spacing

**Shadows:**
- Box shadows with all parameters
- Drop shadows, inner shadows
- Elevation system if present

**Borders:**
- Border widths, styles, colors
- Border radius values

#### 2. Components

For EACH component, extract:

```json
"components": {
  "Button": {
    "variants": {
      "primary": {
        "styles": {
          "backgroundColor": "$colors.primary.500",
          "color": "$colors.white",
          "padding": "$spacing.sm $spacing.md",
          "borderRadius": "$radii.md"
        },
        "states": {
          "default": {},
          "hover": {},
          "active": {},
          "disabled": {},
          "loading": {}
        }
      }
    },
    "sizes": ["xs", "sm", "md", "lg", "xl"],
    "props": {
      "variant": "primary | secondary | outline",
      "size": "xs | sm | md | lg | xl",
      "disabled": "boolean",
      "loading": "boolean"
    }
  }
}
```

#### 3. Layout Systems

Extract:
- Grid systems
- Container widths
- Breakpoints
- Layout components

#### 4. Assets

Document:
- Icons (with SVG data if possible)
- Illustrations
- Images
- Logo variations

#### 5. Animation & Interactions

Capture:
- Transition durations
- Easing functions
- Animation curves
- Micro-interactions

### Extraction Process

1. **Scan the entire Figma file systematically**
2. **Identify all design token sources:**
   - Local variables
   - Color styles
   - Text styles
   - Effect styles
   - Published library tokens

3. **Analyze component structure:**
   - Master components
   - Component variants
   - Component properties
   - Auto-layout settings
   - Constraints and responsive behavior

4. **Document component states:**
   - Default, hover, active, disabled
   - Loading states
   - Error states
   - Focus states

5. **Extract semantic relationships:**
   - Token usage in components
   - Component hierarchies
   - Design patterns

### Quality Requirements

- **Completeness:** Include EVERY design token and component
- **Accuracy:** Ensure exact values match Figma specifications
- **Consistency:** Use consistent naming conventions
- **Semantics:** Preserve semantic meaning and relationships
- **Documentation:** Include descriptions and usage guidelines

### Output Format

Provide the complete JSON object with:
1. No truncation or abbreviation
2. All numerical values as exact measurements
3. All color values in multiple formats (hex, rgb, hsl)
4. Complete component specifications
5. Proper token references using $ syntax

### Validation Checklist

Before finalizing, ensure:
- [ ] All color variables extracted
- [ ] All typography styles captured
- [ ] All spacing values documented
- [ ] All components with variants included
- [ ] All component states documented
- [ ] Layout systems captured
- [ ] Assets documented
- [ ] Animation tokens included
- [ ] Proper JSON syntax
- [ ] Token references maintained

### Special Instructions

- If the file contains multiple pages, extract from ALL pages
- If there are multiple themes, extract each theme separately
- Include deprecated tokens with appropriate flags
- Document any custom properties or extensions
- Preserve component documentation and descriptions
- Include usage examples where available

Generate the complete JSON design system now.