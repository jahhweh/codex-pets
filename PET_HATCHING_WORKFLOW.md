# Pet Hatching Workflow

This repository uses one durable base-to-grid production pipeline for every new or rebuilt Codex pet.

## Scope

- Source concepts live in `concepts/`.
- Working runs live in `pet-runs/<slug>-v2-<date>-<theme>/`.
- Final repo pets live at the repo root in `<slug>/`.
- Live installs under `C:\Users\zach\.codex\pets\` are optional and only happen when the user explicitly asks for an install.

## Canonical order

1. Start from one approved concept image or approved text brief. Treat the concept as the source of truth for silhouette, face, palette, props, material, and attitude.
2. Lock one visually approved base image as the identity source. Check the face, silhouette, palette, materials, props, sharpness, and style at approximately final pet size before any row work starts.
3. Regenerate every animation row as a complete strip from that base. Generate the 9 standard rows plus both 8-frame look rows. Attach the canonical base to every row-generation job. Layout guides, cardinal references, and prior approved rows are supplemental only.
4. Extract each individual pose from every generated strip. Use deterministic component isolation when neighboring poses touch or visually connect. Reject incomplete, clipped, overlapping, or guide-contaminated pose groups.
5. Place the extracted poses into the deterministic 8x11 sprite grid using the expected row and frame counts. Do not hand-paste, tile, or mix unrelated one-off cells into the final atlas.
6. Inspect each row and the assembled contact sheet at normal pet size. If a row has clipping, pose fragments, overlap, detached artifacts, identity drift, wrong direction, or visible background/guide marks, regenerate the complete row from the base. Use deterministic extraction or margin correction only when the source row is visually sound.
7. Apply exactly one final chroma cleanup, then run strict v2 validation and final visual QA.
8. Publish the repo pet folder and README catalog row only after the atlas, manifest, cleanup report, structural checks, and visual review all pass.

## Working inputs and outputs

- Concept images are references, not final sprites.
- The working run should keep source images in `references/` and selected outputs in `decoded/`.
- Generated row strips and intermediate assets belong in the run folder, not in the final repo pet folder.
- Final atlas output is RGBA `1536x2288`, 8 columns by 11 rows, with `192x208` cells.
- Metadata must use `spriteVersionNumber: 2` and `spritesheetPath: "spritesheet.webp"`.

## Sprite grid contract

- Row 0: `idle`
- Row 1: `running-right`
- Row 2: `running-left`
- Row 3: `waving`
- Row 4: `jumping`
- Row 5: `failed`
- Row 6: `waiting`
- Row 7: `running`
- Row 8: `review`
- Row 9: look directions `000`, `022.5`, `045`, `067.5`, `090`, `112.5`, `135`, `157.5`
- Row 10: look directions `180`, `202.5`, `225`, `247.5`, `270`, `292.5`, `315`, `337.5`

## Final folder contract

Every finished repo pet folder must use the slug name and contain only the durable deliverables:

```text
<slug>/
  pet.json
  preview.webp
  spritesheet.webp
```

- `pet.json` must declare the same slug in `id`, a human-readable `displayName`, a concise `description`, `spriteVersionNumber: 2`, and `spritesheetPath: "spritesheet.webp"`.
- `preview.webp` is the gallery image used by the root README catalog. Default to the approved idle crop unless the pet's established convention uses a different approved preview.
- `spritesheet.webp` is the validated final atlas. Do not treat a run artifact as a finished pet until this file exists in the repo pet folder.
- If a final atlas repair touches compiled output, propagate the same correction back into the extracted source frame or row strip, then rebuild the atlas from those corrected extracts before final validation. Keep source frames, run artifacts, and the packaged atlas in sync.

## README catalog rule

The root `README.md` is the public index of finished pets. Add one row per finished pet only after the repo pet folder exists and the preview file is present.

```md
| Pet Name | <img src="slug/preview.webp" alt="Pet Name" width="96"> |
```

Keep the table aligned with the repo pet folder name, not with temporary run names.

## File and folder conventions

- Keep concepts in `concepts/` and do not overwrite them with generated output.
- Keep the active run in `pet-runs/` and use the run folder for prompts, QA, frame extraction, and final validation artifacts.
- Keep final repo pets at the repository root, one folder per pet.
- Keep generated images separate from live installs until the user explicitly asks for installation.
- Use the slug consistently across the repo folder, `pet.json`, README row, run folder naming, and any handoff notes.

## Quality standards

- No sticker borders, halos, or white outlines.
- No text, labels, UI, or readable logos unless the user explicitly provides approved reference art and asks for them.
- No clipped body parts, stray pixels, detached effects, or mixed-generation repair cells in the final atlas.
- No hand-pasting one-off cells into the final grid.
- No visible guide lines or chroma contamination in the final package.
- No install step unless the user explicitly requests the install after validation.

## Acceptance contract

- Every used cell contains a complete extracted pose; unused cells are transparent.
- Every source pose has at least 16 pixels of clear chroma and every normalized cell has an 8-pixel transparent perimeter.
- Row inspection, contact-sheet review, frame inspection, final-cell margin audit, one final cleanup report, and `validate_atlas.py --require-v2` all pass.
- A failed visual row is regenerated as a complete row. Individual repair cells are never pasted into a final atlas.

## Credential fallback

When the user explicitly authorizes the CLI image-generation fallback, resolve credentials in this order: current process environment, then the exact workspace-root `.env` file for this workspace (`I:\.env`). Parse `OPENAI_API_KEY=` in-process, inject it only into the child CLI, and never print or expose the value. An absent process-level variable is not proof that the key is missing.
