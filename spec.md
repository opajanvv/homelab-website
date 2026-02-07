# Homelab website spec

## Goal

Publish the homelab documentation at `~/Cloud/janvv/life/docs/homelab/` as a static Quartz site, accessible at `homelab.janvv.nl`.

## Decisions

- **Framework**: Quartz v4.5.2 (copy from ~/dev/penningmeester)
- **Content source**: Symlink `content/` -> `~/Cloud/janvv/life/docs/homelab/`
- **Index page**: Rename `README.md` to `index.md` in the vault
- **Site title**: "Opa Jan's Homelab"
- **Base URL**: `homelab.janvv.nl` (CNAME to GitHub Pages)
- **Deployment**: GitHub Pages via GitHub Actions (later migrate to Proxmox)
- **Visibility**: Public. Internal IPs/ports in content are acceptable.
- **GitHub repo**: `opajanvv/homelab-website` (does not exist yet)

## Content considerations

- Content uses Obsidian wiki-style links `[[path/doc]]` - handled by ObsidianFlavoredMarkdown plugin
- Custom frontmatter fields (service, status, host, port, tags, updated) - parsed by FrontMatter plugin
- `templates/` directory should be ignored (not published)
- `CLAUDE.md` should be ignored (not published)
- The symlink means content is NOT in the git repo. Deployment to GitHub Pages is deferred (set up the workflow file, but content won't be available in CI until a sync strategy is decided).
