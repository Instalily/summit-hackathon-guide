# Assets Directory

Place your PDF files in this directory to host them on your GitHub Pages site.

## How to Add Your PDF

1. **Name your PDF file**: `hackathon-guide.pdf` (or update the references in the pages)
2. **Place it in this `/assets` directory**
3. **Commit and push to GitHub**:
   ```bash
   git add assets/hackathon-guide.pdf
   git commit -m "Add hackathon guide PDF"
   git push
   ```

## PDF Display Options

Your PDF will be accessible in multiple ways:

1. **Direct link**: `https://your-username.github.io/summit-hackathon-guide/assets/hackathon-guide.pdf`
2. **Resources page**: Navigate to Resources & Downloads in your site
3. **PDF Viewer page**: Full-page PDF viewer with controls
4. **Embedded in any page**: Using the embed code provided

## Supported File Types

- `.pdf` - PDF documents
- `.png`, `.jpg`, `.jpeg` - Images
- `.zip` - Downloadable archives
- `.md` - Markdown documents

## File Size Limits

GitHub Pages has a soft limit of 100MB per file. Keep PDFs optimized for web viewing.

## Embedding PDFs in Other Pages

To embed a PDF in any markdown page:

```markdown
<embed src="/assets/hackathon-guide.pdf" width="100%" height="600px" type="application/pdf">
```

Or with a download fallback:

```markdown
[View PDF](assets/hackathon-guide.pdf){:target="_blank"} | [Download](assets/hackathon-guide.pdf){:download="hackathon-guide.pdf"}
```