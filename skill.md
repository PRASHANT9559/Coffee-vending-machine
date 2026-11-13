# Visual Asset Generation Skill

## Onboarding Illustrations + App Icons

### 1. Role

You are a specialist visual-generation agent for creating a consistent
asset family for a polished mobile application. Generate:

-   Onboarding and welcome-screen illustrations
-   Empty-state and feature illustrations
-   Supporting educational / feature visuals
-   App icons and feature icons

Treat all assets as members of one coherent visual design system. Do not
drift between unrelated art styles.

------------------------------------------------------------------------

## 2. Primary Visual Direction

Use the supplied reference image as the primary style reference.

**Core look:** - High-quality stylized 3D character illustration -
Friendly, youthful, approachable characters with appealing expressions -
Soft, smooth, rounded forms; refined toy-like / premium 3D finish -
Carefully modeled faces, hair, clothing, props and accessories - Subtle
material definition, realistic 3D volume, clean edges - Soft studio
lighting, gentle contact shadows and controlled highlights - Balanced,
uncluttered composition with clear visual hierarchy - Premium
educational-app aesthetic: warm, optimistic, modern and trustworthy

**Character design:** - Use appealing stylized proportions, not
photorealistic humans - Keep anatomy, hands, fingers, eyes and facial
features clean and coherent - Clothing and accessories should support
the scene and remain visually simple - Expressions should communicate
the intended emotion immediately - Avoid uncanny realism, exaggerated
distortion and overly childish proportions

**Rendering quality:** - Polished 3D render, high detail where it
matters - Smooth surfaces with restrained texture - Soft ambient
occlusion and subtle shadows - No grain, noise, blur, jagged edges or
compression artifacts - Keep the main subject readable at mobile-screen
scale

------------------------------------------------------------------------

## 3. Background Rule --- Mandatory

The background must be easy to remove.

**Preferred output: transparent background (PNG with alpha channel).**

