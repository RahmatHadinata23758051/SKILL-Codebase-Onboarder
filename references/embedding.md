# Embedding flow diagrams into deliverable files

Use this only when the user wants a *saved file* (docx/pdf/pptx/md), not an inline chat explanation. For inline, use the Visualizer instead — see main SKILL.md.

## Markdown (.md)
Mermaid renders natively. Just include a fenced ```mermaid code block in the file. No extra steps.

## Word (.docx) / PDF (.pdf) / PowerPoint (.pptx)
These don't render Mermaid natively. Two options, in order of preference:

1. **Render the Mermaid/diagram to SVG/PNG first, then embed as an image.**
   - Write the diagram as Mermaid in a `.mmd` file, then render with `mmdc` (mermaid-cli) if available:
     `npx -y @mermaid-js/mermaid-cli -i diagram.mmd -o diagram.png -b transparent`
   - If mermaid-cli isn't available or fails (no network access to npm beyond the allowed domains), draw the diagram directly as SVG (see the Visualizer `diagram` module's styling conventions for a clean look) and export/screenshot it, or hand-build simple boxes/arrows as a docx/pptx native shape diagram — check the relevant skill (`docx`/`pptx`) for how it handles shapes and images.
2. **For pptx specifically**, prefer native shapes (boxes + connectors) over an embedded image when the diagram is simple — it stays editable for the user. Follow the `pptx` skill's guidance on shapes.

Always double check the rendered image/diagram is legible at the target document's page/slide size before finalizing — flow diagrams with many nodes often need to be split across two pages/slides rather than shrunk to fit.
