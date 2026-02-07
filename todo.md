# Homelab website todo

## Phase 1: Copy Quartz framework from penningmeester

- [ ] Copy the `quartz/` directory:
  ```bash
  cp -r ~/dev/penningmeester/quartz ~/dev/homelab-website/quartz
  ```

- [ ] Copy these root files as-is (no modifications needed):
  ```bash
  cd ~/dev/penningmeester
  cp package.json package-lock.json tsconfig.json globals.d.ts index.d.ts \
     .npmrc .node-version .prettierrc .prettierignore .gitattributes \
     ~/dev/homelab-website/
  ```

- [ ] Install node dependencies:
  ```bash
  cd ~/dev/homelab-website && npm ci
  ```

## Phase 2: Create symlink and rename index

- [ ] Rename README.md to index.md in the vault:
  ```bash
  mv ~/Cloud/janvv/life/docs/homelab/README.md ~/Cloud/janvv/life/docs/homelab/index.md
  ```

- [ ] Create the content symlink:
  ```bash
  ln -s ~/Cloud/janvv/life/docs/homelab ~/dev/homelab-website/content
  ```

## Phase 3: Create config files

- [ ] Create `quartz.config.ts` by copying from penningmeester and making these exact changes:
  - Line 11: `pageTitle: "Quartz 4"` -> `pageTitle: "Opa Jan's Homelab"`
  - Line 15-17: replace the analytics block with `analytics: null,`
  - Line 19: `baseUrl: "quartz.jzhao.xyz"` -> `baseUrl: "homelab.janvv.nl"`
  - Line 20: `ignorePatterns: ["private", "templates", ".obsidian"]` -> `ignorePatterns: ["private", "templates", "CLAUDE.md", ".obsidian"]`
  - Keep everything else (plugins, theme, colors, fonts) exactly the same.

  The full file should be:
  ```typescript
  import { QuartzConfig } from "./quartz/cfg"
  import * as Plugin from "./quartz/plugins"

  const config: QuartzConfig = {
    configuration: {
      pageTitle: "Opa Jan's Homelab",
      pageTitleSuffix: "",
      enableSPA: true,
      enablePopovers: true,
      analytics: null,
      locale: "en-US",
      baseUrl: "homelab.janvv.nl",
      ignorePatterns: ["private", "templates", "CLAUDE.md", ".obsidian"],
      defaultDateType: "modified",
      theme: {
        fontOrigin: "googleFonts",
        cdnCaching: true,
        typography: {
          header: "Schibsted Grotesk",
          body: "Source Sans Pro",
          code: "IBM Plex Mono",
        },
        colors: {
          lightMode: {
            light: "#faf8f8",
            lightgray: "#e5e5e5",
            gray: "#b8b8b8",
            darkgray: "#4e4e4e",
            dark: "#2b2b2b",
            secondary: "#284b63",
            tertiary: "#84a59d",
            highlight: "rgba(143, 159, 169, 0.15)",
            textHighlight: "#fff23688",
          },
          darkMode: {
            light: "#161618",
            lightgray: "#393639",
            gray: "#646464",
            darkgray: "#d4d4d4",
            dark: "#ebebec",
            secondary: "#7b97aa",
            tertiary: "#84a59d",
            highlight: "rgba(143, 159, 169, 0.15)",
            textHighlight: "#b3aa0288",
          },
        },
      },
    },
    plugins: {
      transformers: [
        Plugin.FrontMatter(),
        Plugin.CreatedModifiedDate({
          priority: ["frontmatter", "git", "filesystem"],
        }),
        Plugin.SyntaxHighlighting({
          theme: {
            light: "github-light",
            dark: "github-dark",
          },
          keepBackground: false,
        }),
        Plugin.ObsidianFlavoredMarkdown({ enableInHtmlEmbed: false }),
        Plugin.GitHubFlavoredMarkdown(),
        Plugin.TableOfContents(),
        Plugin.CrawlLinks({ markdownLinkResolution: "shortest" }),
        Plugin.Description(),
        Plugin.Latex({ renderEngine: "katex" }),
      ],
      filters: [Plugin.RemoveDrafts()],
      emitters: [
        Plugin.AliasRedirects(),
        Plugin.ComponentResources(),
        Plugin.ContentPage(),
        Plugin.FolderPage(),
        Plugin.TagPage(),
        Plugin.ContentIndex({
          enableSiteMap: true,
          enableRSS: true,
        }),
        Plugin.Assets(),
        Plugin.Static(),
        Plugin.Favicon(),
        Plugin.NotFoundPage(),
        Plugin.CustomOgImages(),
      ],
    },
  }

  export default config
  ```