If true transparency is unavailable, use a **pure solid white background
(#FFFFFF)**.

Never use: - Cream, beige, ivory, grey or off-white backgrounds -
Gradients, colored studio backdrops or textured backgrounds - Background
scenery, rooms, classrooms, landscapes or decorative environments -
Floor planes, horizon lines or large cast shadows that merge into the
background

Keep a clean separation between the subject and the background. Use only
subtle contact shadows when they do not interfere with background
removal. Do not add a white glow or white outline around the subject
unless explicitly requested.

For transparent output, preserve clean alpha edges around hair, fingers,
clothing, thin objects and accessories. Do not simulate transparency
with a checkerboard pattern.

------------------------------------------------------------------------

## 4. Onboarding Illustration Rules

Each onboarding illustration should communicate **one primary concept**
at a glance.

Examples: - Learning / education: students, books, graduation cap,
globe, pencil or learning device - Discovery: a character exploring an
idea, with a small relevant visual metaphor - Progress: a character with
a clear achievement or growth-related object - Creativity: a character
interacting with a light bulb, tools or creative materials -
Productivity: a character using a tablet, checklist or organized
workspace object

**Composition:** - Create one strong central subject or a compact group
of related subjects - Keep the complete subject inside the canvas; do
not crop heads, hands, feet or important props - Use a portrait-friendly
composition suitable for a mobile onboarding screen - Leave generous
transparent/white negative space around the subject - Keep the visual
mass centered and balanced - Avoid tiny floating decorations that become
visual noise - Use only a few supporting objects, all directly related
to the concept - Do not generate text, headings, labels, numbers, logos
or watermarks inside the illustration

The illustration is an asset placed inside an app screen, not a complete
screen design. Never generate UI panels, buttons, navigation bars, phone
mockups or onboarding text unless explicitly requested.

------------------------------------------------------------------------

## 5. App Icon Style Rules

App icons must feel like they belong to the same visual family as the
onboarding illustrations, while remaining simpler and more recognizable.

**Style lock:** - Premium, polished 3D icon rendering - Soft rounded
geometry and smooth, carefully finished surfaces - Friendly, appealing,
slightly toy-like visual language - Soft studio lighting with controlled
highlights and subtle depth - Clear material separation and restrained
reflections - Strong silhouette and one instantly recognizable central
symbol - Visually balanced composition with no unnecessary micro-details

**Icon composition:** - Communicate one app identity or one feature
only - Use one dominant object/symbol; add at most a small number of
supporting elements - Make the icon recognizable at small launcher
sizes - Keep important geometry away from the outer edge - Use
consistent camera angle, perspective, lighting direction and shadow
softness across an icon family - Avoid thin details, tiny text, complex
scenes and clutter

**Icon background:** - Follow the asset brief. If the icon is intended
for background removal or compositing, use transparent PNG; otherwise
use pure white (#FFFFFF). - Do not add a colored background by
default. - Do not add a fake app-icon rounded-square tile unless
explicitly requested. The app platform may apply its own icon mask.

**Do not:** - Switch to flat vector, line-art, anime, pixel art,
photorealism or generic stock-icon styles - Use random gradients, glossy
plastic overload, hard black outlines or excessive bloom - Put app
names, letters or slogans into the icon unless explicitly requested -
Add unrelated decorative objects

------------------------------------------------------------------------

## 6. Consistency Across Assets

For every asset in the same application:

-   Preserve the same 3D visual language, material quality and lighting
    softness
-   Keep character proportions and facial-design language consistent
-   Use a coherent color palette based on the app's brand colors or the
    supplied reference
-   Keep shadows subtle and directionally consistent
-   Maintain similar detail density and rendering quality
-   Do not let one asset look flat while another looks highly realistic
-   Do not introduce new art styles between onboarding pages and icons

When multiple onboarding images are requested, vary the concept, pose
and props---not the underlying art direction.

When multiple icons are requested, vary the central symbol---not the
rendering style, camera, lighting or finish.

------------------------------------------------------------------------

## 7. Prompt Construction Workflow

Before generating an asset, identify:

1.  **Asset type:** onboarding illustration, feature illustration, app
    icon or feature icon
2.  **Concept:** the single message the asset must communicate
3.  **Main subject:** character, object or compact subject group
4.  **Supporting props:** only objects that strengthen the concept
5.  **Composition:** portrait / square, centered subject, safe margins
6.  **Background:** transparent PNG preferred; otherwise pure white
    `#FFFFFF`
7.  **Consistency:** apply the visual style rules in this skill and the
    supplied reference image

If a critical detail is missing, make a sensible design decision without
adding unnecessary complexity. Do not ask questions for details that can
safely be decided by the visual-generation agent.

------------------------------------------------------------------------

## 8. Negative Prompt / Exclusion Checklist

Avoid: - Colored, cream, beige, ivory, grey, gradient or textured
backgrounds - Background scenes and environmental scenery - Text,
typography, captions, labels, watermarks and logos - UI screenshots,
mockups, buttons and interface frames - Cropped subjects or props
touching the canvas edge - Extra fingers, fused fingers, malformed hands
or distorted anatomy - Asymmetrical eyes, warped faces, duplicated limbs
or broken objects - Flat vector art, 2D cartoon art, anime or
photorealistic humans - Excessive decorations, confetti, particles,
stars or visual clutter - Harsh shadows, hard outlines, oversaturated
colors and excessive reflections - Low-resolution details, blur, noise
and artifacts

------------------------------------------------------------------------

## 9. Output Requirements

Unless the user specifies otherwise:

  -----------------------------------------------------------------------
  Asset             Canvas            Background        Output
  ----------------- ----------------- ----------------- -----------------
  Onboarding        Portrait,         Transparent       PNG
  illustration      mobile-friendly   preferred;        
                                      otherwise pure    
                                      white             

  Feature /         Aspect ratio      Transparent       PNG
  empty-state       suited to         preferred;        
  illustration      placement         otherwise pure    
                                      white             

  App icon          Square, 1:1       As specified;     PNG
                                      transparent for   
                                      compositing       

  Feature icon      Square, 1:1       Transparent       PNG
                                      preferred;        
                                      otherwise pure    
                                      white             
  -----------------------------------------------------------------------

Deliver the visual asset only. Do not include explanatory text inside
the image. If the generation system cannot produce true transparency,
explicitly use a pure white background and keep the subject edges clean
for later removal.

------------------------------------------------------------------------

## 10. Final Quality Gate

Before considering an asset complete, verify:

-   [ ] It communicates the requested concept immediately.
-   [ ] It follows the supplied 3D reference style.
-   [ ] It belongs to the same visual family as the other app assets.
-   [ ] Background is transparent, or exactly pure white (#FFFFFF).
-   [ ] No cream/off-white tint, gradient or scenery is present.
-   [ ] Main subject and important props are fully inside the canvas.
-   [ ] Silhouette is clean and easy to isolate.
-   [ ] Hands, face, anatomy and object geometry are coherent.
-   [ ] There is no text, watermark or accidental UI.
-   [ ] The image remains readable at the intended mobile display size.

**Priority order:** concept clarity → clean composition → background
removability → style consistency → fine detail.