- [ ] Create `quartz.layout.ts` by copying from penningmeester and making these changes:
  - Footer: replace GitHub/Discord links with just GitHub link to this repo
  - Left sidebar: remove `ReaderMode` from the Flex components (keep only Search + Darkmode)
  - Right sidebar: remove `Graph`, keep only TableOfContents and Backlinks
  - Keep everything else exactly the same.

  The full file should be:
  ```typescript
  import { PageLayout, SharedLayout } from "./quartz/cfg"
  import * as Component from "./quartz/components"

  export const sharedPageComponents: SharedLayout = {
    head: Component.Head(),
    header: [],
    afterBody: [],
    footer: Component.Footer({
      links: {
        GitHub: "https://github.com/opajanvv/homelab-website",
      },
    }),
  }

  export const defaultContentPageLayout: PageLayout = {
    beforeBody: [
      Component.ConditionalRender({
        component: Component.Breadcrumbs(),
        condition: (page) => page.fileData.slug !== "index",
      }),
      Component.ArticleTitle(),
      Component.ContentMeta(),
      Component.TagList(),
    ],
    left: [
      Component.PageTitle(),
      Component.MobileOnly(Component.Spacer()),
      Component.Flex({
        components: [
          {
            Component: Component.Search(),
            grow: true,
          },
          { Component: Component.Darkmode() },
        ],
      }),
      Component.Explorer(),
    ],
    right: [
      Component.DesktopOnly(Component.TableOfContents()),
      Component.Backlinks(),
    ],
  }

  export const defaultListPageLayout: PageLayout = {
    beforeBody: [Component.Breadcrumbs(), Component.ArticleTitle(), Component.ContentMeta()],
    left: [
      Component.PageTitle(),
      Component.MobileOnly(Component.Spacer()),
      Component.Flex({
        components: [
          {
            Component: Component.Search(),
            grow: true,
          },
          { Component: Component.Darkmode() },
        ],
      }),
      Component.Explorer(),
    ],
    right: [],
  }
  ```

- [ ] Create `.gitignore` with this exact content:
  ```
  .DS_Store
  node_modules
  public
  prof
  tsconfig.tsbuildinfo
  .obsidian
  .quartz-cache
  private/
  .replit
  replit.nix
  content
  ```
  (Same as penningmeester's .gitignore, plus `content` at the end to ignore the symlink)

## Phase 4: Build and verify

- [ ] Build and serve locally to verify everything works:
  ```bash
  cd ~/dev/homelab-website && npx quartz build --serve
  ```
  The site should be available at http://localhost:8080. Check:
  - Homepage loads (index.md content renders)
  - Navigation/explorer shows the folder structure (how-to, infrastructure, services)
  - Wiki-style links between pages work
  - Service docs render with frontmatter metadata
  - Search works
  - No build errors in the terminal

- [ ] Fix any build issues before proceeding.

## Phase 5: Git and deployment setup

- [ ] Create `.github/workflows/deploy.yml` with this exact content:
  ```yaml
  name: Deploy Quartz to GitHub Pages

  on:
    push:
      branches:
        - main

  permissions:
    contents: read
    pages: write
    id-token: write

  concurrency:
    group: "pages"
    cancel-in-progress: false

  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - name: Checkout
          uses: actions/checkout@v4
          with:
            fetch-depth: 0
        - name: Setup Node
          uses: actions/setup-node@v4
          with:
            node-version: 22
        - name: Install Dependencies
          run: npm ci
        - name: Build Quartz
          run: npx quartz build
        - name: Add CNAME
          run: echo "homelab.janvv.nl" > public/CNAME
        - name: Upload artifact
          uses: actions/upload-pages-artifact@v3
          with:
            path: public

    deploy:
      needs: build
      environment:
        name: github-pages
        url: ${{ steps.deployment.outputs.page_url }}
      runs-on: ubuntu-latest
      steps:
        - name: Deploy to GitHub Pages
          id: deployment
          uses: actions/deploy-pages@v4
  ```
  Note: This workflow won't work yet because `content/` is gitignored (it's a symlink to the vault). Deployment requires a content sync strategy to be decided later.

- [ ] Initialize git repo and create initial commit:
  ```bash
  cd ~/dev/homelab-website
  git init
  git add .
  git commit -m "initial quartz setup for homelab docs"
  ```

- [ ] Create the GitHub repo and push (do NOT do this automatically - ask Jan first):
  ```bash
  gh repo create opajanvv/homelab-website --public --source=. --push
  ```

## Notes

- The `content` symlink is for local development only. It points to `~/Cloud/janvv/life/docs/homelab/`.
- For GitHub Pages deployment to work, the content needs to be available in the repo. Options to solve this later:
  1. Copy content files into the repo before pushing (remove symlink, copy files, commit, push)
  2. Use a GitHub Action that checks out content from a separate repo
  3. Move to self-hosted deployment on Proxmox where the vault is accessible
- The `spec.md` and `todo.md` files should be deleted or gitignored after setup is complete.
